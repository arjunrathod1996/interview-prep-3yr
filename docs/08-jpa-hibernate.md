# 8. Spring Data JPA & Hibernate - Complete Guide

## 📊 Interview Frequency: 100% | Expected Questions: 20

---

## ORM Concept

```
Object-Oriented World          Relational Database World
        ↓                                    ↓
     Java Object                    SQL Table Rows
     (User.class)      ←→ ORM ←→    (users table)
     
Hibernate handles the mapping between objects and database tables
```

---

## 1️⃣ Entity Mapping

### Basic Entity:

```java
@Entity
@Table(name = "users", indexes = {@Index(name = "idx_email", columnList = "email")})
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "user_name", nullable = false, length = 100)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(name = "age")
    private Integer age;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
    
    @Transient  // Not stored in database
    private String computedField;
}
```

### Inheritance Mapping:

```java
// Single Table Strategy (Default)
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "type")
public abstract class Vehicle {
    @Id
    private Long id;
    private String brand;
}

@Entity
@DiscriminatorValue("CAR")
public class Car extends Vehicle {
    private int doors;
}

@Entity
@DiscriminatorValue("BIKE")
public class Bike extends Vehicle {
    private boolean hasSidecar;
}
```

---

## 2️⃣ Relationships

### One-to-Many / Many-to-One:

```java
// Parent
@Entity
public class Department {
    @Id
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Employee> employees = new ArrayList<>();
}

// Child
@Entity
public class Employee {
    @Id
    private Long id;
    
    private String name;
    
    @ManyToOne(fetch = FetchType.LAZY)  // Lazy load department
    @JoinColumn(name = "department_id")
    private Department department;
}

// Usage
Department dept = new Department();
dept.setName("IT");

Employee emp1 = new Employee();
emp1.setName("John");
emp1.setDepartment(dept);

Employee emp2 = new Employee();
emp2.setName("Jane");
emp2.setDepartment(dept);

dept.getEmployees().add(emp1);
dept.getEmployees().add(emp2);

employeeRepository.save(emp1);  // Saves both due to cascade
```

### Many-to-Many:

```java
// Student
@Entity
public class Student {
    @Id
    private Long id;
    private String name;
    
    @ManyToMany(cascade = CascadeType.PERSIST)
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}

// Course
@Entity
public class Course {
    @Id
    private Long id;
    private String title;
    
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}

// Usage
Student student = new Student();
student.setName("Alice");

Course course1 = new Course();
course1.setTitle("Java");

Course course2 = new Course();
course2.setTitle("Spring Boot");

student.getCourses().add(course1);
student.getCourses().add(course2);

studentRepository.save(student);
```

### One-to-One:

```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;
    
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id", unique = true)
    private UserProfile profile;
}

@Entity
public class UserProfile {
    @Id
    private Long id;
    private String bio;
    private String profilePicUrl;
    
    @OneToOne(mappedBy = "profile")
    private User user;
}
```

---

## 3️⃣ JpaRepository Query Methods

### Automatic Query Methods:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Naming convention generates queries automatically
    
    // Find by single field
    User findByEmail(String email);
    Optional<User> findById(Long id);
    
    // Find multiple with condition
    List<User> findByNameContaining(String name);
    List<User> findByNameContainingIgnoreCase(String name);
    List<User> findByAgeGreaterThan(int age);
    List<User> findByAgeLessThan(int age);
    List<User> findByAgeBetween(int minAge, int maxAge);
    
    // And, Or conditions
    List<User> findByNameAndAge(String name, int age);
    List<User> findByNameOrEmail(String name, String email);
    
    // Check existence
    boolean existsByEmail(String email);
    
    // Delete
    void deleteByEmail(String email);
    long deleteByAgeGreaterThan(int age);
    
    // Count
    long countByAgeGreaterThan(int age);
    
    // Ordering
    List<User> findByAgeGreaterThanOrderByNameAsc(int age);
}
```

### Custom @Query Annotations:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // JPQL (Database-independent)
    @Query("SELECT u FROM User u WHERE u.email = ?1")
    User findUserByEmail(String email);
    
    // With named parameters
    @Query("SELECT u FROM User u WHERE u.name = :name AND u.age > :age")
    List<User> findByNameAndAgeGreaterThan(
        @Param("name") String name,
        @Param("age") int age
    );
    
    // Native SQL
    @Query(value = "SELECT * FROM users WHERE age > :age", nativeQuery = true)
    List<User> findUsersOlderThan(@Param("age") int age);
    
    // Update with @Modifying
    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.name = :name WHERE u.id = :id")
    void updateUserName(@Param("id") Long id, @Param("name") String name);
    
    // Delete
    @Modifying
    @Transactional
    @Query("DELETE FROM User u WHERE u.age > :age")
    void deleteUsersOlderThan(@Param("age") int age);
}
```

---

## 4️⃣ Lazy vs Eager Loading

### Lazy Loading (Default for @ManyToOne, @OneToMany):

```java
// Problem: N+1 Query
List<User> users = userRepository.findAll();
for (User user : users) {
    // Each iteration triggers new query to load department
    System.out.println(user.getDepartment().getName());
    // If 1000 users: 1 query for users + 1000 queries for departments!
}

// Solution 1: Use JOIN FETCH
@Query("SELECT u FROM User u LEFT JOIN FETCH u.department")
List<User> findAllWithDepartment();

// Solution 2: Use @EntityGraph
@EntityGraph(attributePaths = {"department"})
@Query("SELECT u FROM User u")
List<User> findAllWithDepartment();
```

### Eager Loading:

```java
@Entity
public class User {
    @Id
    private Long id;
    
    // Load department immediately
    @ManyToOne(fetch = FetchType.EAGER)
    private Department department;
}

// Caveat: Always loads even if not needed
// Can lead to performance issues with many relationships
```

---

## 5️⃣ Pagination & Sorting

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Paginated query
    Page<User> findAll(Pageable pageable);
    Page<User> findByAgeGreaterThan(int age, Pageable pageable);
    
    // Sorted query
    List<User> findAll(Sort sort);
    List<User> findByAgeGreaterThan(int age, Sort sort);
}

// Usage
PageRequest pageRequest = PageRequest.of(0, 10, Sort.by("name").ascending());
Page<User> page = userRepository.findAll(pageRequest);

System.out.println("Total elements: " + page.getTotalElements());
System.out.println("Total pages: " + page.getTotalPages());
System.out.println("Current page: " + page.getNumber());
System.out.println("Page size: " + page.getSize());

List<User> content = page.getContent();
for (User user : content) {
    System.out.println(user.getName());
}

// Multiple sort criteria
Sort sort = Sort.by(
    new Sort.Order(Sort.Direction.ASC, "name"),
    new Sort.Order(Sort.Direction.DESC, "age")
);
List<User> users = userRepository.findAll(sort);
```

---

## 6️⃣ Cascade & Orphan Removal

```java
@Entity
public class Post {
    @Id
    private Long id;
    
    private String title;
    
    // All operations cascade to comments
    // When post deleted, comments also deleted
    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();
}

@Entity
public class Comment {
    @Id
    private Long id;
    
    private String text;
    
    @ManyToOne
    private Post post;
}

// Usage
Post post = new Post();
post.setTitle("Spring Boot Guide");

Comment c1 = new Comment();
c1.setText("Great post!");
c1.setPost(post);

Comment c2 = new Comment();
c2.setText("Thanks for sharing");
c2.setPost(post);

post.getComments().add(c1);
post.getComments().add(c2);

postRepository.save(post);  // Saves post and all comments

// Removing from collection also deletes from DB
post.getComments().remove(c1);
postRepository.save(post);  // Deletes c1 from database
```

---

## Real-World Example: Blog System

```java
// Entity
@Entity
public class Article {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    private String title;
    
    @Lob
    private String content;
    
    @ManyToOne
    @JoinColumn(name = "author_id")
    private User author;
    
    @OneToMany(mappedBy = "article", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();
    
    @ManyToMany
    @JoinTable(
        name = "article_tag",
        joinColumns = @JoinColumn(name = "article_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private Set<Tag> tags = new HashSet<>();
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}

// Repository
public interface ArticleRepository extends JpaRepository<Article, Long> {
    
    @EntityGraph(attributePaths = {"author", "comments", "tags"})
    Page<Article> findAll(Pageable pageable);
    
    @Query("SELECT a FROM Article a WHERE LOWER(a.title) LIKE LOWER(CONCAT('%', :keyword, '%'))")
    List<Article> searchByTitle(@Param("keyword") String keyword);
    
    List<Article> findByAuthorId(Long authorId, Pageable pageable);
    
    List<Article> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);
}

// Service
@Service
@Transactional
public class ArticleService {
    
    @Autowired
    private ArticleRepository articleRepository;
    
    public Page<Article> getAllArticles(int page, int size) {
        return articleRepository.findAll(
            PageRequest.of(page, size, Sort.by("createdAt").descending())
        );
    }
    
    public Article getArticleById(Long id) {
        return articleRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Article not found"));
    }
    
    public Article createArticle(Article article) {
        return articleRepository.save(article);
    }
    
    public Article updateArticle(Long id, Article articleDetails) {
        Article article = getArticleById(id);
        article.setTitle(articleDetails.getTitle());
        article.setContent(articleDetails.getContent());
        return articleRepository.save(article);
    }
}
```

---

## Hibernate Best Practices:

✅ **Use constructor injection for repositories**

✅ **Mark repositories with @Repository**

✅ **Use @Transactional on service layer, not controller**

✅ **Avoid N+1 queries with @EntityGraph or JOIN FETCH**

✅ **Use Lazy loading for @ManyToOne, @OneToMany**

✅ **Use @Modifying @Transactional for bulk updates**

✅ **Create indexes on frequently queried columns**

✅ **Use Pagination for large result sets**

---

📖 **Next Topic:** [REST API Design](./09-rest-api.md)
