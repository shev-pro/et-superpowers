# Go — Language-Specific Review Checks

Load this file during review when the diff contains `.go` files.

---

## Code Quality

### Error ignored with `_`
```go
// ❌
result, _ := db.Query("SELECT ...")
file, _ := os.Open(path)

// ✅
result, err := db.Query("SELECT ...")
if err != nil {
    return fmt.Errorf("querying users: %w", err)
}
```
Flag every `_` on the error return position outside of intentional patterns like `range`. 🔴

### Error not wrapped with context
```go
// ❌ — caller has no idea where the error came from
if err != nil {
    return err
}

// ✅
if err != nil {
    return fmt.Errorf("creating user %d: %w", userID, err)
}
```

### Named return values misused
```go
// ❌ — naked returns in long functions are confusing
func process() (result string, err error) {
    // ... 40 lines ...
    return
}

// ✅ — naked returns only in very short functions (< 5 lines)
func minMax(a, b int) (min, max int) {
    if a < b {
        return a, b
    }
    return b, a
}
```

### Struct without field tags on serialized types
```go
// ❌ — field names exposed as-is (PascalCase) in JSON
type User struct {
    UserID   int
    FullName string
}

// ✅
type User struct {
    UserID   int    `json:"user_id"`
    FullName string `json:"full_name"`
}
```

### `init()` function with side effects
Flag `init()` functions that open connections, read files, or call external services — these are hard to test and control. 🟡

---

## Concurrency

### Goroutine without done signal / context
```go
// ❌ — goroutine leaks if caller returns early
go func() {
    result := doWork()
    ch <- result
}()

// ✅
go func(ctx context.Context) {
    select {
    case <-ctx.Done():
        return
    case ch <- doWork():
    }
}(ctx)
```

### Context not propagated to downstream calls
```go
// ❌
func GetUser(id string) (*User, error) {
    row := db.QueryRow("SELECT * FROM users WHERE id = $1", id)
    ...
}

// ✅
func GetUser(ctx context.Context, id string) (*User, error) {
    row := db.QueryRowContext(ctx, "SELECT * FROM users WHERE id = $1", id)
    ...
}
```
Flag any function doing I/O (DB, HTTP, file) without `context.Context` as first parameter. 🟡

### Mutex not protecting all accesses to shared state
```go
// ❌ — read outside lock
type Cache struct {
    mu    sync.Mutex
    items map[string]Item
}

func (c *Cache) Get(key string) (Item, bool) {
    item, ok := c.items[key]  // ❌ no lock on read
    return item, ok
}

// ✅
func (c *Cache) Get(key string) (Item, bool) {
    c.mu.Lock()
    defer c.mu.Unlock()
    item, ok := c.items[key]
    return item, ok
}
// or use sync.RWMutex with RLock() for reads
```

### WaitGroup misuse
```go
// ❌ — Add() inside goroutine (race: goroutine may not start before Wait())
for _, item := range items {
    go func(i Item) {
        wg.Add(1)  // ❌
        defer wg.Done()
        process(i)
    }(item)
}

// ✅
for _, item := range items {
    wg.Add(1)
    go func(i Item) {
        defer wg.Done()
        process(i)
    }(item)
}
```

### Channel direction not specified in function signatures
```go
// ❌ — bidirectional channel, caller could accidentally send/receive wrong way
func producer(ch chan int) { ... }

// ✅ — directional channels document intent and catch bugs at compile time
func producer(ch chan<- int) { ... }
func consumer(ch <-chan int) { ... }
```

---

## Security (Go-specific)

### SQL injection via `fmt.Sprintf` in query
```go
// ❌
query := fmt.Sprintf("SELECT * FROM users WHERE name = '%s'", name)
db.Query(query)

// ✅
db.QueryContext(ctx, "SELECT * FROM users WHERE name = $1", name)
```

### `math/rand` for security-sensitive randomness
```go
// ❌ — predictable seed
import "math/rand"
token := rand.Int63()

// ✅
import "crypto/rand"
b := make([]byte, 32)
_, err := rand.Read(b)
```
Flag `math/rand` for token generation, session IDs, or any security-sensitive value. 🔴

### `http.ListenAndServe` with default mux
```go
// ❌ — default mux can be polluted by imported packages
http.HandleFunc("/api", handler)
http.ListenAndServe(":8080", nil)

// ✅
mux := http.NewServeMux()
mux.HandleFunc("/api", handler)
http.ListenAndServe(":8080", mux)
```

### G304 — file path from variable (path traversal)
```go
// ❌
filePath := r.URL.Query().Get("file")
f, err := os.Open(filePath)

// ✅
safeBase := filepath.Base(r.URL.Query().Get("file"))
f, err := os.Open(filepath.Join("/uploads", safeBase))
```

### Timeout missing on `http.Client`
```go
// ❌ — default client has no timeout
resp, err := http.Get(url)

// ✅
client := &http.Client{Timeout: 10 * time.Second}
resp, err := client.Get(url)
```

---

## Performance

### Unnecessary allocation in hot path
```go
// ❌ — allocates new slice header per call
func getNames(users []User) []string {
    names := []string{}
    for _, u := range users {
        names = append(names, u.Name)
    }
    return names
}

// ✅ — pre-allocate
func getNames(users []User) []string {
    names := make([]string, 0, len(users))
    for _, u := range users {
        names = append(names, u.Name)
    }
    return names
}
```

### String conversion in loop
```go
// ❌ — allocation per iteration
for _, b := range data {
    process(string(b))
}

// ✅
s := string(data)
for _, c := range s {
    process(string(c))
}
```

### Defer in loop
```go
// ❌ — defers accumulate until function returns, not loop iteration
for _, file := range files {
    f, _ := os.Open(file)
    defer f.Close()  // all closes happen at function end
}

// ✅
for _, file := range files {
    func() {
        f, err := os.Open(file)
        if err != nil { ... }
        defer f.Close()  // closes at end of anonymous function
        process(f)
    }()
}
```

### Interface method called in tight loop (heap escape)
Flag interface method calls inside performance-critical loops. Suggest concrete types or function pointers if profiling shows allocation overhead. 🟢

---

## Testing

### Test without `t.Parallel()`
```go
// ❌ — tests run sequentially by default
func TestGetUser(t *testing.T) { ... }

// ✅ — for tests that can run concurrently
func TestGetUser(t *testing.T) {
    t.Parallel()
    ...
}
```
Flag as 🟢 for new test files to encourage parallelism.

### Table-driven test missing subtests
```go
// ❌ — hard to identify which case fails
for _, tc := range testCases {
    result := process(tc.input)
    if result != tc.expected {
        t.Errorf("got %v, want %v", result, tc.expected)
    }
}

// ✅
for _, tc := range testCases {
    tc := tc  // capture range var
    t.Run(tc.name, func(t *testing.T) {
        t.Parallel()
        result := process(tc.input)
        require.Equal(t, tc.expected, result)
    })
}
```

### Comparing errors with `==` instead of `errors.Is`
```go
// ❌ — fails for wrapped errors
if err == sql.ErrNoRows { ... }

// ✅
if errors.Is(err, sql.ErrNoRows) { ... }
```

### Missing `t.Cleanup` for resource teardown
```go
// ❌ — resource not cleaned up if test panics
db := setupTestDB(t)
defer db.Close()

// ✅
db := setupTestDB(t)
t.Cleanup(func() { db.Close() })
```
