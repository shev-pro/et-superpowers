# Kotlin — Language-Specific Review Checks

Load this file during review when the diff contains `.kt` files.

---

## Code Quality

### Null safety bypassed with `!!`
```kotlin
// ❌ — throws NullPointerException if null
val name = user.profile!!.name!!

// ✅
val name = user.profile?.name ?: "Unknown"
// or fail fast with a clear message
val name = user.profile?.name
    ?: error("User ${user.id} has no profile")
```
Flag every `!!` — each one is a potential crash. 🟡 (🔴 if on user-controlled input or in hot path).

### `let` / `run` / `apply` / `also` misuse
```kotlin
// ❌ — nested lets reduce readability
user.address?.let { address ->
    address.city?.let { city ->
        city.zipCode?.let { zip -> process(zip) }
    }
}

// ✅
val zip = user.address?.city?.zipCode ?: return
process(zip)
```

### `var` instead of `val`
```kotlin
// ❌
var userId = getUserId()  // never reassigned
var config = loadConfig()

// ✅
val userId = getUserId()
val config = loadConfig()
```
Flag `var` where value is never reassigned after initialization. 🟢

### Data class with mutable properties in collections
```kotlin
// ❌ — mutable data class in a Set or as Map key: equals/hashCode changes
data class Tag(var name: String)
val tags = mutableSetOf(Tag("kotlin"))
tags.first().name = "java"  // set is now broken

// ✅
data class Tag(val name: String)
```

### `when` without `else` on non-sealed type
```kotlin
// ❌ — silently does nothing for unhandled cases
when (status) {
    Status.ACTIVE -> activate()
    Status.INACTIVE -> deactivate()
    // new enum values silently ignored
}

// ✅
when (status) {
    Status.ACTIVE -> activate()
    Status.INACTIVE -> deactivate()
    else -> throw IllegalStateException("Unhandled status: $status")
}
```
Prefer sealed classes/interfaces over enums when exhaustiveness matters.

---

## Coroutines

### GlobalScope usage
```kotlin
// ❌ — not tied to any lifecycle, leaks if caller is destroyed
GlobalScope.launch {
    fetchData()
}

// ✅ — use appropriate scope
viewModelScope.launch { ... }      // ViewModel (Android)
lifecycleScope.launch { ... }      // Activity/Fragment (Android)
coroutineScope { ... }             // structured concurrency in suspend fun
```

### Blocking call inside coroutine
```kotlin
// ❌ — blocks thread, defeats coroutine purpose
suspend fun fetchUser(id: Long): User {
    Thread.sleep(1000)       // blocks dispatcher thread
    return userRepo.findById(id)  // if synchronous JDBC
}

// ✅
suspend fun fetchUser(id: Long): User = withContext(Dispatchers.IO) {
    userRepo.findById(id)
}
```

### Missing `Dispatchers.IO` for blocking I/O
Any `suspend` function that calls blocking JDBC, file I/O, or legacy synchronous HTTP without `withContext(Dispatchers.IO)`. Flag as 🟡.

### Unhandled exceptions in `launch`
```kotlin
// ❌ — exception silently kills coroutine
scope.launch {
    riskyOperation()
}

// ✅
val handler = CoroutineExceptionHandler { _, e ->
    logger.error("Coroutine failed", e)
}
scope.launch(handler) {
    riskyOperation()
}
// or use async/await to propagate exceptions explicitly
```

### `runBlocking` in production code
```kotlin
// ❌ — blocks thread; acceptable in tests/main, not in services
fun getUser(id: Long) = runBlocking {
    userService.findById(id)
}
```
Flag `runBlocking` outside of `fun main` or test classes. 🟡

---

## Spring / Backend (Kotlin)

### Data class as JPA entity
```kotlin
// ❌ — data class generates equals/hashCode based on all fields,
//      conflicts with JPA proxy behavior
@Entity
data class User(
    @Id val id: Long,
    val name: String
)

// ✅
@Entity
class User {
    @Id
    var id: Long = 0
    var name: String = ""

    override fun equals(other: Any?): Boolean { ... }
    override fun hashCode(): Int { ... }
}
```

### Open classes required for Spring proxying
```kotlin
// ❌ — Kotlin classes are final by default; Spring CGLIB proxy fails
@Service
class UserService { ... }

// ✅ — use kotlin-spring plugin (applies open automatically)
//      or annotate manually
@Service
open class UserService { ... }
```
Check that `kotlin-spring` (allopen) Gradle/Maven plugin is configured — if not, flag all `@Service`, `@Component`, `@Repository`, `@Controller` classes. 🔴

### Companion object for constants (prefer `object` or top-level)
```kotlin
// ❌ — companion adds overhead and generates extra class
class UserService {
    companion object {
        const val MAX_USERS = 1000
    }
}

// ✅ — top-level constant
const val MAX_USERS = 1000
```

---

## Security (Kotlin-specific)

### Hardcoded secrets in companion/object
```kotlin
// ❌
object Config {
    const val API_KEY = "sk-abc123"
    const val DB_PASSWORD = "secret"
}

// ✅
object Config {
    val apiKey: String = System.getenv("API_KEY")
        ?: error("API_KEY environment variable not set")
}
```

### `!!` on user-controlled input
```kotlin
// ❌ — attacker can send null/missing field, causes 500
val userId = request.getHeader("X-User-Id")!!.toLong()

// ✅
val userId = request.getHeader("X-User-Id")?.toLongOrNull()
    ?: throw BadRequestException("Missing X-User-Id header")
```

---

## Performance

### `toString()` on large object in string template (always evaluated)
```kotlin
// ❌ — object.toString() called even if DEBUG log disabled
logger.debug("Processing $largeObject")

// ✅ — lazy evaluation
logger.debug { "Processing $largeObject" }  // SLF4J Kotlin extensions
```

### `filter + map` vs `mapNotNull`
```kotlin
// ❌
val names = users
    .filter { it.name != null }
    .map { it.name!! }

// ✅
val names = users.mapNotNull { it.name }
```

### `List` operations creating unnecessary intermediate lists
```kotlin
// ❌ — each step creates a new list
val result = users
    .filter { it.active }
    .map { it.name }
    .filter { it.length > 3 }

// ✅ — use sequences for large collections
val result = users.asSequence()
    .filter { it.active }
    .map { it.name }
    .filter { it.length > 3 }
    .toList()
```
Flag for collections likely > 1000 elements. 🟢

---

## Testing

### `runBlocking` vs `runTest` in coroutine tests
```kotlin
// ❌ — doesn't handle TestCoroutineScheduler, delays not skipped
@Test
fun testSuspendFn() = runBlocking {
    val result = mySuspendFn()
    assertEquals(expected, result)
}

// ✅
@Test
fun testSuspendFn() = runTest {
    val result = mySuspendFn()
    assertEquals(expected, result)
}
```

### Missing `MockK` relaxed mock for unused interactions
```kotlin
// ❌ — every call must be stubbed even if irrelevant to test
val repo = mockk<UserRepository>()

// ✅ — when you don't care about most interactions
val repo = mockk<UserRepository>(relaxed = true)
```

### Hardcoded delays in tests
Flag `delay(1000)` or `Thread.sleep` in test bodies — use `runTest` with `advanceTimeBy` instead.
