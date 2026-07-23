# 5. Multithreading - Complete Guide

## 📊 Interview Frequency: 90% | Expected Questions: 15

---

## Thread Lifecycle

```
        ┌──────────────┐
        │   NEW        │
        │ (Created)    │
        └──────┬───────┘
               │
         thread.start()
               │
               ▼
        ┌──────────────┐
        │  RUNNABLE    │◄──────────┐
        │ (Ready/      │           │
        │  Running)    │      notify()
        └──────┬───────┘      wait()    ┌─────────────────┐
               │                    ├──►│ WAITING         │
               │              ┌──────┘   │ (Indefinite)    │
               │              │          │ wait()          │
               │              │          │ join()          │
               │              │          │ park()          │
               │              │          └────────┬────────┘
               │              │                   │
               │              │  ┌────────────────┼──────────────┐
               │              │  │                │              │
               │    Time out  │  ▼                │              ▼
               │    ────►  ┌──────────────┐    ┌──────────────┐
               │          │  TIMED_WAIT  │    │  BLOCKED     │
               │          │ (Temporary)  │    │ (I/O, Locks) │
               │          │ sleep()      │    └──────────────┘
               │          │ wait(time)   │           ▲
               │          └──────┬───────┘           │
               │                 │       (lock acquired)
               │                 └────────────────────┘
               │
         run() completes
         or exception
               │
               ▼
        ┌──────────────┐
        │  TERMINATED  │
        │ (Dead)       │
        └──────────────┘
```

---

## 1️⃣ Creating Threads

### Method 1: Extend Thread Class

```java
public class MyThread extends Thread {
    private String name;
    
    public MyThread(String name) {
        this.name = name;
    }
    
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(name + " - Count: " + i);
            try {
                Thread.sleep(1000);  // Sleep for 1 second
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

// Usage
MyThread t1 = new MyThread("Thread-1");
MyThread t2 = new MyThread("Thread-2");

t1.start();  // ✅ Use start()
t2.start();

// t1.run();  // ❌ Wrong! Calls run() in current thread
```

### Method 2: Implement Runnable (Better)

```java
public class MyRunnable implements Runnable {
    private String name;
    
    public MyRunnable(String name) {
        this.name = name;
    }
    
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(name + " - Count: " + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

// Usage
Thread t1 = new Thread(new MyRunnable("Thread-1"));
Thread t2 = new Thread(new MyRunnable("Thread-2"));

t1.start();
t2.start();

// Lambda (Java 8+)
Thread t3 = new Thread(() -> {
    System.out.println("Running in lambda thread");
});
t3.start();
```

### Why Runnable over Thread?
- ✅ Java doesn't support multiple inheritance
- ✅ Can implement other interfaces too
- ✅ Class can extend another class and implement Runnable

---

## 2️⃣ Thread Synchronization

### Race Condition Example:

```java
// ❌ Race Condition - Data Corruption
public class BankAccount {
    private int balance = 1000;
    
    public void withdraw(int amount) {
        if (balance >= amount) {
            // Critical section - problem here!
            int temp = balance;
            temp -= amount;  // Simulate processing
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            balance = temp;
            System.out.println(Thread.currentThread().getName() + " withdrew " + amount);
        }
    }
}

// Problem: Two threads can both check balance, then both withdraw
BankAccount account = new BankAccount();

// Thread 1 and 2 both see balance = 1000
// Both withdraw 600
// Final balance = 400 (should be -200 or throw error)
```

### Solution 1: Synchronized Method

```java
public class BankAccountSafe {
    private int balance = 1000;
    
    // Only one thread at a time can execute this method
    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            int temp = balance;
            temp -= amount;
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            balance = temp;
            System.out.println(Thread.currentThread().getName() + " withdrew " + amount);
        } else {
            System.out.println("Insufficient funds");
        }
    }
    
    public synchronized int getBalance() {
        return balance;
    }
}
```

### Solution 2: Synchronized Block (More Flexible)

```java
public class BankAccountOptimized {
    private int balance = 1000;
    private final Object lock = new Object();
    
    public void withdraw(int amount) {
        // Only lock the critical section
        synchronized (lock) {
            if (balance >= amount) {
                balance -= amount;
                System.out.println("Withdrawn: " + amount);
            }
        }
        // Other non-critical code can run without lock
    }
}
```

### Solution 3: ReentrantLock (Java 5+)

```java
public class BankAccountWithLock {
    private int balance = 1000;
    private final Lock lock = new ReentrantLock();
    private final Condition sufficientFunds = lock.newCondition();
    
    public void withdraw(int amount) {
        lock.lock();  // Acquire lock
        try {
            while (balance < amount) {
                sufficientFunds.await();  // Wait until balance sufficient
            }
            balance -= amount;
            sufficientFunds.signalAll();  // Notify waiting threads
        } finally {
            lock.unlock();  // Always unlock
        }
    }
    
    public void deposit(int amount) {
        lock.lock();
        try {
            balance += amount;
            sufficientFunds.signalAll();
        } finally {
            lock.unlock();
        }
    }
}
```

---

## 3️⃣ Producer-Consumer Pattern

```java
public class ProducerConsumer {
    private Queue<Integer> queue = new LinkedList<>();
    private final int MAX_SIZE = 5;
    
    // Producer thread
    public void produce() throws InterruptedException {
        for (int i = 1; i <= 20; i++) {
            synchronized (queue) {
                // Wait if queue is full
                while (queue.size() == MAX_SIZE) {
                    queue.wait();  // Release lock and wait
                }
                queue.add(i);
                System.out.println("Produced: " + i + ", Queue size: " + queue.size());
                queue.notifyAll();  // Wake up consumer
            }
            Thread.sleep(100);
        }
    }
    
    // Consumer thread
    public void consume() throws InterruptedException {
        for (int i = 1; i <= 20; i++) {
            synchronized (queue) {
                // Wait if queue is empty
                while (queue.isEmpty()) {
                    queue.wait();
                }
                int item = queue.poll();
                System.out.println("Consumed: " + item + ", Queue size: " + queue.size());
                queue.notifyAll();  // Wake up producer
            }
            Thread.sleep(150);
        }
    }
}

// Usage
ProducerConsumer pc = new ProducerConsumer();

Thread producer = new Thread(() -> {
    try { pc.produce(); }
    catch (InterruptedException e) { e.printStackTrace(); }
});

Thread consumer = new Thread(() -> {
    try { pc.consume(); }
    catch (InterruptedException e) { e.printStackTrace(); }
});

producer.start();
consumer.start();
producer.join();
consumer.join();
```

---

## 4️⃣ Executor Framework

### Thread Pools:

```java
// ExecutorService - manage thread pool
ExecutorService executor = Executors.newFixedThreadPool(3);  // 3 worker threads

// Submit tasks
for (int i = 1; i <= 10; i++) {
    int taskId = i;
    executor.submit(() -> {
        System.out.println("Task " + taskId + " running on " + 
            Thread.currentThread().getName());
        try { Thread.sleep(1000); }
        catch (InterruptedException e) {}
    });
}

// Shutdown
executor.shutdown();  // No new tasks, wait for current
// executor.shutdownNow();  // Stop immediately

// Wait for completion
if (executor.awaitTermination(10, TimeUnit.SECONDS)) {
    System.out.println("All tasks completed");
} else {
    System.out.println("Timeout waiting for tasks");
}
```

### Types of Thread Pools:

```java
// Fixed size pool
ExecutorService fixed = Executors.newFixedThreadPool(5);

// Growing pool (creates new threads as needed)
ExecutorService cached = Executors.newCachedThreadPool();

// Single worker thread
ExecutorService single = Executors.newSingleThreadExecutor();

// Scheduled tasks
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(3);
scheduled.schedule(() -> System.out.println("Task after 5 seconds"), 
    5, TimeUnit.SECONDS);
scheduled.scheduleAtFixedRate(() -> System.out.println("Repeated task"), 
    0, 1, TimeUnit.SECONDS);
```

---

## 5️⃣ Concurrent Collections

### Thread-Safe Collections:

```java
// ConcurrentHashMap (thread-safe HashMap)
Map<String, Integer> map = new ConcurrentHashMap<>();

// CopyOnWriteArrayList (thread-safe ArrayList)
List<String> list = new CopyOnWriteArrayList<>();

// BlockingQueue (producer-consumer)
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);
try {
    queue.put(1);  // Wait if full
    int item = queue.take();  // Wait if empty
} catch (InterruptedException e) { }

// Semaphore (limit concurrent access)
Semaphore semaphore = new Semaphore(3);  // Max 3 concurrent
try {
    semaphore.acquire();  // Acquire permit
    // Do work
    semaphore.release();  // Release permit
} catch (InterruptedException e) { }

// CountDownLatch (synchronize threads)
CountDownLatch latch = new CountDownLatch(3);
// Each worker does: latch.countDown()
// Main thread does: latch.await()  // Waits for count to reach 0

// CyclicBarrier (multiple threads meet at barrier)
CyclicBarrier barrier = new CyclicBarrier(3);
// Each thread does: barrier.await()  // Waits for all 3
```

---

## Real-World Example: Download Manager

```java
public class DownloadManager {
    private final ExecutorService executor = Executors.newFixedThreadPool(3);
    private final List<String> downloads = new CopyOnWriteArrayList<>();
    
    public void downloadFile(String url) {
        executor.submit(() -> {
            System.out.println("Starting download: " + url);
            try {
                // Simulate download
                Thread.sleep(2000);
                synchronized (downloads) {
                    downloads.add(url);
                    System.out.println("Completed: " + url + 
                        ". Total: " + downloads.size());
                }
            } catch (InterruptedException e) {
                System.err.println("Download interrupted: " + url);
            }
        });
    }
    
    public void waitForCompletion() {
        executor.shutdown();
        try {
            if (!executor.awaitTermination(1, TimeUnit.MINUTES)) {
                executor.shutdownNow();
                System.out.println("Timeout waiting for downloads");
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
        System.out.println("All downloads completed: " + downloads);
    }
}

// Usage
DownloadManager dm = new DownloadManager();
dm.downloadFile("http://example.com/file1.zip");
dm.downloadFile("http://example.com/file2.zip");
dm.downloadFile("http://example.com/file3.zip");
dm.waitForCompletion();
```

---

## Interview Tips:

✅ **Explain thread lifecycle**

✅ **Demonstrate synchronization (synchronized, Lock)**

✅ **Show producer-consumer pattern**

✅ **Mention ExecutorService for thread pool**

✅ **Discuss race conditions and deadlock prevention**

---

📖 **Next Topic:** [Java 8 & Streams](./06-java8-streams.md)
