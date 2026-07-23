# 7. Spring Boot - Complete Guide

## 📊 Interview Frequency: 100% | Expected Questions: 35+

---

## Spring Boot Lifecycle

```
┌──────────────────────────────────────────────────────┐
│        Application Startup Flow                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  1. SpringApplication.run() called                   │
│     └─ Creates ApplicationContext                    │
│                                                      │
│  2. Auto-Configuration (@EnableAutoConfiguration)   │
│     └─ Scans classpath for spring boot starters     │
│     └─ Auto-configures beans based on dependencies  │
│                                                      │
│  3. Component Scanning (@ComponentScan)             │
│     └─ Finds @Component, @Service, @Repository      │
│     └─ Finds @Configuration classes                 │
│                                                      │
│  4. Bean Creation & Initialization                  │
│     └─ @Bean methods executed                       │
│     └─ Dependency Injection (@Autowired)            │
│     └─ @PostConstruct methods called                │
│                                                      │
│  5. ApplicationContext Ready                        │
│     └─ ApplicationReadyEvent fired                  │
│                                                      │
│  6. Embedded Server Started (Tomcat)                │
│     └─ Listens on configured port (default 8080)    │
│                                                      │
│  ✅ Application Ready to Handle Requests             │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 1️⃣ Spring Boot Basics

### Main Application Class:

```java
@SpringBootApplication  // Includes:
                        // @EnableAutoConfiguration
                        // @ComponentScan
                        // @Configuration
public class UserManagementApp {
    
    public static void main(String[] args) {
        SpringApplication.run(UserManagementApp.class, args);
        // Starts embedded Tomcat on port 8080
        // Scans and registers all components
        // Ready to handle HTTP requests
    }
}
```

### Application Properties:

```properties
# application.properties

# Server Configuration
server.port=8080
server.servlet.context-path=/api

# Actuator (Monitoring)
management.endpoints.web.exposure.include=health,metrics

# Database (Auto-configured with JPA)
spring.datasource.url=jdbc:mysql://localhost:3306/userdb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Logging
logging.level.root=INFO
logging.level.com.myapp=DEBUG
logging.file.name=logs/application.log
```

### Or use application.yml:

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/userdb
    username: root
    password: password
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    root: INFO
    com.myapp: DEBUG
```

---

## 2️⃣ Dependency Injection (DI)

### Constructor Injection (Best Practice):

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // Constructor injection - immutable, testable
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public User createUser(User user) {
        User savedUser = userRepository.save(user);
        emailService.sendWelcomeEmail(savedUser.getEmail());
        return savedUser;
    }
}
```

### Field Injection (Not Recommended):

```java
@Service
public class UserServiceBad {
    @Autowired  // ❌ Mutable, hard to test
    private UserRepository userRepository;
    
    public User createUser(User user) {
        return userRepository.save(user);
    }
}
```

### Setter Injection:

```java
@Service
public class UserServiceSetter {
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository repo) {
        this.userRepository = repo;
    }
}
```

### Using @Qualifier for Multiple Implementations:

```java
// Interface
public interface PaymentProcessor {
    void process(double amount);
}

// Implementation 1
@Component("creditCardProcessor")
public class CreditCardProcessor implements PaymentProcessor {
    @Override
    public void process(double amount) {
        System.out.println("Processing via Credit Card: " + amount);
    }
}

// Implementation 2
@Component("paypalProcessor")
public class PayPalProcessor implements PaymentProcessor {
    @Override
    public void process(double amount) {
        System.out.println("Processing via PayPal: " + amount);
    }
}

// Usage with @Qualifier
@Service
public class PaymentService {
    private final PaymentProcessor creditCardProcessor;
    private final PaymentProcessor paypalProcessor;
    
    public PaymentService(
        @Qualifier("creditCardProcessor") PaymentProcessor cc,
        @Qualifier("paypalProcessor") PaymentProcessor pp) {
        this.creditCardProcessor = cc;
        this.paypalProcessor = pp;
    }
    
    public void payWithCard(double amount) {
        creditCardProcessor.process(amount);
    }
    
    public void payWithPayPal(double amount) {
        paypalProcessor.process(amount);
    }
}
```

---

## 3️⃣ Spring Annotations

### Stereotype Annotations:

```java
// @Component - Generic component
@Component
public class UtilityClass { }

// @Service - Business logic layer
@Service
public class UserService {
    public User createUser(User user) { }
}

// @Repository - Data access layer
@Repository
public class UserRepository extends JpaRepository<User, Long> { }

// @Controller - Web/Request handler
@Controller
public class UserController {
    @GetMapping("/users")
    public String listUsers() { }
}

// @RestController - REST API endpoints
@RestController
@RequestMapping("/api/users")
public class UserRestController {
    @GetMapping
    public List<User> getAllUsers() { }
}
```

### Configuration & Bean Management:

```java
// Define custom beans
@Configuration
public class AppConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    @Bean
    @ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
    public SpecialService specialService() {
        return new SpecialService();
    }
    
    @Bean
    @Primary  // Use this bean if multiple of same type
    public DataSource primaryDataSource() {
        return new DataSource();
    }
}
```

### Scope Annotations:

```java
// Singleton (default) - one instance per application
@Service
@Scope("singleton")
public class SingletonService { }

// Prototype - new instance each time
@Service
@Scope("prototype")
public class PrototypeService { }

// Request - new instance per HTTP request
@Component
@Scope("request")
public class RequestScopedBean { }
```

---

## 4️⃣ REST Controller Example

```java
@RestController
@RequestMapping("/api/users")
@CrossOrigin(origins = "*")  // Allow CORS
public class UserController {
    
    @Autowired
    private UserService userService;
    
    // GET all users
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        List<User> users = userService.getAllUsers();
        return ResponseEntity.ok(users);
    }
    
    // GET user by ID
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        return ResponseEntity.ok(user);
    }
    
    // POST create user
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User created = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    // PUT update user
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
        @PathVariable Long id,
        @Valid @RequestBody User userDetails) {
        User updated = userService.updateUser(id, userDetails);
        return ResponseEntity.ok(updated);
    }
    
    // DELETE user
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
    
    // Query parameters
    @GetMapping("/search")
    public ResponseEntity<List<User>> searchUsers(
        @RequestParam(name = "name", required = false) String name,
        @RequestParam(name = "age", required = false) Integer age) {
        List<User> results = userService.search(name, age);
        return ResponseEntity.ok(results);
    }
}
```

---

## 5️⃣ Exception Handling

### Global Exception Handler:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
        ResourceNotFoundException ex,
        HttpServletRequest request) {
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            request.getRequestURI(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationError(
        MethodArgumentNotValidException ex,
        HttpServletRequest request) {
        
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            errors,
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(
        Exception ex,
        HttpServletRequest request) {
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal server error",
            request.getRequestURI(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}

// Error Response DTO
@Data
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private String path;
    private long timestamp;
}
```

---

## 6️⃣ Validation

```java
@Data
public class User {
    @NotNull(message = "ID cannot be null")
    private Long id;
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    private String name;
    
    @Email(message = "Email should be valid")
    private String email;
    
    @NotNull
    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 100, message = "Age must not exceed 100")
    private Integer age;
    
    @Pattern(regexp = "^[0-9]{10}$", message = "Phone must be 10 digits")
    private String phone;
    
    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;
}

// In Controller
@PostMapping
public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
    // If validation fails, MethodArgumentNotValidException is thrown
    User created = userService.createUser(user);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

---

## 7️⃣ Actuator (Monitoring)

```properties
# Enable actuator endpoints
management.endpoints.web.exposure.include=health,metrics,info,prometheus
management.endpoint.health.show-details=always
management.metrics.export.prometheus.enabled=true
```

### Built-in Endpoints:

```bash
# Health check
GET /actuator/health
# Response: {"status":"UP"}

# Metrics
GET /actuator/metrics
# Response: list of available metrics

# Specific metric
GET /actuator/metrics/jvm.memory.used

# Application info
GET /actuator/info

# Prometheus format (for monitoring)
GET /actuator/prometheus
```

---

## 8️⃣ Profiles (Environment Configuration)

```properties
# application.properties (default)
spring.application.name=user-management

# application-dev.properties
spring.datasource.url=jdbc:mysql://localhost:3306/userdb_dev
logging.level.com.myapp=DEBUG

# application-prod.properties
spring.datasource.url=jdbc:mysql://prod-db:3306/userdb
logging.level.com.myapp=WARN
server.ssl.enabled=true
```

### Activate Profile:

```bash
# Via command line
java -jar app.jar --spring.profiles.active=prod

# Via environment variable
export SPRING_PROFILES_ACTIVE=prod

# In application.properties
spring.profiles.active=dev
```

---

## Real-World Example: Complete User API

```java
// Entity
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    private String name;
    
    @Email
    private String email;
    
    private Integer age;
}

// Repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByNameContainingIgnoreCase(String name);
    Optional<User> findByEmail(String email);
}

// Service
@Service
public class UserService {
    private final UserRepository repo;
    private final EmailService emailService;
    
    public UserService(UserRepository repo, EmailService emailService) {
        this.repo = repo;
        this.emailService = emailService;
    }
    
    public List<User> getAllUsers() {
        return repo.findAll();
    }
    
    public Optional<User> findById(Long id) {
        return repo.findById(id);
    }
    
    public User createUser(User user) {
        User saved = repo.save(user);
        emailService.sendWelcomeEmail(saved.getEmail());
        return saved;
    }
    
    @Transactional
    public User updateUser(Long id, User userDetails) {
        User user = repo.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        user.setName(userDetails.getName());
        user.setEmail(userDetails.getEmail());
        user.setAge(userDetails.getAge());
        return repo.save(user);
    }
    
    public void deleteUser(Long id) {
        repo.deleteById(id);
    }
}

// REST Controller
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        return ResponseEntity.ok(userService.getAllUsers());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(userService.createUser(user));
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
        @PathVariable Long id,
        @Valid @RequestBody User userDetails) {
        return ResponseEntity.ok(userService.updateUser(id, userDetails));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## Interview Tips:

✅ **Explain auto-configuration**

✅ **Show dependency injection with constructor**

✅ **Demonstrate REST controller with proper HTTP status**

✅ **Show exception handling with @RestControllerAdvice**

✅ **Mention application.properties and profiles**

✅ **Know Spring Boot starters**

---

📖 **Next Topic:** [Spring Data JPA & Hibernate](./08-jpa-hibernate.md)
