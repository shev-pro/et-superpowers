# TypeScript / Node.js — Language-Specific Review Checks

Load this file during review when the diff contains `.ts`, `.tsx`, or `.js` files.

---

## TypeScript — Code Quality

### `any` type
```typescript
// ❌ — defeats type system
function process(data: any) {
    return data.id;
}

// ✅
function process(data: { id: string; name: string }) {
    return data.id;
}
// or use generics
function process<T extends { id: string }>(data: T): string {
    return data.id;
}
```
Flag every `any` in new code. Use `unknown` + type narrowing if type is genuinely unknown. 🟡

### Type assertion without validation (`as`)
```typescript
// ❌ — no runtime check, will break if shape doesn't match
const user = response.data as User;

// ✅ — validate at boundary (e.g. with zod)
const user = UserSchema.parse(response.data);
```
Flag `as` casts on external data (API responses, DB results, user input). 🟡

### Non-null assertion on possibly-null value
```typescript
// ❌
const name = user.profile!.name!;

// ✅
const name = user.profile?.name ?? 'Unknown';
```

### Implicit `any` from missing return type
```typescript
// ❌ — return type inferred as any if implementation is complex
async function fetchUser(id: string) {
    const res = await api.get(`/users/${id}`);
    return res.data;
}

// ✅
async function fetchUser(id: string): Promise<User> {
    const res = await api.get<User>(`/users/${id}`);
    return res.data;
}
```

### `enum` instead of `const enum` or union types
```typescript
// ❌ — generates runtime object, increases bundle size
enum Status { Active, Inactive }

// ✅ — zero runtime cost
type Status = 'active' | 'inactive';
// or
const Status = { Active: 'active', Inactive: 'inactive' } as const;
type Status = typeof Status[keyof typeof Status];
```
Flag as 🟢.

---

## TypeScript — Async / Promises

### Unhandled promise rejection
```typescript
// ❌ — rejection silently swallowed
async function handler(req, res) {
    const data = await fetchData();  // if throws, crashes process
    res.json(data);
}

// ✅ — wrap with error handler or use express-async-errors
app.get('/data', asyncHandler(async (req, res) => {
    const data = await fetchData();
    res.json(data);
}));
```

### Mixing `.then()` and `async/await`
```typescript
// ❌
async function process() {
    return fetchUser()
        .then(user => {
            return fetchOrders(user.id);
        });
}

// ✅
async function process() {
    const user = await fetchUser();
    return fetchOrders(user.id);
}
```

### `await` in loop (sequential when parallel is possible)
```typescript
// ❌ — O(n) sequential awaits
for (const id of userIds) {
    const user = await fetchUser(id);
    results.push(user);
}

// ✅ — parallel
const results = await Promise.all(userIds.map(id => fetchUser(id)));
```
Note: if order matters and items are independent, `Promise.all` is correct. If rate-limiting is a concern, flag and suggest batching. 🟡

### `Promise.all` without error handling
```typescript
// ❌ — one rejection rejects all
const [users, orders] = await Promise.all([fetchUsers(), fetchOrders()]);

// ✅ — if partial failure is acceptable
const [users, orders] = await Promise.allSettled([fetchUsers(), fetchOrders()]);
```

---

## Node.js — Specific

### Missing `await` on async call
```typescript
// ❌ — fire-and-forget, errors unhandled
async function deleteUser(id: string) {
    db.delete(id);  // not awaited
    return { success: true };
}
```
Flag missing `await` on async functions that mutate state. 🔴

### Synchronous file I/O in request handler
```typescript
// ❌ — blocks event loop
app.get('/config', (req, res) => {
    const config = fs.readFileSync('./config.json', 'utf-8');
    res.json(JSON.parse(config));
});

// ✅
app.get('/config', async (req, res) => {
    const config = await fs.promises.readFile('./config.json', 'utf-8');
    res.json(JSON.parse(config));
});
```

### `process.env` access without validation
```typescript
// ❌ — undefined at runtime if var missing
const dbUrl = process.env.DATABASE_URL;
const client = new Client(dbUrl);  // Client(undefined) silently

// ✅ — validate at startup
const dbUrl = process.env.DATABASE_URL;
if (!dbUrl) throw new Error('DATABASE_URL environment variable is required');
```
Or use a config validation library (zod, envalid). Flag missing env validation at app entry point. 🟡

### EventEmitter — missing error handler
```typescript
// ❌ — unhandled 'error' event crashes process
const emitter = new EventEmitter();
emitter.on('data', handleData);

// ✅
emitter.on('error', (err) => logger.error('Emitter error:', err));
emitter.on('data', handleData);
```

---

## Security (TypeScript/Node.js)

### XSS via `innerHTML` / `dangerouslySetInnerHTML`
```typescript
// ❌
element.innerHTML = userInput;
// React:
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅
element.textContent = userInput;
// React: just render as text
<div>{userInput}</div>
```

### Command injection
```typescript
// ❌
exec(`convert ${userFilename} output.png`);

// ✅
execFile('convert', [userFilename, 'output.png']);
```

### JWT — missing algorithm restriction
```typescript
// ❌
jwt.verify(token, secret);

// ✅
jwt.verify(token, secret, { algorithms: ['HS256'] });
```

### Prototype pollution
```typescript
// ❌ — user input merged into object
Object.assign(config, userInput);
// or
const result = { ...defaultConfig, ...userInput };

// ✅ — validate/allowlist keys first
const safe = pick(userInput, ['theme', 'language']);
const result = { ...defaultConfig, ...safe };
```
Flag when user-controlled object is spread or merged without key filtering. 🔴

### Path traversal in file serving
```typescript
// ❌
app.get('/files/:name', (req, res) => {
    res.sendFile(path.join(__dirname, 'uploads', req.params.name));
});

// ✅
const safeName = path.basename(req.params.name);
const fullPath = path.join(__dirname, 'uploads', safeName);
res.sendFile(fullPath);
```

---

## Performance

### Re-creating objects/functions inside render (React)
```typescript
// ❌ — new function reference on every render
function Component() {
    const handleClick = () => doSomething();  // recreated each render
    return <Button onClick={handleClick} />;
}

// ✅
function Component() {
    const handleClick = useCallback(() => doSomething(), []);
    return <Button onClick={handleClick} />;
}
```
Flag inside components with frequent re-renders. 🟢

### Missing index on frequently queried fields (ORM migrations)
Flag new `findBy*` queries or `WHERE` clauses on columns not covered by an index in migration files.

### Synchronous JSON.parse on large payloads
For payloads potentially > 10MB, flag synchronous `JSON.parse` in request handlers — suggest streaming parser. 🟡

---

## Testing

### `expect.assertions(n)` missing in async tests
```typescript
// ❌ — if promise resolves without entering catch, test passes vacuously
it('should throw on invalid input', async () => {
    try {
        await processInvalid();
    } catch (e) {
        expect(e.message).toBe('Invalid');
    }
});

// ✅
it('should throw on invalid input', async () => {
    expect.assertions(1);
    try {
        await processInvalid();
    } catch (e) {
        expect(e.message).toBe('Invalid');
    }
});
// or cleaner:
await expect(processInvalid()).rejects.toThrow('Invalid');
```

### `jest.mock` hoisting gotchas
Flag `jest.mock` calls that reference variables defined in the same scope — they are hoisted above `import` statements and the variable will be `undefined`.

### `any` in test assertions
```typescript
// ❌
expect(result).toEqual(expect.any(Object));

// ✅
expect(result).toEqual({
    id: expect.any(String),
    name: 'Alice',
    createdAt: expect.any(Date),
});
```
