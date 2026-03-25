# Python / FastAPI — Language-Specific Review Checks

Load this file during review when the diff contains `.py` files.

---

## Code Quality

### Type hints
```python
# ❌ — no type hints on public functions
def get_user(user_id):
    ...

# ✅
def get_user(user_id: int) -> User | None:
    ...
```
Flag missing type hints on all public functions/methods. Private helpers (`_foo`) are lower priority (🟢).

### Pydantic v2 idioms
```python
# ❌ — v1 API, breaks in v2
user.dict()
user.json()

# ✅ — v2
user.model_dump()
user.model_dump_json()
```
Also flag `validator` decorator → should be `field_validator` in v2.

### Mutable default arguments
```python
# ❌ — shared across all calls
def append_item(item, lst=[]):
    lst.append(item)
    return lst

# ✅
def append_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

### Exception handling too broad
```python
# ❌
try:
    process()
except Exception:
    pass  # swallows everything

# ✅
try:
    process()
except ValueError as e:
    logger.warning("Invalid input: %s", e)
    raise
```

### f-string in logger (performance)
```python
# ❌ — string built even if log level disabled
logger.debug(f"Processing user {user_id}")

# ✅
logger.debug("Processing user %s", user_id)
```

---

## FastAPI-Specific

### Missing response_model
```python
# ❌ — leaks internal fields (e.g. password_hash)
@app.get("/users/{id}")
async def get_user(id: int):
    return db.query(User).get(id)

# ✅
@app.get("/users/{id}", response_model=UserResponse)
async def get_user(id: int):
    return db.query(User).get(id)
```

### Dependency injection — not using `Depends`
```python
# ❌ — manual session management
@app.get("/items")
async def list_items():
    db = SessionLocal()
    try:
        return db.query(Item).all()
    finally:
        db.close()

# ✅
@app.get("/items")
async def list_items(db: Session = Depends(get_db)):
    return db.query(Item).all()
```

### Background tasks blocking endpoint
```python
# ❌
@app.post("/notify")
async def notify(user_id: int):
    send_email(user_id)  # slow, blocks response
    return {"status": "ok"}

# ✅
@app.post("/notify")
async def notify(user_id: int, background_tasks: BackgroundTasks):
    background_tasks.add_task(send_email, user_id)
    return {"status": "ok"}
```

### Missing status codes on write operations
```python
# ❌
@app.post("/users")
async def create_user(data: UserCreate): ...

# ✅
@app.post("/users", status_code=status.HTTP_201_CREATED)
async def create_user(data: UserCreate): ...
```

---

## Security (Python-specific)

### Pickle deserialization
```python
# ❌ — arbitrary code execution if data is attacker-controlled
obj = pickle.loads(user_data)
```
Flag any `pickle.loads` / `pickle.load` receiving external data. 🔴

### Shell injection via subprocess
```python
# ❌
subprocess.run(f"convert {filename}", shell=True)

# ✅
subprocess.run(["convert", filename], shell=False)
```

### Path traversal
```python
# ❌
with open(f"/uploads/{user_filename}") as f: ...

# ✅
safe_path = Path("/uploads") / Path(user_filename).name
with open(safe_path) as f: ...
```

### Timing attack on secret comparison
```python
# ❌
if token == expected_token:

# ✅
import hmac
if hmac.compare_digest(token, expected_token):
```

---

## Performance

### N+1 with SQLAlchemy
```python
# ❌
users = db.query(User).all()
for user in users:
    print(user.orders)  # one query per user

# ✅
users = db.query(User).options(selectinload(User.orders)).all()
```

### Missing `__slots__` on hot-path dataclass alternatives
For classes instantiated millions of times, suggest `__slots__` or `dataclass(slots=True)` (Python 3.10+). Flag as 🟢.

### Unbounded list comprehension on large dataset
```python
# ❌ — loads all into memory
results = [process(x) for x in db.query(Item).all()]

# ✅ — generator or paginated query
results = (process(x) for x in db.query(Item).yield_per(100))
```

---

## Testing

### `assert` in production code (not tests)
```python
# ❌
def divide(a, b):
    assert b != 0, "b must not be zero"  # stripped with -O flag
    return a / b

# ✅
def divide(a, b):
    if b == 0:
        raise ValueError("b must not be zero")
    return a / b
```

### Pytest fixture scope mismatch
Flag `@pytest.fixture` with `scope="session"` on mutable objects — causes test pollution.

### Mocking too broadly
```python
# ❌ — patches entire module, hides import errors
with patch("myapp.services"):
    ...

# ✅ — patch only the specific name used
with patch("myapp.services.user_service.send_email"):
    ...
```
