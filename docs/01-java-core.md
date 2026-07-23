# 1. Java Core Concepts - Complete Guide

## 📊 Interview Frequency: 100% | Expected Questions: 15-20

---

## 1️⃣ JDK vs JRE vs JVM

### Answer:

**Visual Hierarchy:**
```
┌────────────────────────────────────────────────┐
│            JDK (Development Kit)               │
│  ┌───────────────────────────────────────────┐ │
│  │  JRE (Runtime Environment)                │ │
│  │  ┌─────────────────────────────────────┐  │ │
│  │  │ JVM (Virtual Machine)               │  │ │
│  │  │ • Executes bytecode                 │  │ │
│  │  │ • Platform Independent              │  │ │
│  │  │ • Memory Management (GC)            │  │ │
│  │  └─────────────────────────────────────┘  │ │
│  │ • Class Libraries (java.lang, java.util) │ │
│  │ • Garbage Collector                      │ │
│  │ • Other Runtime Tools                    │ │
│  └───────────────────────────────────────────┘ │
│ • Java Compiler (javac)                        │
│ • Debugger (jdb)                               │
│ • Development Tools                            │
│ • Documentation Generator (javadoc)            │
└────────────────────────────────────────────────┘
```

| Component | Purpose | Contains | Size |
|-----------|---------|----------|------|
| **JVM** | Executes Java bytecode | Interpreter, Garbage Collector | ~50 MB |
| **JRE** | Runtime environment for Java programs | JVM + Class Libraries + Tools | ~150 MB |
| **JDK** | Complete development kit | JRE + Compiler + Debugger + Tools | ~300 MB |

### Real-World Example:
```java
// Step 1: Write Java code
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}

// Step 2: Compile (uses JDK - compiler)
// $ javac HelloWorld.java
// Creates: HelloWorld.class (bytecode)

// Step 3: Run (uses JRE/JVM - interpreter)
// $ java HelloWorld
// Output: Hello, World!
```

### Key Differences:
```
JDK: For developers writing Java code
JRE: For users running Java applications
JVM: Core engine that executes bytecode

✅ You need JDK to compile Java code
✅ You need JRE to run Java code
✅ JVM is inside JRE (automatically included)
```

### Interview Follow-up Questions:
- Can JRE run without JDK? **Yes** - but you can't compile
- Why is Java platform-independent? **Because JVM exists on all platforms**
- Can you have multiple JVM versions on one machine? **Yes** - JDK/JRE can be installed separately

---

## 2️⃣ Pass by Value vs Pass by Reference

### Answer:

**Java ALWAYS passes by value, but for objects it passes the reference VALUE (not the reference itself).**

### Concept:
```
Primitives:        int, long, float, double, boolean, char, byte, short
↓
Passed by VALUE (copy of value)

Objects:           String, ArrayList, CustomClass, etc.
↓
Passed by VALUE OF REFERENCE (copy of reference/address, not reference itself)
```

### Real Code Examples:

**Example 1: Primitives (True Pass by Value)**
```java
public class PassByValueExample {
    public static void modifyPrimitive(int num) {
        num = 100;  // Changes only the local copy
        System.out.println("Inside method: " + num);  // 100
    }
    
    public static void main(String[] args) {
        int value = 5;
        System.out.println("Before: " + value);  // 5
        modifyPrimitive(value);
        System.out.println("After: " + value);   // 5 (unchanged!)
    }
}

// Output:
// Before: 5
// Inside method: 100
// After: 5
```

**Why?** Java passes a COPY of the value. Original variable is unaffected.

---

**Example 2: Objects (Pass by Value of Reference)**
```java
public class PassByReferenceExample {
    static class Person {
        String name;
        Person(String name) { this.name = name; }
    }
    
    // Case 1: Modify object's properties
    public static void modifyObjectProperty(Person person) {
        person.name = "Modified";  // Changes the object in heap
    }
    
    // Case 2: Change reference (won't affect original)
    public static void changeReference(Person person) {
        person = new Person("New");  // Local variable only
    }
    
    public static void main(String[] args) {
        Person p = new Person("Original");
        System.out.println("Initial: " + p.name);  // Original
        
        modifyObjectProperty(p);
        System.out.println("After modify: " + p.name);  // Modified ✅ Changed!
        
        changeReference(p);
        System.out.println("After reference change: " + p.name);  // Modified ❌ NOT changed
    }
}

// Output:
// Initial: Original
// After modify: Modified
// After reference change: Modified
```

**Why?** Java passes a COPY of the reference/address. We can modify the object through that reference, but we can't change which object it points to.

### Visual Memory Diagram:
```
STACK                           HEAP
┌──────────────────────┐       ┌────────────────────────┐
│ value (primitive)    │       │                        │
│ = 5                  │       │ (primitives stored     │
│                      │       │  directly in stack)    │
├──────────────────────┤       │                        │
│ p (reference)        │──────→│ Person object          │
│ = 0x123 (address)    │       │ {name = "Modified"}    │
└──────────────────────┘       └────────────────────────┘
```

### Key Takeaway:
```
💡 If you modify an object's PROPERTY through a method parameter:
   ✅ Changes ARE visible outside the method
   
💡 If you REASSIGN the parameter to a new object:
   ❌ Changes are NOT visible outside (different variable)
```

---

## 3️⃣ Immutability and Creating Immutable Objects

### Answer:

**Immutable objects cannot be modified after creation.**

### How to Create Immutable Objects:

```java
public final class ImmutableUser {
    private final String name;        // 1. Make fields final
    private final int age;
    private final List<String> hobbies;  // Watch out for mutable collections!
    
    // 2. Private constructor
    public ImmutableUser(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // 3. Defensive copy for mutable objects
        this.hobbies = Collections.unmodifiableList(new ArrayList<>(hobbies));
    }
    
    // 4. Only provide getters, NO setters
    public String getName() { return name; }
    public int getAge() { return age; }
    public List<String> getHobbies() { 
        return Collections.unmodifiableList(hobbies);
    }
}

// Usage
List<String> initialHobbies = new ArrayList<>();
initialHobbies.add("Reading");

ImmutableUser user = new ImmutableUser("John", 25, initialHobbies);
initialHobbies.add("Gaming");  // Won't affect user's hobbies

System.out.println(user.getHobbies());  // [Reading]
// user.getName() = "John";  // ❌ Compile error - no setter
```

### Rules for Immutability:

✅ Make class `final` - prevents subclassing

✅ Make fields `private` - prevents direct access

✅ Make fields `final` - prevents reassignment

✅ Don't provide setter methods

✅ Return defensive copies of mutable objects

✅ Initialize all fields in constructor

### Real-World Example: Cache Entry

```java
// This object can be safely shared across threads
public final class CacheEntry {
    private final String key;
    private final String value;
    private final long timestamp;
    
    public CacheEntry(String key, String value) {
        this.key = key;
        this.value = value;
        this.timestamp = System.currentTimeMillis();
    }
    
    public String getKey() { return key; }
    public String getValue() { return value; }
    public long getTimestamp() { return timestamp; }
}

// Usage in multithreaded environment
CacheEntry entry = new CacheEntry("user_1", "{id:1, name:John}");

// Thread 1
new Thread(() -> System.out.println(entry.getValue())).start();

// Thread 2
new Thread(() -> System.out.println(entry.getKey())).start();

// No synchronization needed! Entry can't change.
```

### Advantages of Immutability:
```
✓ Thread-safe by design (no locks needed)
✓ Suitable for caching and pooling
✓ Can be used as HashMap/HashSet keys safely
✓ Prevents accidental modifications
✓ Easier to reason about code
✓ Can be shared safely between threads
```

---

## 4️⃣ String Immutability and String Pool

### Why is String Immutable in Java?

```java
// Security reason
String password = "secret123";
passwordValidator.checkPassword(password);
// If String was mutable, someone could modify it after validation!

// Caching reason
String s1 = "hello";
String s2 = "hello";  // Reuses same String object from pool
// Saves memory!

// Thread-safety reason
String s = "hello";
// Can be safely shared across threads without synchronization
```

### String Pool Visualization:

```
╔════════════════════════════════════════════════╗
║        String Pool (Special Area in Heap)      ║
╠════════════════════════════════════════════════╣
║                                                ║
║  "hello"  ──┐                                  ║
║              ├──→ String object: "hello"      ║
║  "hello"  ──┘    Reused!                      ║
║                                                ║
║  "world"  ────→ String object: "world"       ║
║                                                ║
║  "java"   ────→ String object: "java"        ║
║                                                ║
╚════════════════════════════════════════════════╝
```

### String Pool Behavior:

```java
// String literals → Pool
String s1 = "hello";   // Created in pool
String s2 = "hello";   // Reuses s1 reference
System.out.println(s1 == s2);      // true (same reference)
System.out.println(s1.equals(s2)); // true (same content)

// new keyword → Heap (outside pool)
String s3 = new String("hello");   // New object in heap
System.out.println(s1 == s3);      // false (different objects)
System.out.println(s1.equals(s3)); // true (same content)

// intern() → adds to pool
String s4 = new String("hello").intern();
System.out.println(s1 == s4);      // true (interned to pool)
```

### Performance Impact:

```java
// ❌ BAD: Creates 1000 intermediate String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += "item" + i;  // Creates new String each iteration
    // String is immutable, so += creates new object every time!
}
// Creates 1000 String objects! Memory intensive + slow

// ✅ GOOD: Use StringBuilder (mutable)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append("item").append(i);
    // Reuses same buffer, just appends
}
String result = sb.toString();  // Only 1 final String creation
// Much faster and memory efficient!
```

---

## 5️⃣ Autoboxing and Unboxing

### Concept:

```
Autoboxing:   Primitive Type → Wrapper Object (int → Integer)
Unboxing:     Wrapper Object → Primitive Type (Integer → int)
```

### Real Code Examples:

```java
public class AutoboxingExample {
    public static void main(String[] args) {
        // AUTOBOXING: Automatic conversion
        Integer num = 5;           // int 5 → Integer object
        Integer[] arr = {1, 2, 3}; // Each int auto-boxed
        
        // UNBOXING: Automatic conversion
        int value = num;           // Integer object → int
        int sum = arr[0] + arr[1]; // Unbox, then add
        
        System.out.println(value); // 5
        System.out.println(sum);   // 3
    }
}

// Under the hood:
// Integer num = 5;         → Integer num = Integer.valueOf(5);
// int value = num;         → int value = num.intValue();
```

### Wrapper Classes:

| Primitive | Wrapper Class | Memory | Caching |
|-----------|---------------|---------|---------|
| byte | Byte | 1 byte | -128 to 127 |
| short | Short | 2 bytes | -128 to 127 |
| int | Integer | 4 bytes | **-128 to 127** |
| long | Long | 8 bytes | -128 to 127 |
| float | Float | 4 bytes | None |
| double | Double | 8 bytes | None |
| boolean | Boolean | 1 byte | true/false (cached) |
| char | Character | 2 bytes | 0 to 127 |

### ⚠️ Common Gotchas:

**Integer Caching Issue:**
```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b);  // true (cached)

Integer c = 128;
Integer d = 128;
System.out.println(c == d);  // false (NOT cached, new objects)

// Solution: Always use .equals() for comparison
System.out.println(c.equals(d));  // true
```

**NullPointerException Risk:**
```java
Integer num = null;
int value = num;  // ❌ Unboxing null → NullPointerException!

// Safe way:
Integer num2 = null;
if (num2 != null) {
    int value2 = num2;  // ✅ Safe
}

// Or use Optional
Optional<Integer> num3 = Optional.ofNullable(null);
int value3 = num3.orElse(0);  // ✅ Returns 0 if null
```

---

## 6️⃣ Memory Management and Garbage Collection

### Heap Memory Layout:

```
┌───────────────────────��──────────────────────────────┐
│               HEAP MEMORY                            │
├──────────────────────────────────────────────────────┤
│ YOUNG GENERATION (Most GC happens here)              │
│ ┌────────────────────────────────────────────────┐   │
│ │ Eden Space       Survivor S0    Survivor S1    │   │
│ │ (New objects)    (Survived 1x)  (Survived 2x)  │   │
│ │                                                │   │
│ │ 80% of GC        10% each                      │   │
│ └────────────────────────────────────────────────┘   │
├──────────────────────────────────────────────────────┤
│ OLD GENERATION (Long-lived objects)                  │
│ ┌────────────────────────────────────────────────┐   │
│ │ Objects survived 15+ minor GCs move here       │   │
│ │ Major GC (Full GC) happens when this fills     │   │
│ │ Major GC is SLOW - application pauses!        │   │
│ └────────────────────────────────────────────────┘   │
├──────────────────────────────────────────────────────┤
│ METASPACE (Java 8+)                                  │
│ ┌────────────────────────────────────────────────┐   │
│ │ Class definitions, method data, field data     │   │
│ │ (Not collected like old heap)                  │   │
│ └────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘
```

### GC Process Flow:

```
1. Object Creation
   └─→ Allocated in Eden Space

2. Eden Full → Minor GC
   ├─→ Mark referenced objects
   ├─→ Copy survivors to S0
   └─→ Clear Eden

3. After Multiple Minor GCs
   ├─→ Survivors in S0 move to S1
   ├─→ Objects that survived 15+ times
   └─→ Move to Old Generation

4. Old Generation Full → Major GC (Full GC)
   ├─→ Mark all reachable objects
   ├─→ Remove unreferenced objects
   ├─→ Compact memory
   └─→ ⚠️  APPLICATION PAUSES (STW)
```

### Real-World Example:

```java
public class GCExample {
    public static void main(String[] args) {
        // Step 1: Create objects in Eden
        User[] users = new User[1000];
        for (int i = 0; i < 1000; i++) {
            users[i] = new User("User" + i, i);  // Eden Space
        }
        
        // ... process users ...
        
        // Step 2: Remove reference
        users = null;  // All User objects are now unreferenced
        
        // Step 3: Minor GC triggered (automatic)
        // JVM detects no references, marks for collection
        // Memory is reclaimed
        
        // Step 4: Create new objects in freed space
        User newUser = new User("New", 999);  // Uses freed memory
    }
}

// Impact:
// ✅ Automatic memory management (no manual free)
// ❌ GC pause time (especially Full GC)
// ❌ Unpredictable in latency-sensitive apps
```

### Memory Leak Example:

```java
// ❌ Memory Leak: Unreferenced objects keep growing
public class MemoryLeakExample {
    static List<byte[]> cache = new ArrayList<>();
    
    public static void cacheData(byte[] data) {
        cache.add(data);  // Never removed!
        // After many calls, cache grows infinitely
        // Even if data is no longer needed
        // Eventually → OutOfMemoryError
    }
}

// ✅ Fix: Add cache eviction policy
public class CacheWithEviction {
    private static final int MAX_SIZE = 100;
    static List<byte[]> cache = new ArrayList<>();
    
    public static void cacheData(byte[] data) {
        cache.add(data);
        if (cache.size() > MAX_SIZE) {
            cache.remove(0);  // Remove oldest entry
        }
    }
}

// OR use WeakHashMap for automatic cleanup
Map<String, byte[]> weakCache = new WeakHashMap<>();
// Keys are garbage collected if no strong references
```

---

## Interview Tips:

✅ **For JVM questions:** Emphasize platform independence - "Write once, run anywhere"

✅ **For pass by value:** Give both examples - primitives AND objects

✅ **For immutability:** Mention real use case - caching, thread safety

✅ **For String pool:** Show memory savings with code example

✅ **For GC questions:** Mention generational garbage collection and impact on latency

---

## Common Interview Questions & Answers

**Q: What happens if you call `System.gc()`?**
A: It's just a request. JVM may or may not honor it. Don't rely on it in production.

**Q: Can you explicitly delete an object?**
A: No. Java has automatic GC. Setting reference to null is a hint, but no guarantee.

**Q: What's the difference between `==` and `.equals()`?**
A: `==` compares references (addresses), `.equals()` compares values.

---

📖 **Next Topic:** [Collections Framework](./02-collections.md)
