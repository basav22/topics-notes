# Java Multithreading & Concurrency

---

## 1. Creating Threads — 3 Ways

### a) Extending Thread

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread: " + Thread.currentThread().getName());
    }
}

MyThread t = new MyThread();
t.start();   // start() creates new thread, run() just calls method in same thread
```

### b) Implementing Runnable (Preferred)

```java
class MyTask implements Runnable {
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}

Thread t = new Thread(new MyTask());
t.start();

// Lambda shorthand
Thread t2 = new Thread(() -> System.out.println("Lambda thread"));
t2.start();
```

### c) Implementing Callable (returns result)

```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(task);
Integer result = future.get();  // blocks until result is ready
executor.shutdown();
```

**Runnable vs Callable:**

| Feature | Runnable | Callable |
|---------|----------|----------|
| Return value | void | V (generic) |
| Exception | Can't throw checked | Can throw checked |
| Method | `run()` | `call()` |

---

## 2. Thread Lifecycle

```
NEW  →  RUNNABLE  →  RUNNING  →  TERMINATED
              ↕           ↕
          BLOCKED    WAITING / TIMED_WAITING
```

```java
Thread t = new Thread(() -> {});
t.getState();   // NEW
t.start();      // RUNNABLE
// inside run() // RUNNING
// after run()  // TERMINATED
```

Key methods:

```java
t.start();               // start the thread
t.join();                // current thread waits for t to finish
t.join(1000);            // wait max 1 second
Thread.sleep(500);       // pause current thread for 500ms
Thread.yield();          // hint to scheduler to give other threads a turn
t.interrupt();           // interrupt a waiting/sleeping thread
t.isAlive();             // true if started and not yet terminated
```

---

## 3. Synchronization

### The Problem — Race Condition

```java
class Counter {
    int count = 0;
    void increment() { count++; }  // NOT atomic! (read → modify → write)
}

// Two threads calling increment() 1000 times each
// Expected: 2000, Actual: could be anything < 2000
```

### a) synchronized keyword

```java
class Counter {
    int count = 0;

    // synchronized method — locks on `this`
    synchronized void increment() {
        count++;
    }

    // synchronized block — finer control
    void decrement() {
        synchronized (this) {
            count--;
        }
    }

    // static synchronized — locks on Class object
    static synchronized void staticMethod() { /* ... */ }
}
```

### b) volatile keyword

```java
class Flag {
    volatile boolean running = true;  // ensures visibility across threads
}

// Thread 1
flag.running = false;

// Thread 2 — guaranteed to see the updated value
while (flag.running) { /* ... */ }
```

**synchronized vs volatile:**

| Feature | synchronized | volatile |
|---------|-------------|----------|
| Atomicity | Yes | No (only visibility) |
| Mutual exclusion | Yes | No |
| Use case | Compound operations | Simple flags/booleans |

---

## 4. Locks (java.util.concurrent.locks)

### ReentrantLock — More flexible than synchronized

```java
Lock lock = new ReentrantLock();

void doWork() {
    lock.lock();
    try {
        // critical section
    } finally {
        lock.unlock();  // ALWAYS unlock in finally
    }
}

// tryLock — non-blocking
if (lock.tryLock()) {
    try { /* critical section */ }
    finally { lock.unlock(); }
} else {
    System.out.println("Could not acquire lock");
}

// tryLock with timeout
if (lock.tryLock(2, TimeUnit.SECONDS)) { /* ... */ }
```

### ReadWriteLock — Multiple readers, single writer

```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// Multiple threads can read simultaneously
void read() {
    readLock.lock();
    try { /* read data */ }
    finally { readLock.unlock(); }
}

// Only one thread can write, blocks all readers
void write() {
    writeLock.lock();
    try { /* write data */ }
    finally { writeLock.unlock(); }
}
```

---

## 5. ExecutorService & Thread Pools

```java
// Fixed thread pool — exactly n threads
ExecutorService pool = Executors.newFixedThreadPool(4);

// Cached pool — creates threads as needed, reuses idle ones
ExecutorService cached = Executors.newCachedThreadPool();

// Single thread — sequential execution
ExecutorService single = Executors.newSingleThreadExecutor();

// Scheduled — run tasks with delay or periodically
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);

// Submit tasks
pool.execute(() -> System.out.println("Fire and forget"));  // Runnable
Future<String> future = pool.submit(() -> "result");         // Callable

// Submit multiple tasks
List<Callable<String>> tasks = List.of(
    () -> "Task 1",
    () -> "Task 2",
    () -> "Task 3"
);
List<Future<String>> futures = pool.invokeAll(tasks);   // wait for all
String first = pool.invokeAny(tasks);                   // first completed

// Shutdown
pool.shutdown();                    // no new tasks, finish existing
pool.shutdownNow();                 // attempt to cancel running tasks
pool.awaitTermination(5, TimeUnit.SECONDS);  // wait for completion
```

---

## 6. CompletableFuture (Java 8+)

```java
// Basic async task
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
    // runs in ForkJoinPool.commonPool()
    return "Hello";
});

// Chaining
CompletableFuture<String> result = CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")        // transform result
    .thenApply(String::toUpperCase);     // "HELLO WORLD"

result.get();   // blocks and returns "HELLO WORLD"

// Side effect (no return)
CompletableFuture.supplyAsync(() -> "data")
    .thenAccept(System.out::println);    // consume result

// Combine two futures
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "World");

f1.thenCombine(f2, (a, b) -> a + " " + b)  // "Hello World"
  .thenAccept(System.out::println);

// Wait for all
CompletableFuture.allOf(f1, f2).join();

// Wait for any (first completed)
CompletableFuture.anyOf(f1, f2).join();

// Error handling
CompletableFuture.supplyAsync(() -> {
    if (true) throw new RuntimeException("Error!");
    return "ok";
})
.exceptionally(ex -> "Fallback: " + ex.getMessage())
.thenAccept(System.out::println);  // "Fallback: Error!"

// handle — process both result and exception
.handle((result2, ex) -> {
    if (ex != null) return "Error";
    return result2;
});
```

---

## 7. Synchronizers

### CountDownLatch — Wait for N tasks to complete

```java
CountDownLatch latch = new CountDownLatch(3);

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        System.out.println("Task done by " + Thread.currentThread().getName());
        latch.countDown();  // decrement count
    }).start();
}

latch.await();  // main thread waits until count reaches 0
System.out.println("All tasks completed!");
```

### CyclicBarrier — Threads wait for each other

```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("All threads reached barrier!");
});

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        System.out.println("Working...");
        barrier.await();  // wait until all 3 arrive
        System.out.println("Continuing...");
    }).start();
}
```

### Semaphore — Limit concurrent access

```java
Semaphore semaphore = new Semaphore(3);  // max 3 concurrent

for (int i = 0; i < 10; i++) {
    new Thread(() -> {
        try {
            semaphore.acquire();  // blocks if 3 already acquired
            System.out.println("Accessing resource: " + Thread.currentThread().getName());
            Thread.sleep(1000);
        } finally {
            semaphore.release();
        }
    }).start();
}
```

**CountDownLatch vs CyclicBarrier:**

| Feature | CountDownLatch | CyclicBarrier |
|---------|---------------|---------------|
| Reusable | No (one-time) | Yes (resets after each cycle) |
| Who waits | One thread waits for others | All threads wait for each other |
| Use case | Main waits for workers | Parallel phases |

---

## 8. Concurrent Collections

```java
// Thread-safe Map — segments, no full lock
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("a", 1);
map.compute("a", (k, v) -> v + 1);         // atomic
map.putIfAbsent("b", 2);                   // atomic

// Thread-safe List — copies array on write
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
// Good for: many reads, few writes (e.g., listener lists)

// Blocking Queues — producer-consumer pattern
BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);
queue.put("item");     // blocks if full
queue.take();          // blocks if empty

// Other blocking queues
ArrayBlockingQueue<String> abq = new ArrayBlockingQueue<>(10);  // bounded
PriorityBlockingQueue<String> pbq = new PriorityBlockingQueue<>();  // sorted
```

---

## 9. Deadlock, Livelock, Starvation

### Deadlock — Two threads waiting for each other forever

```java
Object lock1 = new Object();
Object lock2 = new Object();

// Thread 1
new Thread(() -> {
    synchronized (lock1) {
        Thread.sleep(100);
        synchronized (lock2) { /* never reached */ }
    }
}).start();

// Thread 2
new Thread(() -> {
    synchronized (lock2) {
        Thread.sleep(100);
        synchronized (lock1) { /* never reached */ }
    }
}).start();
```

**Prevention:**

- Always acquire locks in the **same order**
- Use `tryLock()` with timeout
- Avoid nested locks when possible

### Livelock

- Threads keep responding to each other but make **no progress**
- Like two people in a corridor stepping aside for each other endlessly

### Starvation

- A thread **never gets CPU time** because high-priority threads dominate
- Fix: use fair locks `new ReentrantLock(true)`

---

## 10. Atomic Classes (java.util.concurrent.atomic)

```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();          // atomic ++count
count.decrementAndGet();          // atomic --count
count.addAndGet(5);               // atomic count += 5
count.compareAndSet(5, 10);       // CAS: if count == 5, set to 10
count.get();                      // read

AtomicLong longVal = new AtomicLong();
AtomicBoolean flag = new AtomicBoolean(false);
AtomicReference<String> ref = new AtomicReference<>("hello");
```

---

## Interview Quick Reference

| Question | Answer |
|----------|--------|
| `start()` vs `run()` | `start()` creates new thread; `run()` executes in current thread |
| synchronized vs Lock | Lock has tryLock, timeout, fairness; synchronized is simpler |
| `wait()` vs `sleep()` | `wait()` releases lock, needs `notify()`; `sleep()` holds lock |
| volatile vs AtomicInteger | volatile ensures visibility; AtomicInteger ensures atomicity |
| Thread pool benefit | Reuses threads, avoids creation overhead, controls concurrency |
| `shutdown()` vs `shutdownNow()` | shutdown finishes queued tasks; shutdownNow cancels all |
| `Future` vs `CompletableFuture` | CompletableFuture supports chaining, combining, error handling |
