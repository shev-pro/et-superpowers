# Security Anti-Patterns by Stack

Reference file for Phase 3 of code review. Load when a security finding needs
more depth or when the diff touches auth, input handling, or external I/O.

---

## Python / FastAPI

### SQL Injection
```python
# ❌
query = f"SELECT * FROM users WHERE id = {user_id}"
db.execute(query)

# ✅
db.execute("SELECT * FROM users WHERE id = :id", {"id": user_id})
# or via ORM: db.query(User).filter(User.id == user_id)
```

### Hardcoded secrets
```python
# ❌
SECRET_KEY = "supersecret123"
API_KEY = "sk-abc123"

# ✅
SECRET_KEY = os.environ["SECRET_KEY"]  # or via pydantic BaseSettings
```

### Missing input validation on FastAPI endpoints
```python
# ❌ — accepts arbitrary body
@app.post("/users")
async def create_user(data: dict):
    ...

# ✅
class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    email: EmailStr

@app.post("/users")
async def create_user(data: UserCreate):
    ...
```

### Blocking I/O in async endpoint
```python
# ❌ — blocks event loop
@app.get("/report")
async def get_report():
    time.sleep(5)          # blocks
    result = requests.get(url)  # blocks

# ✅
@app.get("/report")
async def get_report():
    await asyncio.sleep(5)
    async with httpx.AsyncClient() as client:
        result = await client.get(url)
```

### SSRF via user-controlled URL
```python
# ❌
@app.get("/fetch")
async def fetch(url: str):
    return requests.get(url).text  # user controls destination
```
Flag any endpoint that fetches a URL derived from user input without allowlist validation.

---

## Java / Spring

### SQL Injection
```java
// ❌
String query = "SELECT * FROM users WHERE id = " + userId;
jdbcTemplate.queryForObject(query, User.class);

// ✅
jdbcTemplate.queryForObject(
    "SELECT * FROM users WHERE id = ?", User.class, userId);
```

### Entity exposed at API boundary (mass assignment risk)
```java
// ❌ — exposes internal fields like 'role', 'passwordHash' to binding
@PostMapping("/users")
public User create(@RequestBody User user) { ... }

// ✅
@PostMapping("/users")
public User create(@RequestBody UserCreateDTO dto) { ... }
```

### Overly broad @Transactional
```java
// ❌ — wraps entire service in one transaction including external calls
@Transactional
public void processAndNotify(Order order) {
    save(order);
    emailService.sendConfirmation(order);  // network call inside TX
}
```
Flag `@Transactional` on methods that include network I/O or long-running operations.

### Field injection (makes testing and tracing harder, not strictly security but flag 🟢)
```java
// ❌
@Autowired
private UserRepository userRepository;

// ✅
private final UserRepository userRepository;

public UserService(UserRepository userRepository) {
    this.userRepository = userRepository;
}
```

### Actuator endpoints exposed without auth
Flag if `management.endpoints.web.exposure.include=*` appears in config without
corresponding security rules.

---

## TypeScript / Node.js

### Unhandled promise rejection
```typescript
// ❌
app.get('/users', async (req, res) => {
  const users = await db.findAll();  // unhandled if throws
  res.json(users);
});

// ✅
app.get('/users', asyncHandler(async (req, res) => {
  const users = await db.findAll();
  res.json(users);
}));
```

### `any` type defeating type safety
```typescript
// ❌
function process(data: any) {
  return data.id;  // no type safety
}

// ✅
function process(data: { id: string; name: string }) {
  return data.id;
}
```

### XSS via innerHTML
```typescript
// ❌
element.innerHTML = userInput;

// ✅
element.textContent = userInput;
// or use DOMPurify if HTML is required
```

### Command injection (Node.js)
```typescript
// ❌
exec(`ffmpeg -i ${userFilename} output.mp4`);

// ✅
execFile('ffmpeg', ['-i', userFilename, 'output.mp4']);
```

### JWT — algorithm confusion
```typescript
// ❌ — accepts 'none' algorithm
jwt.verify(token, secret);

// ✅
jwt.verify(token, secret, { algorithms: ['HS256'] });
```

---

## Go

### Error ignored with `_`
```go
// ❌
result, _ := db.Query("SELECT ...")

// ✅
result, err := db.Query("SELECT ...")
if err != nil {
    return fmt.Errorf("querying users: %w", err)
}
```

### Goroutine leak (channel never closed)
```go
// ❌
ch := make(chan Result)
go func() {
    ch <- doWork()
}()
// ch never read if caller returns early → goroutine leaks
```
Flag goroutines where the channel or done signal has no guaranteed receiver.

### Context not propagated
```go
// ❌
func GetUser(id string) (*User, error) {
    return db.QueryRow("SELECT ...").Scan(...)
}

// ✅
func GetUser(ctx context.Context, id string) (*User, error) {
    return db.QueryRowContext(ctx, "SELECT ...").Scan(...)
}
```

### SQL Injection
```go
// ❌
query := fmt.Sprintf("SELECT * FROM users WHERE id = %s", id)
db.Query(query)

// ✅
db.QueryContext(ctx, "SELECT * FROM users WHERE id = $1", id)
```

### Integer overflow in HTTP parsing
Flag manual parsing of numeric HTTP parameters without bounds checking.

---

## Kotlin

### Null safety bypassed with `!!`
```kotlin
// ❌
val name = user.profile!!.name!!  // panics if null

// ✅
val name = user.profile?.name ?: "Unknown"
```

### Coroutine scope — GlobalScope usage
```kotlin
// ❌ — not tied to lifecycle, leaks
GlobalScope.launch {
    fetchData()
}

// ✅
viewModelScope.launch {  // or appropriate lifecycle scope
    fetchData()
}
```

### Data class used as JPA entity (mutable state / equals issues)
```kotlin
// ❌
@Entity
data class User(var id: Long, var name: String)

// ✅
@Entity
class User {
    @Id var id: Long = 0
    var name: String = ""
}
```

### Hardcoded credentials in companion object
```kotlin
// ❌
companion object {
    const val API_KEY = "abc123"
}

// ✅ — read from environment or config
val apiKey: String = System.getenv("API_KEY") ?: error("API_KEY not set")
```

---

## Cross-stack patterns (always check)

| Pattern | Signal in diff |
|---|---|
| Secret in source | Any string matching `sk-`, `-----BEGIN`, `password =`, `token =` |
| New dependency added | `requirements.txt`, `pom.xml`, `package.json`, `go.mod`, `build.gradle` changed |
| New public endpoint | New route/controller/handler without obvious auth middleware |
| File path from user input | Any `open()`, `File()`, `ReadFile()` with user-derived argument |
| Redirect to user URL | `redirect(user_input)` without allowlist |
