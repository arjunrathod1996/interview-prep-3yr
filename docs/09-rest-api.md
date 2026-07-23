# 9. REST API Design - Complete Guide

## 📊 Interview Frequency: 100% | Expected Questions: 15

---

## REST Principles

```
REST = Representational State Transfer

Key Principles:
1. Client-Server Architecture
2. Statelessness (each request contains all info)
3. Uniform Interface
4. Cacheability
5. Layered Architecture
6. Code on Demand (optional)
```

---

## HTTP Methods

```
┌─────────┬────────────────────────────┬─────────────────┬──────────────┐
│ Method  │ Purpose                    │ Idempotent?     │ Safe?        │
├─────────┼────────────────────────────┼─────────────────┼──────────────┤
│ GET     │ Retrieve resource          │ Yes             │ Yes (read)   │
│ POST    │ Create new resource        │ No              │ No           │
│ PUT     │ Replace entire resource    │ Yes             │ No           │
│ PATCH   │ Partial update             │ No              │ No           │
│ DELETE  │ Remove resource            │ Yes             │ No           │
│ HEAD    │ Like GET but no body       │ Yes             │ Yes          │
│ OPTIONS │ Describe communication     │ Yes             │ Yes          │
└─────────┴────────────────────────────┴─────────────────┴──────────────┘

Idempotent = Multiple calls produce same result
Safe = Doesn't modify resource
```

---

## HTTP Status Codes

```
2xx Success
├─ 200 OK              - Request successful, returning content
├─ 201 Created         - Resource created successfully
├─ 202 Accepted        - Request accepted for processing
├─ 204 No Content      - Successful but no content to return
└─ 206 Partial Content - Returning part of resource

3xx Redirection
├─ 301 Moved Permanently
├─ 302 Found (temporary redirect)
├─ 304 Not Modified    - Cached response valid
└─ 307 Temporary Redirect

4xx Client Error
├─ 400 Bad Request     - Invalid request syntax
├─ 401 Unauthorized    - Auth required/failed
├─ 403 Forbidden       - Auth OK but access denied
├─ 404 Not Found       - Resource doesn't exist
├─ 405 Method Not Allowed
├─ 409 Conflict        - Request conflicts (duplicate entry)
├─ 410 Gone            - Resource permanently deleted
├─ 422 Unprocessable Entity - Validation failed
├─ 429 Too Many Requests    - Rate limited
└─ 500 Internal Server Error

5xx Server Error
├─ 500 Internal Server Error
├─ 501 Not Implemented
├─ 502 Bad Gateway
├─ 503 Service Unavailable
└─ 504 Gateway Timeout
```

---

## REST API Best Practices

### 1. Versioning:

```java
// Version in URL
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 { }

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 { }

// Version in header
@RestController
@RequestMapping("/api/users")
@ApiVersion("1.0")
public class UserController { }
```

### 2. Naming Conventions:

```
✅ GOOD
GET    /api/users              - List all users
GET    /api/users/123          - Get user by ID
GET    /api/users?age=25       - Filter
GET    /api/users?sort=name    - Sort
GET    /api/users?page=1&size=10 - Paginate
POST   /api/users              - Create user
PUT    /api/users/123          - Replace user
PATCH  /api/users/123          - Partial update
DELETE /api/users/123          - Delete user

❌ BAD
GET    /api/getUsers
GET    /api/getUserById
POST   /api/createUser
POST   /api/deleteUser/123  (should be DELETE)
```

### 3. Request/Response Format:

```java
// Request
POST /api/users
Content-Type: application/json

{
    "name": "John Doe",
    "email": "john@example.com",
    "age": 30
}

// Response 201 Created
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "age": 30,
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z"
}
```

### 4. Error Response:

```java
// Standard Error Response
{
    "timestamp": "2024-01-15T10:30:00Z",
    "status": 404,
    "error": "Not Found",
    "message": "User with ID 999 not found",
    "path": "/api/users/999"
}

// Validation Error Response
{
    "timestamp": "2024-01-15T10:30:00Z",
    "status": 422,
    "error": "Unprocessable Entity",
    "message": "Validation failed",
    "errors": {
        "name": "Name is required",
        "email": "Email should be valid",
        "age": "Age must be at least 18"
    }
}
```

---

## Complete REST API Implementation

```java
// Request DTOs
@Data
@NoArgsConstructor
@AllArgsConstructor
public class CreateUserRequest {
    @NotBlank(message = "Name is required")
    private String name;
    
    @Email(message = "Email should be valid")
    private String email;
    
    @Min(value = 18, message = "Must be 18+")
    private Integer age;
}

@Data
public class UpdateUserRequest {
    private String name;
    private String email;
    private Integer age;
}

// Response DTOs
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserResponse {
    private Long id;
    private String name;
    private String email;
    private Integer age;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}

// Controller
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    // GET all users with pagination
    @GetMapping
    public ResponseEntity<Page<UserResponse>> getAllUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy) {
        
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        Page<UserResponse> users = userService.getAllUsers(pageable);
        return ResponseEntity.ok(users);
    }
    
    // GET user by ID
    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUserById(@PathVariable Long id) {
        UserResponse user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
    
    // CREATE user
    @PostMapping
    public ResponseEntity<UserResponse> createUser(
        @Valid @RequestBody CreateUserRequest request) {
        UserResponse user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
    
    // UPDATE user (full replacement)
    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> updateUser(
        @PathVariable Long id,
        @Valid @RequestBody CreateUserRequest request) {
        UserResponse user = userService.updateUser(id, request);
        return ResponseEntity.ok(user);
    }
    
    // PARTIAL UPDATE
    @PatchMapping("/{id}")
    public ResponseEntity<UserResponse> partialUpdate(
        @PathVariable Long id,
        @RequestBody UpdateUserRequest request) {
        UserResponse user = userService.partialUpdate(id, request);
        return ResponseEntity.ok(user);
    }
    
    // DELETE user
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
    
    // Search
    @GetMapping("/search")
    public ResponseEntity<List<UserResponse>> searchUsers(
        @RequestParam String keyword) {
        List<UserResponse> users = userService.searchUsers(keyword);
        return ResponseEntity.ok(users);
    }
    
    // Bulk delete
    @DeleteMapping
    public ResponseEntity<Void> deleteMultiple(
        @RequestParam List<Long> ids) {
        userService.deleteMultiple(ids);
        return ResponseEntity.noContent().build();
    }
}
```

---

## API Documentation (Swagger/OpenAPI)

```java
@Configuration
public class SwaggerConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("User Management API")
                .version("1.0.0")
                .description("Complete user management system")
                .contact(new Contact()
                    .name("API Support")
                    .email("support@example.com")))
            .servers(Arrays.asList(
                new Server().url("http://localhost:8080").description("Dev"),
                new Server().url("https://api.example.com").description("Prod")
            ));
    }
}

// On Controller/Endpoint
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    @Operation(summary = "Get user by ID", description = "Returns user details")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "User found"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    public ResponseEntity<UserResponse> getUserById(
        @PathVariable @Parameter(description = "User ID") Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }
}

// Access at: http://localhost:8080/swagger-ui.html
```

---

## CORS Configuration

```java
@Configuration
public class CorsConfig {
    
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("http://localhost:3000", "https://example.com")
                    .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH")
                    .allowedHeaders("*")
                    .allowCredentials(true)
                    .maxAge(3600);
            }
        };
    }
}
```

---

## Interview Tips:

✅ **Know the 6 HTTP methods and when to use each**

✅ **Use correct HTTP status codes**

✅ **Design RESTful URLs (nouns, not verbs)**

✅ **Return proper response types (DTO, Page, etc.)**

✅ **Handle errors gracefully with proper error responses**

✅ **Support pagination and filtering**

✅ **Version your APIs**

✅ **Document with Swagger/OpenAPI**

---

📖 **Next Topic:** [SQL & Databases](./10-sql.md)
