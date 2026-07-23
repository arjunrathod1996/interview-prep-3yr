# 6. Java 8 & Streams - Complete Guide

## 📊 Interview Frequency: 100% | Expected Questions: 20

---

## Key Java 8 Features

```
┌─────────────────────────────────────┐
│    Java 8 Major Features            │
├─────────────────────────────────────┤
│  1. Lambda Expressions              │
│  2. Functional Interfaces           │
│  3. Streams API                     │
│  4. Default Methods in Interfaces   │
│  5. Method References               │
│  6. Optional Class                  │
└─────────────────────────────────────┘
```

---

## 1️⃣ Lambda Expressions

### What is Lambda?
**Anonymous function with minimal syntax. Maps functional interface to inline implementation.**

### Syntax:
```
(parameters) -> body

Example:
(x, y) -> x + y
(name) -> System.out.println(name)
() -> return 42
```

### Before & After:

```java
// ❌ Before Java 8 (Anonymous Inner Class)
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("Button clicked");
    }
});

// ✅ Java 8 Lambda
button.setOnClickListener(v -> System.out.println("Button clicked"));

// ✅ More complex lambda
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.forEach(n -> System.out.println(n * 2));
```

### Different Lambda Forms:

```java
// No parameter
Supplier<Integer> random = () -> new Random().nextInt(100);

// Single parameter (no parentheses needed)
Consumer<String> print = name -> System.out.println(name);

// Multiple parameters
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// Multiple statements (need braces and return)
Function<Integer, Integer> square = n -> {
    System.out.println("Squaring: " + n);
    return n * n;
};
```

---

## 2️⃣ Functional Interfaces

### What is a Functional Interface?
**An interface with exactly ONE abstract method. Can be implemented using lambda.**

### Built-in Functional Interfaces:

```
┌─────────────────────────────────────────────────────┐
│  Functional Interface  │  Method           │ Usage   │
├────────────────────────────────────────────────────┤
│  Supplier<T>          │  T get()          │ Produce │
│  Consumer<T>          │  void accept(T)   │ Consume │
│  Function<T, R>       │  R apply(T)       │ Convert │
│  Predicate<T>         │  boolean test(T)  │ Filter  │
│  BiFunction<T,U,R>    │  R apply(T,U)     │ 2-param │
│  BiConsumer<T,U>      │  void accept(T,U) │ 2-param │
└─────────────────────────────────────────────────────┘
```

### Examples:

```java
// Supplier - provides value
Supplier<String> greeting = () -> "Hello, World!";
System.out.println(greeting.get());  // Hello, World!

// Consumer - accepts value
Consumer<String> greet = name -> System.out.println("Hi " + name);
greet.accept("John");  // Hi John

// Function - transform
Function<String, Integer> strlen = String::length;
System.out.println(strlen.apply("Hello"));  // 5

// Predicate - test condition
Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4));  // true

// BiFunction - two inputs
BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;
System.out.println(multiply.apply(3, 4));  // 12

// Custom Functional Interface
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
}

Calculator add = (a, b) -> a + b;
Calculator multiply2 = (a, b) -> a * b;
System.out.println(add.calculate(5, 3));  // 8
```

---

## 3️⃣ Method References

### Shorthand for Lambda:

```
Type of Method Reference   │   Syntax        │  Example
──────────────────────────────────────────────────────
Static method             │  Class::method  │  Integer::parseInt
Instance method           │  obj::method    │  str::toUpperCase
Constructor               │  Class::new     │  ArrayList::new
Arbitrary instance method │  Class::method  │  String::compareTo
```

### Examples:

```java
// Static method reference
Function<String, Integer> parseInt = Integer::parseInt;
System.out.println(parseInt.apply("123"));  // 123

// Instance method reference
String str = "hello";
Supplier<String> upper = str::toUpperCase;
System.out.println(upper.get());  // HELLO

// Constructor reference
Supplier<List<String>> listFactory = ArrayList::new;
List<String> list = listFactory.get();

// Arbitrary instance method
Comparator<String> comp = String::compareTo;
System.out.println(comp.compare("a", "b"));  // -1

// In streams
List<String> names = Arrays.asList("john", "jane", "bob");
names.stream()
    .map(String::toUpperCase)  // Method reference
    .forEach(System.out::println);  // JOHN, JANE, BOB
```

---

## 4️⃣ Streams API

### What is a Stream?
**Sequence of elements supporting sequential and parallel operations. Lazy evaluation.**

### Stream Pipeline:
```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐     ┌────────────┐
│   Source    │────►│ Intermediate │────►│ Intermediate │────►│ Terminal   │
│             │     │ Operations   │     │ Operations   │     │ Operation  │
│ List/Array  │     │ filter, map  │     │ sorted, skip │     │ forEach    │
└─────────────┘     └──────────────┘     └──────────────┘     └────────────┘
                           ↓                     ↓                   ↓
                    (Lazy - not executed) (Lazy) ...        (Eager - executes)
```

### Creating Streams:

```java
// From Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();

// From Array
Stream<String> stream2 = Stream.of("a", "b", "c");

// Generate stream
Stream<Integer> stream3 = Stream.generate(() -> 1).limit(5);

// Iterate stream
Stream<Integer> stream4 = Stream.iterate(0, n -> n + 1).limit(5);

// Parallel stream
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.parallelStream()
    .map(n -> n * 2)
    .forEach(System.out::println);
```

### Intermediate Operations (Lazy):

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// filter - keep elements matching predicate
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// [2, 4, 6, 8, 10]

// map - transform each element
List<Integer> squared = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());
// [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

// flatMap - map + flatten
List<String> words = Arrays.asList("hello", "world");
List<String> letters = words.stream()
    .flatMap(word -> Arrays.stream(word.split("")))
    .collect(Collectors.toList());
// [h, e, l, l, o, w, o, r, l, d]

// distinct - remove duplicates
List<Integer> unique = Arrays.asList(1, 1, 2, 2, 3, 3).stream()
    .distinct()
    .collect(Collectors.toList());
// [1, 2, 3]

// sorted - sort elements
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());

// sorted with comparator
List<Integer> reverse = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());

// skip - skip first n elements
List<Integer> skipFirst3 = numbers.stream()
    .skip(3)
    .collect(Collectors.toList());
// [4, 5, 6, 7, 8, 9, 10]

// limit - take first n elements
List<Integer> first3 = numbers.stream()
    .limit(3)
    .collect(Collectors.toList());
// [1, 2, 3]

// peek - perform action without changing stream
List<Integer> result = numbers.stream()
    .filter(n -> n > 5)
    .peek(n -> System.out.println("Filtered: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("Mapped: " + n))
    .collect(Collectors.toList());
```

### Terminal Operations (Eager):

```java
// collect - gather into collection
List<Integer> list = numbers.stream()
    .filter(n -> n > 5)
    .collect(Collectors.toList());

// toSet, toMap
Set<Integer> set = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toSet());

// forEach - apply function to each element
numbers.stream()
    .forEach(System.out::println);

// reduce - combine elements
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);  // 55

int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);  // 3628800

// min/max
Optional<Integer> min = numbers.stream().min(Comparator.naturalOrder());
Optional<Integer> max = numbers.stream().max(Comparator.naturalOrder());

// count
long count = numbers.stream()
    .filter(n -> n > 5)
    .count();  // 5

// anyMatch/allMatch/noneMatch
boolean hasEven = numbers.stream().anyMatch(n -> n % 2 == 0);
boolean allPositive = numbers.stream().allMatch(n -> n > 0);
boolean noNegative = numbers.stream().noneMatch(n -> n < 0);

// findFirst/findAny
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 5)
    .findFirst();

Optional<Integer> any = numbers.parallelStream()
    .filter(n -> n > 5)
    .findAny();
```

### Advanced Collectors:

```java
// groupingBy
Map<String, List<Person>> byDept = people.stream()
    .collect(Collectors.groupingBy(p -> p.department));
// {IT: [John, Bob], HR: [Jane]}

// groupingBy with counting
Map<String, Long> deptCount = people.stream()
    .collect(Collectors.groupingBy(
        p -> p.department,
        Collectors.counting()
    ));
// {IT: 2, HR: 1}

// groupingBy with averaging
Map<String, Double> avgSalary = people.stream()
    .collect(Collectors.groupingBy(
        p -> p.department,
        Collectors.averagingDouble(p -> p.salary)
    ));

// partitioningBy
Map<Boolean, List<Integer>> evenOdd = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
// {true: [2,4,6], false: [1,3,5]}

// toMap
Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(
        p -> p.name,
        p -> p.age
    ));

// joining
String csv = people.stream()
    .map(p -> p.name)
    .collect(Collectors.joining(", "));
// "John, Jane, Bob"
```

---

## 5️⃣ Optional Class

### Avoid NullPointerException:

```java
// ❌ Without Optional
public User findUser(String id) {
    User user = userRepository.findById(id);
    if (user != null) {
        return user;
    }
    return null;  // Can cause NPE
}

// ✅ With Optional
public Optional<User> findUser(String id) {
    return userRepository.findById(id);  // Returns Optional
}

// Usage
Optional<User> user = findUser("123");

if (user.isPresent()) {
    System.out.println(user.get().getName());
}

// Better way
user.ifPresent(u -> System.out.println(u.getName()));
user.ifPresentOrElse(
    u -> System.out.println(u.getName()),
    () -> System.out.println("User not found")
);

// Provide default
String name = user.map(User::getName).orElse("Unknown");

// Throw exception if not present
User u = user.orElseThrow(() -> new UserNotFoundException("User not found"));

// Chain operations
Optional<String> email = user
    .map(User::getEmail)
    .filter(e -> e.contains("@"))
    .orElse("no-email");
```

---

## Real-World Example: Data Analysis

```java
public class DataAnalyzer {
    static class Order {
        String customerId;
        double amount;
        String category;
        LocalDate date;
        
        Order(String cid, double amt, String cat, LocalDate d) {
            customerId = cid; amount = amt; category = cat; date = d;
        }
    }
    
    public static void main(String[] args) {
        List<Order> orders = Arrays.asList(
            new Order("C1", 100, "Electronics", LocalDate.now()),
            new Order("C2", 200, "Clothing", LocalDate.now()),
            new Order("C1", 150, "Electronics", LocalDate.now()),
            new Order("C3", 300, "Furniture", LocalDate.now()),
            new Order("C2", 50, "Clothing", LocalDate.now())
        );
        
        // Total revenue
        double totalRevenue = orders.stream()
            .mapToDouble(o -> o.amount)
            .sum();
        System.out.println("Total: $" + totalRevenue);  // $800
        
        // Revenue by category
        Map<String, Double> revenueByCategory = orders.stream()
            .collect(Collectors.groupingBy(
                o -> o.category,
                Collectors.summingDouble(o -> o.amount)
            ));
        System.out.println(revenueByCategory);
        // {Electronics=250, Clothing=250, Furniture=300}
        
        // Top customer by spending
        Optional<String> topCustomer = orders.stream()
            .collect(Collectors.groupingBy(
                o -> o.customerId,
                Collectors.summingDouble(o -> o.amount)
            ))
            .entrySet().stream()
            .max(Comparator.comparingDouble(Map.Entry::getValue))
            .map(Map.Entry::getKey);
        System.out.println("Top customer: " + topCustomer.orElse("None"));
    }
}
```

---

## Interview Tips:

✅ **Explain lambda syntax and functional interfaces**

✅ **Show method references usage**

✅ **Demonstrate stream operations (filter, map, collect)**

✅ **Know intermediate vs terminal operations**

✅ **Use Optional to avoid null checks**

✅ **Understand lazy evaluation**

---

📖 **Next Topic:** [Spring Boot](./07-spring-boot.md)
