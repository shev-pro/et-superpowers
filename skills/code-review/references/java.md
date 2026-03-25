# Java / Spring — Language-Specific Review Checks

Load this file during review when the diff contains `.java` files.

---

## Code Quality

### Optional misuse
```java
// ❌ — Optional.get() without isPresent() check
User user = userRepo.findById(id).get();

// ❌ — returning null instead of empty Optional
public Optional<User> findUser(Long id) {
    if (...) return null;
}

// ✅
User user = userRepo.findById(id)
    .orElseThrow(() -> new UserNotFoundException(id));
```

### Checked exceptions swallowed
```java
// ❌
try {
    riskyOperation();
} catch (Exception e) {
    // TODO: handle
}

// ✅
try {
    riskyOperation();
} catch (SpecificException e) {
    log.error("Operation failed for id {}: {}", id, e.getMessage());
    throw new ServiceException("Operation failed", e);
}
```

### String concatenation in loop
```java
// ❌
String result = "";
for (String s : list) {
    result += s;  // O(n²)
}

// ✅
StringBuilder sb = new StringBuilder();
for (String s : list) {
    sb.append(s);
}
String result = sb.toString();
```

### Raw types
```java
// ❌
List items = new ArrayList();
Map map = new HashMap();

// ✅
List<Item> items = new ArrayList<>();
Map<String, Item> map = new HashMap<>();
```

### equals/hashCode missing on entity
Flag any `@Entity` class without overriding `equals` and `hashCode`, especially when used in Sets or as Map keys.

---

## Spring-Specific

### Field injection instead of constructor injection
```java
// ❌
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}

// ✅
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```
Constructor injection enables immutability, easier testing, and clear dependency graph. Flag as 🟡.

### @Transactional on wrong layer or too broad
```java
// ❌ — on controller
@Transactional
@GetMapping("/users")
public List<User> getUsers() { ... }

// ❌ — wraps external HTTP call
@Transactional
public void processAndNotify(Order order) {
    orderRepo.save(order);
    emailClient.send(order);  // network call inside TX
}
```

### Entity exposed at API boundary
```java
// ❌ — User entity directly as request/response body
@PostMapping("/users")
public User createUser(@RequestBody User user) { ... }

// ✅
@PostMapping("/users")
public UserResponse createUser(@RequestBody UserCreateRequest request) { ... }
```
Flag missing DTO layer. Exposes internal fields and risks mass assignment. 🔴 if sensitive fields present.

### Missing `@Valid` on request body
```java
// ❌ — validation annotations on DTO are ignored
@PostMapping("/users")
public UserResponse create(@RequestBody UserCreateRequest request) { ... }

// ✅
@PostMapping("/users")
public UserResponse create(@Valid @RequestBody UserCreateRequest request) { ... }
```

### `@SpringBootTest` for unit tests
```java
// ❌ — loads full context for a simple unit test (slow)
@SpringBootTest
class UserServiceTest { ... }

// ✅ — for unit tests
class UserServiceTest {
    private UserService userService = new UserService(mockRepo);
}

// ✅ — for slice tests
@WebMvcTest(UserController.class)
class UserControllerTest { ... }
```

### Actuator endpoints exposed without security
Flag `management.endpoints.web.exposure.include=*` in `application.properties` / `application.yml` without corresponding Spring Security configuration.

---

## Security (Java-specific)

### SQL injection via JPQL string concatenation
```java
// ❌
String jpql = "SELECT u FROM User u WHERE u.name = '" + name + "'";
em.createQuery(jpql).getResultList();

// ✅
em.createQuery("SELECT u FROM User u WHERE u.name = :name", User.class)
    .setParameter("name", name)
    .getResultList();
```

### Deserialization of untrusted data
Flag `ObjectInputStream.readObject()` on data from external sources (HTTP body, files, queues). 🔴

### XXE in XML parsing
```java
// ❌ — default DocumentBuilderFactory allows XXE
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();

// ✅
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
```

### Hardcoded credentials in application.properties
```properties
# ❌
spring.datasource.password=mysecretpassword
jwt.secret=hardcoded-secret

# ✅ — use env vars or external secrets manager
spring.datasource.password=${DB_PASSWORD}
jwt.secret=${JWT_SECRET}
```

---

## Performance

### N+1 with JPA / Hibernate
```java
// ❌ — LAZY fetch triggers N queries in loop
List<Order> orders = orderRepo.findAll();
orders.forEach(o -> System.out.println(o.getItems().size()));

// ✅ — fetch join or @EntityGraph
@Query("SELECT o FROM Order o JOIN FETCH o.items")
List<Order> findAllWithItems();
```

### FetchType.EAGER as default
Flag `@OneToMany(fetch = FetchType.EAGER)` — loads entire collection on every parent fetch, regardless of need. Suggest LAZY + explicit fetch when needed. 🟡

### Stream collect then stream again
```java
// ❌
List<String> names = users.stream()
    .map(User::getName)
    .collect(Collectors.toList())
    .stream()    // redundant re-stream
    .filter(...)
    .collect(Collectors.toList());

// ✅ — single pipeline
List<String> names = users.stream()
    .map(User::getName)
    .filter(...)
    .collect(Collectors.toList());
```

### Missing pagination on repository method returning `List`
```java
// ❌ — could return millions of rows
List<Order> findByStatus(OrderStatus status);

// ✅
Page<Order> findByStatus(OrderStatus status, Pageable pageable);
```

---

## Testing

### Test method not annotated with `@Test`
Easily missed in JUnit 5 migration from JUnit 4 — test silently not run.

### Mockito `verify` without `times`
```java
// ❌ — passes even if called 100 times
verify(emailService).send(any());

// ✅
verify(emailService, times(1)).send(expectedEmail);
```

### Using `Thread.sleep` in tests
Flag any `Thread.sleep` in tests — indicates missing async handling. Suggest `Awaitility` or `CompletableFuture.get()` with timeout.

### Test data bleeding between tests
Flag static mutable fields or missing `@BeforeEach` / `@AfterEach` cleanup in test classes that share state.
