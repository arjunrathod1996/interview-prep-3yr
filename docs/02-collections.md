# 2. Collections Framework - Complete Guide

## 📊 Interview Frequency: 100% | Expected Questions: 15

---

## Collections Hierarchy

```
                    Iterable (interface)
                        ▲
                        │
                    Collection (interface)
              ┌─────────┼──────────┬────────────────┐
              │         │          │                │
            List      Set        Queue         Stream API
    ┌─────────┼────────┐  ┌────────┼──────┐
    │         │        │  │        │      │
 ArrayList LinkedList  │  │      PriorityQueue
  Stack/Vector      │  │
            HashSet TreeSet LinkedHashSet
             |
             Map (separate from Collection)
    ┌────────┼───────────┬──────────────┐
    │        │           │              │
 HashMap TreeMap ConcurrentHashMap LinkedHashMap
```

---

## 1️⃣ List Interface

### Overview:
```
List = Ordered + Allows Duplicates
```

### ArrayList vs LinkedList vs Vector

| Operation | ArrayList | LinkedList | Vector |
|-----------|-----------|-----------|--------|
| get(i) | O(1) ✅ Fast | O(n) ❌ Slow | O(1) ✅ Fast |
| add(end) | O(1) | O(1) | O(1) |
| add(start) | O(n) | O(1) ✅ Fast | O(n) |
| remove(start) | O(n) | O(1) ✅ Fast | O(n) |
| Memory | Contiguous | Non-contiguous | Contiguous |
| Thread-safe | No | No | Yes (synchronized) |
| Performance | Best | Good | Slower |

### Real Code Examples:

```java
// ArrayList - best for random access
List<String> users = new ArrayList<>();
users.add("John");
users.add("Jane");
users.add("Bob");

// Fast random access
String first = users.get(0);  // O(1)
String last = users.get(users.size() - 1);  // O(1)

// Slow insertion at beginning
users.add(0, "Alice");  // O(n) - shifts all elements

// LinkedList - best for frequent insertions
List<String> queue = new LinkedList<>();
queue.add("First");
queue.add("Second");

// Fast insertion at beginning
queue.add(0, "New First");  // O(1)
queue.remove(0);  // O(1)

// Slow random access
// String third = queue.get(100);  // O(n) - traverses links
```

### Internal Structure:

**ArrayList (Array-based):**
```
Array: ["John", "Jane", "Bob", null, null]
Index:   0       1       2     3     4

Direct index access → O(1)
```

**LinkedList (Node-based):**
```
Node → Node → Node
┌──┐   ┌──┐   ┌──┐
│ ├──→ │ ├──→ │ ├──→ null
null ← │ │   ← │ │   ← │ │
│data  │data  │data
└──┘   └──┘   └──┘

Traverse from head to reach index → O(n)
```

---

## 2️⃣ Set Interface

### Overview:
```
Set = Unique + No Order (except TreeSet/LinkedHashSet)
```

| Type | Order | Performance | Use Case |
|------|-------|-------------|----------|
| HashSet | None (random) | O(1) avg | Fast lookups, uniqueness |
| TreeSet | Sorted (natural) | O(log n) | Range queries, sorted results |
| LinkedHashSet | Insertion order | O(1) avg | Maintain insertion order |

### Real Code Examples:

```java
// HashSet - fastest, no order
Set<String> hashSet = new HashSet<>();
hashSet.add("Apple");
hashSet.add("Banana");
hashSet.add("Apple");  // Duplicate, ignored
System.out.println(hashSet);  // [Apple, Banana] or [Banana, Apple]

// Iteration order is unpredictable
for (String fruit : hashSet) {
    System.out.println(fruit);  // Random order
}

// TreeSet - sorted, slower
Set<String> treeSet = new TreeSet<>();
treeSet.add("Zebra");
treeSet.add("Apple");
treeSet.add("Mango");
System.out.println(treeSet);  // [Apple, Mango, Zebra] - sorted!

// Range operations
NavigableSet<String> range = treeSet.subSet("B", "Z");
System.out.println(range);  // [Mango] - all strings from B to Z (exclusive)

// LinkedHashSet - insertion order
Set<String> linkedSet = new LinkedHashSet<>();
linkedSet.add("Third");
linkedSet.add("First");
linkedSet.add("Second");
System.out.println(linkedSet);  // [Third, First, Second] - insertion order
```

### HashSet Performance:

```java
// HashSet uses HashMap internally
// Time complexity depends on hash function

Set<Integer> numbers = new HashSet<>();
for (int i = 0; i < 1000000; i++) {
    numbers.add(i);  // O(1) average case
}
boolean exists = numbers.contains(500000);  // O(1) lookup
```

---

## 3️⃣ Queue Interface

### Overview:
```
Queue = FIFO (First In First Out)
```

| Method | Throws Exception | Returns null |
|--------|------------------|---------------|
| Insert | add(E) | offer(E) |
| Remove | remove() | poll() |
| Examine | element() | peek() |

### Real Code Examples:

```java
// Simple Queue
Queue<Integer> queue = new LinkedList<>();
queue.add(1);
queue.add(2);
queue.add(3);

System.out.println(queue.poll());  // 1 - removes first
System.out.println(queue.poll());  // 2
System.out.println(queue.peek());  // 3 - doesn't remove
System.out.println(queue.isEmpty());  // false
System.out.println(queue.poll());  // 3
System.out.println(queue.isEmpty());  // true

// Priority Queue - elements in priority order
Queue<Integer> pq = new PriorityQueue<>();  // Min heap
pq.add(5);
pq.add(1);
pq.add(3);
System.out.println(pq.poll());  // 1 - smallest first
System.out.println(pq.poll());  // 3
System.out.println(pq.poll());  // 5

// Priority Queue with custom comparator
Queue<Person> peopleByAge = new PriorityQueue<>(
    (p1, p2) -> Integer.compare(p1.age, p2.age)
);
peopleByAge.add(new Person("John", 30));
peopleByAge.add(new Person("Jane", 25));
System.out.println(peopleByAge.poll());  // Jane (age 25)
```

---

## 4️⃣ Map Interface

### Overview:
```
Map = Key-Value Pairs (No Collection, separate hierarchy)
```

| Type | Order | Thread-safe | Performance | Use Case |
|------|-------|-------------|-------------|----------|
| HashMap | None | No | O(1) | General purpose |
| TreeMap | Sorted by key | No | O(log n) | Range queries |
| LinkedHashMap | Insertion | No | O(1) | Maintain order |
| ConcurrentHashMap | None | Yes | O(1) | Multi-threaded |

### Real Code Examples:

```java
// HashMap - no guaranteed order
Map<String, Integer> map = new HashMap<>();
map.put("apple", 1);
map.put("banana", 2);
map.put("cherry", 3);
System.out.println(map);  // Random order

// Iteration
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    Integer value = entry.getValue();
}

// TreeMap - sorted by key
Map<String, Integer> sortedMap = new TreeMap<>();
sortedMap.put("Zebra", 1);
sortedMap.put("Apple", 2);
sortedMap.put("Mango", 3);
System.out.println(sortedMap);  // {Apple=2, Mango=3, Zebra=1} - sorted!

// Range queries
Map<String, Integer> range = sortedMap.subMap("B", "Z");
System.out.println(range);  // {Mango=3} - all keys from B (inclusive) to Z (exclusive)

// LinkedHashMap - insertion order
Map<String, Integer> linkedMap = new LinkedHashMap<>();
linkedMap.put("Third", 1);
linkedMap.put("First", 2);
linkedMap.put("Second", 3);
System.out.println(linkedMap);  // {Third=1, First=2, Second=3} - insertion order

// LRU Cache with LinkedHashMap
LinkedHashMap<String, String> lruCache = new LinkedHashMap<String, String>(16, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > 5;  // Keep only 5 entries
    }
};
```

### ConcurrentHashMap (Thread-safe):

```java
// HashMap in multi-threaded environment - NOT SAFE
Map<String, Integer> unsafeMap = new HashMap<>();
for (int i = 0; i < 1000; i++) {
    new Thread(() -> {
        unsafeMap.put("key" + i, i);  // ❌ Race condition
    }).start();
}

// ✅ Use ConcurrentHashMap
Map<String, Integer> safeMap = new ConcurrentHashMap<>();
for (int i = 0; i < 1000; i++) {
    new Thread(() -> {
        safeMap.put("key" + i, i);  // ✅ Thread-safe
    }).start();
}

// ConcurrentHashMap uses segment locks (doesn't lock entire map)
// Multiple threads can write to different segments simultaneously
```

---

## 5️⃣ Comparators and Sorting

### Comparable vs Comparator:

```java
// Comparable - Natural ordering
public class Person implements Comparable<Person> {
    String name;
    int age;
    
    @Override
    public int compareTo(Person other) {
        return this.age - other.age;  // Sort by age naturally
    }
}

// Usage
List<Person> people = Arrays.asList(
    new Person("John", 30),
    new Person("Jane", 25),
    new Person("Bob", 35)
);
Collections.sort(people);  // Uses compareTo()
// Result: Jane (25), John (30), Bob (35)

// Comparator - Flexible ordering
List<Person> peopleSortedByName = people.stream()
    .sorted(Comparator.comparing(p -> p.name))
    .collect(Collectors.toList());
// Result: Bob, Jane, John

List<Person> peopleSortedByAgeDesc = people.stream()
    .sorted(Comparator.comparingInt((Person p) -> p.age).reversed())
    .collect(Collectors.toList());
// Result: Bob (35), John (30), Jane (25)
```

---

## 6️⃣ Common Interview Questions

**Q: What's the difference between ArrayList and LinkedList?**
```
ArrayList: Array-based, fast random access O(1), slow insertion O(n)
LinkedList: Node-based, slow random access O(n), fast insertion O(1)
Use ArrayList by default unless you need frequent insertions at start/middle
```

**Q: Can we have duplicate keys in HashMap?**
```
No. If you add duplicate key, it overwrites the previous value.
The key itself must be unique, but values can be duplicated.
```

**Q: Is TreeMap thread-safe?**
```
No. Use ConcurrentSkipListMap for thread-safe sorted map.
```

**Q: What's the load factor in HashMap?**
```
Default is 0.75. When map is 75% full, it resizes (doubles capacity).
Higher load factor = more collisions but saves memory
Lower load factor = fewer collisions but uses more memory
```

---

## Collections Utility Methods

```java
// Convert to sorted list
List<String> sorted = new ArrayList<>(set);
Collections.sort(sorted);

// Binary search (must be sorted!)
int index = Collections.binarySearch(sorted, "John");

// Reverse
Collections.reverse(list);

// Shuffle
Collections.shuffle(list);

// Unmodifiable collections
List<String> unmodifiable = Collections.unmodifiableList(list);
// unmodifiable.add("new");  // ❌ UnsupportedOperationException

// Frequency
int count = Collections.frequency(list, "John");

// Max/Min
String maxValue = Collections.max(list);
String minValue = Collections.min(list);
```

---

## Interview Tips:

✅ **Always mention:** Time complexity for your chosen collection

✅ **For performance:** Understand when to use ArrayList vs LinkedList

✅ **For thread-safety:** Know about ConcurrentHashMap and CopyOnWriteArrayList

✅ **For sorting:** Show both Comparable and Comparator usage

✅ **For memory:** Understand initial capacity and load factor impact

---

📖 **Next Topic:** [OOPs Principles](./03-oops.md)
