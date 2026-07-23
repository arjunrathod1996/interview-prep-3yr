# 3. OOPs Principles - Complete Guide

## 📊 Interview Frequency: 100% | Expected Questions: 10

---

## Overview: 4 Pillars of OOP

```
┌─────────────────────────────────────────────────────────────┐
│                    OOP Principles                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. ENCAPSULATION                                            │
│     ├─ Bundle data & methods together                       │
│     ├─ Hide internal implementation                         │
│     └─ Use private/public access modifiers                  │
│                                                              │
│  2. INHERITANCE                                              │
│     ├─ Child class inherits from Parent                     │
│     ├─ Reuse code and functionality                         │
│     └─ IS-A relationship                                    │
│                                                              │
│  3. POLYMORPHISM                                             │
│     ├─ Compile-time (Method Overloading)                    │
│     └─ Runtime (Method Overriding)                          │
│                                                              │
│  4. ABSTRACTION                                              │
│     ├─ Hide complexity                                       │
│     ├─ Show only necessary interface                        │
│     └─ Abstract classes & Interfaces                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ ENCAPSULATION

### Definition:
**Bundling data (variables) and methods together, hiding internal details from the outside world.**

### Real-World Example: Bank Account

```java
// ❌ BAD: Direct access to balance
public class BankAccountBad {
    public double balance;  // Public - anyone can modify!
    
    public void withdraw(double amount) {
        balance -= amount;
    }
}

// Usage
BankAccountBad account = new BankAccountBad();
account.balance = -1000;  // ❌ Invalid state! No validation

// ✅ GOOD: Encapsulation with private data
public class BankAccount {
    private double balance;  // Hidden - controlled access
    private String accountNumber;
    
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }
    
    // Controlled deposit with validation
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited: " + amount);
        } else {
            System.out.println("Invalid amount");
        }
    }
    
    // Controlled withdrawal with validation
    public boolean withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            System.out.println("Withdrawn: " + amount);
            return true;
        }
        System.out.println("Insufficient funds or invalid amount");
        return false;
    }
    
    // Read-only access to balance
    public double getBalance() {
        return balance;
    }
    
    // No public setter - balance can only change through methods
}

// Usage
BankAccount account = new BankAccount("ACC001", 1000);
account.deposit(500);     // ✅ Valid operation
account.withdraw(200);    // ✅ Valid operation
account.withdraw(2000);   // ❌ Insufficient funds - blocked
// account.balance = -1000;  // ❌ Compile error - private
```

### Advantages:
- ✅ Control over data modification
- ✅ Validation and business logic enforcement
- ✅ Internal changes don't affect external code
- ✅ Prevents invalid object states

---

## 2️⃣ INHERITANCE

### Definition:
**A class (child) inherits properties and methods from another class (parent). Creates IS-A relationship.**

### Real Code Example:

```java
// Parent class (Super class)
public class Animal {
    private String name;
    protected int age;  // Protected: accessible to child classes
    
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public String getName() {
        return name;
    }
}

// Child class 1
public class Dog extends Animal {
    private String breed;
    
    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }
    
    // Override parent method
    @Override
    public void eat() {
        System.out.println(getName() + " is eating dog food");
    }
    
    // Dog-specific method
    public void bark() {
        System.out.println(getName() + " says: Woof Woof!");
    }
}

// Child class 2
public class Cat extends Animal {
    
    public Cat(String name, int age) {
        super(name, age);
    }
    
    @Override
    public void eat() {
        System.out.println(getName() + " is eating cat food");
    }
    
    public void meow() {
        System.out.println(getName() + " says: Meow!");
    }
}

// Usage
Dog dog = new Dog("Buddy", 3, "Golden Retriever");
dog.eat();    // "Buddy is eating dog food"
dog.bark();   // "Buddy says: Woof Woof!"

Cat cat = new Cat("Whiskers", 2);
cat.eat();    // "Whiskers is eating cat food"
cat.meow();   // "Whiskers says: Meow!"
```

### Types of Inheritance:

```
1. Single Inheritance
   Parent ← Child
   
2. Multilevel Inheritance
   GrandParent ← Parent ← Child
   
3. Hierarchical Inheritance
        Parent
       /      \
    Child1   Child2
   
4. Multiple Inheritance (NOT SUPPORTED IN JAVA)
   ❌ class Child extends Parent1, Parent2 { }  // Compile error
   ✅ Use Interfaces instead
```

### super vs this:

```java
public class Parent {
    int value = 10;
    
    public void display() {
        System.out.println("Parent display");
    }
}

public class Child extends Parent {
    int value = 20;
    
    public void test() {
        System.out.println(this.value);    // 20 (child's value)
        System.out.println(super.value);   // 10 (parent's value)
        
        this.display();   // Calls Child's display if overridden
        super.display();  // Calls Parent's display
    }
}
```

---

## 3️⃣ POLYMORPHISM

### Definition:
**"Many forms" - same method name, different implementations based on context.**

### Type 1: Compile-Time Polymorphism (Method Overloading)

```java
public class Calculator {
    // Same method name, different parameters
    
    public int add(int a, int b) {
        return a + b;
    }
    
    public double add(double a, double b) {
        return a + b;
    }
    
    public String add(String a, String b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
}

// Usage
Calculator calc = new Calculator();
System.out.println(calc.add(5, 10));              // 15 (int)
System.out.println(calc.add(5.5, 10.5));         // 16.0 (double)
System.out.println(calc.add("Hello", "World")); // HelloWorld (String)
System.out.println(calc.add(5, 10, 15));         // 30 (three ints)
```

### Type 2: Runtime Polymorphism (Method Overriding)

```java
// Parent class
public abstract class Shape {
    abstract public void draw();
    abstract public double getArea();
}

// Child classes
public class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing Circle");
    }
    
    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle extends Shape {
    private double length, width;
    
    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing Rectangle");
    }
    
    @Override
    public double getArea() {
        return length * width;
    }
}

// Usage - Polymorphism in action
Shape circle = new Circle(5);      // Reference: Shape, Object: Circle
Shape rectangle = new Rectangle(4, 6);

circle.draw();      // "Drawing Circle" - which draw() is called depends on runtime type
rectangle.draw();   // "Drawing Rectangle"

System.out.println(circle.getArea());      // 78.5
System.out.println(rectangle.getArea());   // 24.0

// Polymorphic collection
List<Shape> shapes = Arrays.asList(circle, rectangle);
for (Shape shape : shapes) {
    shape.draw();  // Correct draw() called based on actual type
}
```

### Method Resolution:

```
Compile Time (Static):      Based on declared type
Runtime (Dynamic):         Based on actual object type

Shape shape = new Circle();
shape.draw();  // Compile time: Shape has draw()
               // Runtime: Circle's draw() is executed
```

---

## 4️⃣ ABSTRACTION

### Definition:
**Hiding complexity and showing only the essential features. Users know WHAT to use, not HOW it works internally.**

### Using Abstract Classes:

```java
// Abstract class - blueprint
public abstract class DatabaseConnection {
    // Abstract methods - must be implemented by child
    abstract public void connect();
    abstract public void executeQuery(String query);
    abstract public void disconnect();
    
    // Concrete method - common implementation
    public void log(String message) {
        System.out.println("[LOG] " + message);
    }
}

// Concrete implementations
public class MySQLConnection extends DatabaseConnection {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL...");
        // MySQL-specific connection logic
    }
    
    @Override
    public void executeQuery(String query) {
        System.out.println("Executing MySQL query: " + query);
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from MySQL...");
    }
}

public class PostgreSQLConnection extends DatabaseConnection {
    @Override
    public void connect() {
        System.out.println("Connecting to PostgreSQL...");
    }
    
    @Override
    public void executeQuery(String query) {
        System.out.println("Executing PostgreSQL query: " + query);
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from PostgreSQL...");
    }
}

// Usage - user doesn't care about implementation
public class DatabaseService {
    private DatabaseConnection connection;
    
    public DatabaseService(DatabaseConnection conn) {
        this.connection = conn;  // Abstraction - works with any DB
    }
    
    public void performQuery() {
        connection.connect();
        connection.executeQuery("SELECT * FROM users");
        connection.log("Query executed");
        connection.disconnect();
    }
}

// Usage
DatabaseService mysqlService = new DatabaseService(new MySQLConnection());
mysqlService.performQuery();

DatabaseService postgresService = new DatabaseService(new PostgreSQLConnection());
postgresService.performQuery();
```

### Using Interfaces:

```java
// Interface - defines contract
public interface PaymentMethod {
    void pay(double amount);
    void refund(double amount);
}

// Implementations
public class CreditCard implements PaymentMethod {
    @Override
    public void pay(double amount) {
        System.out.println("Paying " + amount + " via Credit Card");
    }
    
    @Override
    public void refund(double amount) {
        System.out.println("Refunding " + amount + " to Credit Card");
    }
}

public class PayPal implements PaymentMethod {
    @Override
    public void pay(double amount) {
        System.out.println("Paying " + amount + " via PayPal");
    }
    
    @Override
    public void refund(double amount) {
        System.out.println("Refunding " + amount + " to PayPal");
    }
}

// Service uses abstraction
public class ShoppingCart {
    private PaymentMethod paymentMethod;
    
    public ShoppingCart(PaymentMethod method) {
        this.paymentMethod = method;
    }
    
    public void checkout(double total) {
        paymentMethod.pay(total);
    }
}

// Usage
ShoppingCart cart1 = new ShoppingCart(new CreditCard());
cart1.checkout(99.99);  // "Paying 99.99 via Credit Card"

ShoppingCart cart2 = new ShoppingCart(new PayPal());
cart2.checkout(99.99);  // "Paying 99.99 via PayPal"
```

---

## Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---------|----------------|----------|
| Constructor | Yes | No |
| State (variables) | Yes (any) | Only constants (final static) |
| Methods | Both abstract & concrete | Only abstract (before Java 8) |
| Access Modifier | public, protected, private | Only public |
| Inheritance | Single only | Multiple |
| Use Case | IS-A relationship | CAN-DO capability |
| When to use | Share common code | Define contract |

```java
// Abstract class - represents what something IS
public abstract class Vehicle {
    int speed;  // State
    abstract void drive();  // Abstract
    void log() { }  // Concrete
}

// Interface - represents what something CAN DO
public interface Flyable {
    void fly();  // Only abstract methods (by default)
}

// A class can implement multiple interfaces
public class Airplane extends Vehicle implements Flyable {
    @Override
    void drive() { }
    
    @Override
    public void fly() { }
}
```

---

## Interview Tips:

✅ **For Encapsulation:** Show private variables with public getters/setters and validation

✅ **For Inheritance:** Mention single inheritance in Java, use super keyword

✅ **For Polymorphism:** Give both compile-time (overloading) and runtime (overriding) examples

✅ **For Abstraction:** Show interface/abstract class usage for flexibility

✅ **Real-World Story:** Relate to a project where you used these principles

---

## Common Interview Questions

**Q: Can you instantiate an abstract class?**
A: No. Abstract classes are incomplete blueprints. But you can have abstract class references pointing to concrete subclass objects.

**Q: What's the difference between final and abstract?**
A: abstract = must be overridden, final = cannot be overridden

**Q: Can an interface have state?**
A: Only constants (public static final). From Java 8+, interfaces can have default methods.

---

📖 **Next Topic:** [Exception Handling](./04-exception-handling.md)
