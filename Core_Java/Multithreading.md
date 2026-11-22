Multithreading is the capability of a CPU (or a single process within an OS) to execute multiple threads concurrently. In Java, multithreading is managed primarily by the JVM, which maps Java threads to operating system kernel threads (or lightweight virtual threads in modern Java).

Threads within the same process share:

- Heap memory
- Objects
- Static variables
- Resources

But each thread has its own:

- Stack
- Program counter
- Local variables

#### **Thread Creation:**

1 - Extend `Thread`

```java
class MyThread extends Thread {

    @Override
    public void run() {
    System.out.println("Thread running");
    }
}
public class Main {
public static void main(String[]args) {
MyThread thread=new MyThread();
thread.start();
    }
}
```

2- Implement Runnable

---

Preferred over extending `Thread` when you only need to define a task.

```java
class MyTask implements Runnable {

    @Override
    public void run() {
    System.out.println("Task running");
    }
}
public class Main {
public static void main(String[] args) {
Thread thread=new Thread(new MyTask());
thread.start();
    }
}
```

Using lambda:

```java
Thread thread=new Thread(() -> System.out.println("Task running"));
thread.start();
```

---

3- Callable

`Runnable` cannot return a result.

`Callable` can return a result and throw checked exceptions.

```java
Callable<Integer> task= () -> {
return 10+20;
};
```

Usually used with `ExecutorService`:

```java
ExecutorService executor=Executors.newFixedThreadPool(2);
Future<Integer> future=executor.submit(task);
Integer result=future.get();
executor.shutdown();
```

#### Runnable vs Callable

| Runnable | Callable |
| --- | --- |
| `run()` | `call()` |
| No return value | Returns value |
| Cannot throw checked exception directly | Can throw checked exception |
| Used for tasks without results | Used when result is required |

---

### Thread Lifecycle

A Java thread can have these states:

```
NEW
 |
 | start()
 ↓
RUNNABLE
 |
 +------→ BLOCKED
 |
 +------→ WAITING
 |
 +------→ TIMED_WAITING
 |
 ↓
TERMINATED
```

Java's official `Thread.State` values are:

```
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

**NEW**

Thread created but not started.

```java
Thread t = new Thread();
```

**RUNNABLE**

After:

```
t.start();
```

It is eligible to run.

**BLOCKED**

Waiting to acquire a monitor lock.

**WAITING**

Waiting indefinitely for another thread.

Examples:

```java
Object.wait();
Thread.join();
LockSupport.park();
```

**TIMED_WAITING**

Waiting for a specified amount of time.

Examples:

```
Thread.sleep(1000);
Object.wait(1000);
Thread.join(1000);
```

**TERMINATED**

Thread has completed execution.

---

#### **`start()` vs `run()`**

### `start()`

```
thread.start();
```

Creates a new thread and JVM eventually invokes `run()` on that thread.

### `run()`

```
thread.run();
```

Does **not** create a new thread.

It runs in the current thread.

### `sleep()`

Pauses the current thread for a specified period.

```
Thread.sleep(1000);
```

Important:

> `sleep()` does NOT release a lock held by the thread.
> 

Example:

```
synchronized (lock) {
Thread.sleep(1000);

}
```

The thread sleeps but continues holding `lock`.

---

### `join()`

`join()` makes one thread wait for another thread to complete.

```java
Thread t1 = new Thread(() -> {
System.out.println("Processing...");
});
t1.start();
t1.join();
System.out.println("Main continues");
```

Flow:

```
Main
 |
 +---- start t1
 |       |
 |       +---- processing
 |
 +---- join()
          |
          | waits
          ↓
       t1 finishes
          |
          ↓
       Main continues
```

---

### Daemon Threads

Daemon threads perform background work.

Examples:

- Garbage collection
- Background monitoring
- Cleanup tasks

```java
Thread thread = new Thread(() -> {
while (true) {
System.out.println("Background work");
 }
});
thread.setDaemon(true);
thread.start();
```

JVM can terminate when **no non-daemon threads remain**, even if daemon threads are still running.

Important:

```
thread.setDaemon(true);
```

must be called **before `start()`**.

---

### Race Condition

A **race condition** happens when two or more programs, threads, or actions try to change the same piece of data at the exact same time, and the final outcome depends on who gets there first.

It is called a "race" because the execution order isn't guaranteed—whichever action wins the speed race determines the result, often causing unexpected bugs or errors.

Example:

```java
class Counter {
int count=0;
void increment() {
count++;
  }
}
```

Suppose:

```
Thread A → count++
Thread B → count++
```

`count++` is not atomic.

Conceptually:

```
read count
add 1
write count
```

Possible execution:

```
count = 0

Thread A reads 0
Thread B reads 0

Thread A writes 1
Thread B writes 1

Final = 1
```

Expected:

```
2
```

Actual:

```
1
```

---

### Thread Safety

Code is thread-safe when it behaves correctly when accessed by multiple threads concurrently.

Ways to achieve thread safety:

- Synchronization
- Locks
- Atomic classes
- Immutable objects
- Concurrent collections
- Thread confinement
- Proper visibility mechanisms

---

#### `synchronized`

`synchronized` provides:

1. **Mutual exclusion**
2. **Visibility guarantees**

Example:

```java
classCounter {
private int count;
public synchronized void increment() {
count++;
    }
    
 public synchronized int getCount() {
    return count;
    }
}
```

Only one thread can execute the synchronized method on the same object monitor at a time.

---

#### Synchronized Block

Instead of locking the entire method:

```java
public void increment() {
synchronized (this) {
count++;
    }
}
```

This can reduce the critical section.

Better:

```java
public void process() {
// non-critical work

synchronized (lock) {
// shared data
 }
    // non-critical work
}
```

---

#### Object Monitor

An object monitor is a lock associated with every Java object. It provides mutual exclusion for synchronized code, ensuring that only one thread can execute a synchronized section guarded by that monitor at a time. The monitor also provides the mechanism behind `wait()`, `notify()`, and `notifyAll()` for thread coordination.

For:

```java
synchronized (obj) {
// critical section
}
```

the thread must acquire the monitor associated with `obj`.

Only one thread can own that monitor at a time.

---

#### Static Synchronized Method

Consider:

```java
public synchronized void method() {
}
```

Lock is associated with the **object instance**.

But:

```java
public static synchronized void method() {
}
```

locks the **Class object**.

Conceptually:

```
instance synchronized
        ↓
this

static synchronized
        ↓
ClassName.class
```

---

#### Synchronization Problem

Synchronization can cause performance problems if the critical section is too large.

Bad:

```java
synchronized (this) {
callExternalAPI();
performDatabaseOperation();
calculateSomething();
updateSharedState();
}
```

Better:

```java
callExternalAPI();
performDatabaseOperation();
synchronized (this) {
updateSharedState();
}
```

Keep the critical section as small as safely possible.

---

### `volatile`

`volatile` ensures that reads/writes to a variable have appropriate **visibility and ordering guarantees** across threads.

When a variable is declared `volatile`, Java guarantees that a thread reads the **latest value** written by another thread.

Example:

```java
class Task {
private volatile boolean running = true;
void stop() {
running=false;
    }
void execute() {
while (running) {
// work
        }
    }
}
```

Without `volatile`, another thread may not promptly observe the updated value due to memory visibility issues.

---

#### Volatile Does NOT Make Compound Operations Atomic

This is important.

```java
volatile int count;
count++;
```

Still unsafe.

Because:

```
read
+
write
```

are separate operations.

Use:

```
Atomic Integer
```

or synchronization.

---

### Atomic Classes

Java provides **atomic classes** in `java.util.concurrent.atomic` for performing **thread-safe operations without using traditional `synchronized` blocks**.

They are particularly useful in concurrent applications where multiple threads need to update shared data.

Example:

```
AtomicInteger
AtomicLong
AtomicBoolean
AtomicReference
LongAdder
LongAccumulator
```

Example:

```java
AtomicInteger counter = new AtomicInteger();
counter.incrementAndGet();
System.out.println(counter.get());
```

Multiple threads can safely increment it without using a synchronized block.

---

### Compare And Set

**CAS (Compare-And-Set/Compare-And-Swap)** atomically checks whether a value is equal to an expected value and, if so, replaces it with a new value. It is a fundamental mechanism behind many lock-free concurrent operations.
****

Conceptually:

```
Expected value = 10

Current value == 10?
       |
      YES
       |
Change to 11
```

If another thread changed the value first:

```
Current = 12

Expected = 10

CAS fails
```

The operation can retry.

---

#### AtomicInteger Example

```java
class Counter {
private final AtomicInteger count= new AtomicInteger(0);
public void increment() {
count.incrementAndGet();
    }
public int getCount() {
return count.get();
    }
}
```

---

### `synchronized` vs `volatile` vs Atomic

| Feature | synchronized | volatile | Atomic |
| --- | --- | --- | --- |
| Visibility | Yes | Yes | Yes |
| Mutual exclusion | Yes | No | No |
| Atomic increment | Yes | No | Yes |
| Blocking | Yes | No | Usually no |
| Good for simple flag | Sometimes | Yes | Sometimes |
| Compound operations | Yes | No | Yes, supported operations |

Example:

```java
volatile boolean running;
```

Good for a state flag.

```java
Atomic Integer counter;
```

Good for concurrent counters.

```java
synchronized
```

Good for protecting multiple related operations/invariants.

---

### ExecutorService

Instead of manually creating threads:

```
newThread(...)
```

use a thread pool.

```java
ExecutorService executor=Executors.newFixedThreadPool(5);
```

Submit tasks:

```
executor.submit(() -> {
System.out.println("Processing");
});
```

Shutdown:

```
executor.shutdown();
```

---

### Why Thread Pools?

Creating threads repeatedly can be expensive.

Thread pools provide:

- Thread reuse
- Controlled concurrency
- Better resource management
- Task queuing
- Easier lifecycle management

Architecture:

```
                Tasks
                  |
        +---------+---------+
        |         |         |
        ↓         ↓         ↓
      Queue     Queue     Queue
                  |
                  ↓
            Thread Pool
          +------+------+------+
          |      |      |      |
        T1     T2     T3     T4
```

---

#### Common Executor Types

Fixed Thread Pool

```java
ExecutorService executor = Executors.newFixedThreadPool(5);
```

Five worker threads.

Useful when you want a controlled number of concurrent tasks.

---

Single Thread Executor

```java
ExecutorService executor=Executors.newSingleThreadExecutor();
```

Only one worker thread.

Useful when tasks must execute sequentially.

---

Scheduled Executor

```java
ScheduledExecutor Serviceexecutor = Executors.newScheduledThreadPool(2);
```

Example:

```
executor.schedule(
    () ->System.out.println("Executed"),5,TimeUnit.SECONDS
);
```

---

Cached Thread Pool

```
Executors.newCachedThreadPool();
```

Can dynamically create/reuse threads.

Be careful with unbounded task submission because the number of threads can grow substantially.

---

### Future

`Future` represents the result of an asynchronous computation.

```java
ExecutorService executor=Executors.newFixedThreadPool(2);
Future<Integer> future=executor.submit(() ->10+20);
Integer result=future.get();
```

`get()` blocks until the result is available.

---

#### Future Timeout

Avoid waiting indefinitely.

```java
Integer result= future.get(2,TimeUnit.SECONDS);
```

Possible exceptions:

```
InterruptedException
ExecutionException
TimeoutException
```

---

#### Cancelling a Task

```
future.cancel(true);
```

`true` requests interruption if the task is running.

Important:

> Cancellation/interruption is cooperative; code should respond appropriately to interruption.
> 

---

### CompletableFuture

`CompletableFuture` is useful for asynchronous programming and composing multiple operations.

Example:

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() ->"Hello");
future.thenApply(value ->value+" World").thenAccept(System.out::println);
```

Flow:

```
supplyAsync()
     ↓
"Hello"
     ↓
thenApply()
     ↓
"Hello World"
     ↓
thenAccept()
```

---

#### `thenApply()` vs `thenAccept()`

### thenApply

Transforms a result.

```
future.thenApply(value ->value.toUpperCase());
```

Returns another `CompletableFuture`.

### thenAccept

Consumes the result.

```
future.thenAccept(System.out::println);
```

Usually returns:

```
CompletableFuture<Void>
```

---

#### `thenRun()`

Runs an action after completion but doesn't receive the result.

```
future.thenRun(() ->System.out.println("Completed"));
```

---

### Combining CompletableFutures

#### thenCombine

Combine two independent results.

```java
CompletableFuture<Integer> price = CompletableFuture.supplyAsync(() ->100);
CompletableFuture<Integer> tax = CompletableFuture.supplyAsync(() ->20);
CompletableFuture<Integer> total = price.thenCombine(tax, (p,t) -> p+t );
```

---

#### `allOf()`

Wait for multiple asynchronous tasks.

```java
CompletableFuture<Void> result = CompletableFuture.allOf(future1,future2,future3);
```

Useful when several independent operations must complete before continuing.

---

#### Exception Handling with CompletableFuture

```java
future.thenApply(value ->process(value)).exceptionally(ex -> {
System.out.println(ex.getMessage());
return"DEFAULT";
 });
```

Another option:

```java
future.handle((result,exception) -> {if (exception!=null) {return "ERROR";
    }
    return result;
});
```

---

#### ForkJoinPool

Designed for divide-and-conquer parallelism.

Example concept:

```
Large Task
    |
    +---- Task A
    |       |
    |       +---- A1
    |       +---- A2
    |
    +---- Task B
            |
            +---- B1
            +---- B2
```

Uses **work stealing**.

A worker with no work can steal tasks from another worker's queue.

---

### Parallel Streams

Example:

```java
List<Integer> numbers = List.of(1,2,3,4,5);
numbers.parallelStream().map(x ->x*2).forEach(System.out::println);
```

Parallel streams commonly use the common `ForkJoinPool`.

### Be careful

Parallel streams aren't automatically faster.

They can be problematic when:

- Dataset is small
- Operations are cheap
- Work is I/O-bound
- Ordering is important
- Shared mutable state exists
- Common pool contention occurs

---

### Concurrent Collections

Normal collections aren't generally safe for concurrent modification.

Examples of concurrent collections:

```
ConcurrentHashMap
CopyOnWriteArrayList
BlockingQueue
ConcurrentLinkedQueue
ConcurrentLinkedDeque
```

---

#### ConcurrentHashMap

```java
ConcurrentHashMap<String,Integer> map = new ConcurrentHashMap<>();
map.put("A",100);
```

Designed for concurrent access.

Example:

```java
map.compute("A", (key,value) -> value==null ? 1 : value+1);
```

Useful for atomic map-level operations.

---

---

#### CopyOnWriteArrayList

Useful when:

> Reads are frequent and writes are relatively rare.
> 

Example:

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
```

When modified, it creates a new underlying array.

Advantages:

- Safe concurrent iteration
- Iterators don't require external locking

Disadvantage:

- Expensive for frequent writes
- Additional memory usage

---

#### BlockingQueue

Extremely useful in producer-consumer systems.

```
Producer
   |
   ↓
BlockingQueue
   |
   ↓
Consumer
```

Example:

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>();
queue.put("Order-1");
String order=queue.take();
```

`put()` can wait if the queue is full.

`take()` can wait if the queue is empty.

---

Producer-Consumer Pattern

```
Producer Thread
      |
      ↓
+-------------+
|   Queue     |
+-------------+
      |
      ↓
Consumer Thread
```

Example:

```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
Thread producer=new Thread(() -> {
try
 { 
 queue.put(100);
 }
 catch (InterruptedExceptione) {
 Thread.currentThread().interrupt();
  }
});
Thread consumer=new Thread(() -> {
try {
Integer value=queue.take();
System.out.println(value);
 }
catch (InterruptedExceptione) {
Thread.currentThread().interrupt();
 }
});
```

---

### Lock Interface

Java provides:

```
java.util.concurrent.locks.Lock
```

Example:

```java
Lock lock=new ReentrantLock();
lock.lock();
try {// critical section
}
finally {
lock.unlock();
}
```

Always release the lock in `finally`.

---

#### ReentrantLock

`ReentrantLock` provides functionality similar to `synchronized`, plus additional features.

```java
ReentrantLock lock=new ReentrantLock();
lock.lock();
try {// critical section
}
finally {
lock.unlock();
}
```

Advantages include:

- `tryLock()`
- Interruptible lock acquisition
- Optional fairness
- More flexible lock management

---

#### `tryLock()`

Instead of waiting indefinitely:

```
if (lock.tryLock()) {
try {// work
    }
    finally {lock.unlock();
    }

}else {
System.out.println("Could not acquire lock");
}
```

With timeout:

```
if (lock.tryLock(2,TimeUnit.SECONDS)) {// work
}
```

Useful for reducing lock-related waiting and designing timeout behavior.

---

#### ReadWriteLock

Useful when:

- Reads are frequent
- Writes are relatively rare

```java
ReadWriteLock lock=new ReentrantReadWriteLock();
```

Read:

```java
lock.readLock().lock();
try {// read
}
finally {
lock.readLock().unlock();
}
```

Write:

```java
lock.writeLock().lock();
try {// write
}
finally {
lock.writeLock().unlock();
}
```

Multiple readers can hold the read lock concurrently, subject to the lock's coordination rules.

---

#### Semaphore

Semaphore controls the number of threads that can access a resource simultaneously.

```java
Semaphore semaphore = new Semaphore(3);
```

Only three permits are initially available.

```java
semaphore.acquire();
try {
// access resource
}
finally {
semaphore.release();
}
```

Example use case:

```
100 requests
      |
      ↓
 Semaphore(10)
      |
      ↓
Only 10 access resource concurrently
```

Useful for:

- Connection pools
- Rate limiting/concurrency limiting
- Limited external resources

---

#### CountDownLatch

Allows one or more threads to wait until a count reaches zero.

```java
CountDownLatch latch = new CountDownLatch(3);
```

Worker:

```
latch.countDown();
```

Waiting thread:

```
latch.await();
```

Example:

```
Worker 1 ──┐
Worker 2 ──┼──→ countDown()
Worker 3 ──┘
             |
          count = 0
             |
             ↓
        Main continues
```

A `CountDownLatch` is generally **one-shot**; once the count reaches zero, it cannot be reset.

---

#### CyclicBarrier

Allows a group of threads to wait for each other at a synchronization point.

```java
CyclicBarrier barrier = new CyclicBarrier(3);
```

Each thread:

```
barrier.await();
```

When all three arrive:

```
Thread 1 ──┐
Thread 2 ──┼──→ Barrier
Thread 3 ──┘
             |
             ↓
        All continue
```

Unlike `CountDownLatch`, a `CyclicBarrier` can be reused.

---

### Latch vs Barrier

| CountDownLatch | CyclicBarrier |
| --- | --- |
| One-shot | Reusable |
| Count reaches zero | All parties arrive |
| Threads don't have to wait for each other in the same way | Participating threads wait at barrier |
| `countDown()` | `await()` |

---

### Phaser

`Phaser` is a more flexible synchronization mechanism for multiple phases and dynamically changing numbers of parties.

Useful when tasks proceed through stages:

```
Phase 1
   ↓
Phase 2
   ↓
Phase 3
```

---

### Deadlock

Deadlock occurs when threads wait for locks held by each other indefinitely.

Example:

```
Thread A:
Lock A → waits for Lock B

Thread B:
Lock B → waits for Lock A
```

Neither can proceed.

Example:

```
synchronized (lockA) {
synchronized (lockB) {// work
    }
}
```

Another thread:

```
synchronized (lockB) {
synchronized (lockA) {
// work
    }
}
```

---

#### Avoiding Deadlock

### 1. Consistent lock ordering

Always acquire:

```
Lock A → Lock B
```

Never:

```
Lock B → Lock A
```

### 2. Use `tryLock()`

Timeout can prevent indefinite waiting.

### 3. Minimize lock scope

### 4. Avoid unnecessary nested locks

---

### Livelock

Threads aren't blocked, but they keep reacting to each other without making progress.

Example:

```
Thread A → backs off
Thread B → backs off

Thread A → tries again
Thread B → tries again

Thread A → backs off
Thread B → backs off
```

CPU may be active, but useful work isn't progressing.

---

### Starvation

A thread waits indefinitely because other threads continually get access to the resource.

Possible causes:

- Unfair lock acquisition
- High-priority thread monopolization
- Poor scheduling
- Long-running critical sections

---

### Deadlock vs Starvation vs Livelock

| Problem | Description |
| --- | --- |
| Deadlock | Threads wait forever for each other's locks |
| Starvation | A thread doesn't get enough access to resources |
| Livelock | Threads keep running but make no progress |

---

### Thread Interruption

Interruption is a cooperative mechanism for requesting that a thread stop what it is doing.

```java
thread.interrupt();
```

Check:

```java
if (Thread.currentThread().isInterrupted()) {
// stop
}
```

Or:

```java
if (Thread.interrupted()) {
// handle interruption
}
```

Important distinction:

```
isInterrupted()
```

does not clear the interrupt status.

```
Thread.interrupted()
```

checks and clears the current thread's interrupt status.

---

#### Handling InterruptedException

Bad:

```
try {
Thread.sleep(1000);
}catch (InterruptedExceptione) {// ignore
}
```

Better:

```
try {Thread.sleep(1000);
}catch (InterruptedExceptione) {Thread.currentThread().interrupt();
}
```

Why?

Because catching `InterruptedException` clears the interrupted status, so restoring it allows higher-level code to see that interruption was requested.

---

### Java Memory Model — JMM

The **Java Memory Model** defines how threads interact through memory.

Important concepts:

```
Visibility
Ordering
Atomicity
Happens-before
```

---

#### Visibility Problem

Consider:

```java
boolean running = true;
```

Thread A:

```java
while (running) {
}
```

Thread B:

```java
running = false;
```

Without proper synchronization/visibility guarantees, Thread A is not guaranteed to promptly observe the update.

Using:

```java
volatile boolean running;
```

provides the needed visibility semantics for this simple flag.

---

#### Happens-Before

A **happens-before relationship** establishes ordering and visibility guarantees between actions.

Important examples:

### Unlock → subsequent lock

An unlock on a monitor happens-before a subsequent lock on the same monitor.

### Volatile write → subsequent volatile read

A write to a volatile variable happens-before a subsequent read of that same variable.

### Thread start

Actions before:

```
thread.start();
```

happen-before actions in the started thread.

### Thread termination/join

Actions performed by a thread happen-before another thread successfully returns from:

```
thread.join();
```

This is an important interview topic.

---

### ThreadLocal

`ThreadLocal` provides thread-specific values.

```java
ThreadLocal<String> user = new ThreadLocal<>();
user.set("John");
System.out.println(user.get());
user.remove();
```

Each thread has its own value.

```
Thread A → ThreadLocal → "John"

Thread B → ThreadLocal → "David"

Thread C → ThreadLocal → "Sarah"
```

Common use cases:

- Request context
- Correlation IDs
- Per-thread state

### Important with thread pools

Always consider:

```java
threadLocal.remove();
```

after use, especially in pooled threads, to avoid stale data being retained across tasks.

---

#### Thread Safety Through Immutability

Immutable objects are naturally easier to use concurrently.

Example:

```java
public final class User {
private final String name;
public User(String name) {
this.name=name;
    }
    public String getName() {
    returnname;
    }
}
```

Once created, state cannot change.

Advantages:

- No synchronization needed for normal reads
- Safe sharing
- Easier reasoning

---

#### Thread Confinement

Keep mutable data accessible to only one thread.

```
Thread A
   |
   +-- Mutable Object

Thread B
   |
   +-- Different Mutable Object
```

No shared mutable state means fewer synchronization problems.

---

#### Context Switching

The CPU switches between threads.

```
Thread A
   ↓
Context switch
   ↓
Thread B
   ↓
Context switch
   ↓
Thread C
```

Context switching has overhead.

Too many threads can therefore hurt performance.

---

#### CPU-Bound vs I/O-Bound Tasks

### CPU-bound

Examples:

- Image processing
- Complex calculations
- Encryption
- Data transformation

Thread count is often kept around the number of available CPU cores, depending on workload and implementation.

### I/O-bound

Examples:

- Database calls
- HTTP calls
- File operations
- Network calls

Threads may spend substantial time waiting, so concurrency can be higher—but the correct pool size depends on latency, resource limits, downstream capacity, and workload.

---

### Thread Pool Sizing

A common interview heuristic:

### CPU-bound

```
Threads ≈ number of CPU cores
```

### I/O-bound

```
Threads > number of CPU cores
```

But don't treat these as universal formulas.

For production systems, consider:

- CPU cores
- Task execution time
- I/O wait time
- Queue size
- DB connection pool
- External API limits
- Memory
- Latency requirements
- Throughput

---

### ExecutorService Best Practice

Instead of:

```
Executors.newFixedThreadPool(100);
```

blindly choosing a number, understand:

```
Incoming requests
       |
       ↓
Thread Pool
       |
       ↓
Database Pool
       |
       ↓
Database
```

If you have:

```
100 application threads
10 DB connections
```

then many application threads may simply wait for DB connections.

Concurrency must be designed across the whole system.

---

### `shutdown()` vs `shutdownNow()`

### shutdown()

Stops accepting new tasks but allows submitted tasks to complete.

```
executor.shutdown();
```

### shutdownNow()

Attempts to stop currently executing tasks, typically by interrupting worker threads, and returns tasks that never started.

```
executor.shutdownNow();
```

Tasks must cooperate with interruption.

---

### ExecutorService Interview Example

```java
ExecutorService executor=Executors.newFixedThreadPool(3);
try {
for (int i=0;i<10;i++) {
int taskId=i;
executor.submit(() -> {
System.out.println("Processing "+taskId+" by "+Thread.currentThread().getName());
        });
    }

}
finally {
executor.shutdown();
}
```

---

## Virtual Threads — Java 21+

Virtual threads are lightweight threads designed for high-concurrency workloads, particularly blocking I/O.

Example:

```java
Thread.startVirtualThread(() -> {System.out.println("Virtual thread");
});
```

Or:

```java
try (ExecutorService executor=Executors.newVirtualThreadPerTaskExecutor()) 
{
executor.submit(() -> {
callDatabase();
    });
}
```

Conceptually:

```
Many Virtual Threads
       |
       ↓
JVM
       |
       ↓
Carrier Platform Threads
       |
       ↓
CPU
```

Virtual threads are scheduled by the JVM onto carrier threads.

---

## Platform Thread vs Virtual Thread

| Platform Thread | Virtual Thread |
| --- | --- |
| Relatively heavier | Very lightweight |
| OS-backed execution | JVM-managed scheduling |
| More expensive to create at scale | Can create very large numbers |
| Good general-purpose choice | Excellent for many blocking I/O tasks |
| Limited practical concurrency | Designed for high concurrency |

Important:

> Virtual threads don't make CPU-intensive work magically faster.
> 

They primarily help with **scalability and efficient waiting**, especially for blocking operations.

---

### Virtual Threads and Thread Pools

A common mistake is trying to use a traditional fixed pool to limit virtual threads:

```
Executors.newFixedThreadPool(...)
```

Virtual threads are designed around a different model:

```
Executors.newVirtualThreadPerTaskExecutor()
```

If you need to limit access to a scarce resource, use an appropriate mechanism such as:

```
Semaphore
```

or the resource's own connection pool/limit.

---

## Structured Concurrency

Modern Java also provides APIs around structured concurrency in newer releases.

The idea is to treat related concurrent tasks as a single unit:

```
Parent Task
   |
   +--- Child Task A
   |
   +--- Child Task B
   |
   +--- Child Task C
```

Benefits include:

- Clear task ownership
- Better cancellation handling
- Easier error propagation
- Easier lifecycle management

For interviews, know the concept and check the exact Java version/API status expected by the employer.

---

# Common Multithreading Interview Questions

### Q1. What is a thread?

> A thread is a lightweight unit of execution within a process. Threads share process-level resources such as heap memory while maintaining their own stack and execution state.
> 

---

### Q2. `start()` vs `run()`?

> `start()` creates a new thread and causes `run()` to execute on that thread. Calling `run()` directly is simply a normal method invocation on the current thread.
> 

---

### Q3. Runnable vs Callable?

> Runnable doesn't return a result and cannot directly throw checked exceptions. Callable returns a result and can throw checked exceptions.
> 

---

### Q4. What is a race condition?

> A race condition occurs when multiple threads access shared mutable state concurrently and the outcome depends on their execution timing.
> 

---

### Q5. How do you make code thread-safe?

Possible approaches:

```
synchronized
Locks
Atomic classes
Concurrent collections
Immutable objects
Thread confinement
Proper volatile usage
```

---

### Q6. What does volatile do?

> `volatile` provides visibility and ordering guarantees for a variable across threads, but it does not make compound operations such as `count++` atomic.
> 

---

### Q7. What is deadlock?

> Deadlock occurs when two or more threads wait indefinitely for resources or locks held by each other.
> 

---

### Q8. How do you prevent deadlocks?

```
Consistent lock ordering
Minimize nested locks
Keep critical sections small
tryLock() with timeout
Avoid unnecessary shared state
```

---

### Q9. What is ExecutorService?

> ExecutorService manages asynchronous task execution using executor implementations and thread pools, allowing applications to reuse worker threads and control concurrency.
> 

---

### Q10. What is CompletableFuture?

> CompletableFuture represents an asynchronous computation and provides APIs for chaining, combining, handling errors, and coordinating asynchronous operations.
> 

---

### Q11. What is ConcurrentHashMap?

> ConcurrentHashMap is a thread-safe map designed for concurrent access, allowing multiple threads to operate on the map without requiring external synchronization for normal operations.
> 

---

### Q12. What is CountDownLatch?

> CountDownLatch allows one or more threads to wait until a specified number of operations have completed. It is generally one-shot.
> 

---

### Q13. CountDownLatch vs CyclicBarrier?

> CountDownLatch waits for a count to reach zero and generally cannot be reset, whereas CyclicBarrier allows a group of threads to wait for each other and can be reused.
> 

---

### Q14. What is ThreadLocal?

> ThreadLocal provides each thread with its own independent value, avoiding sharing for that particular state.
> 

---

### Q15. What are virtual threads?

> Virtual threads are lightweight JVM-managed threads designed to support very high concurrency, particularly for workloads involving blocking I/O.
> 

---

# Important Interview Scenario

### Problem

You have:

```java
class OrderService {
private int orderCount=0;
public void createOrder() {
orderCount++;
    }
}
```

100 threads call:

```
createOrder();
```

Expected:

```
100
```

But result may be lower.

### Solution 1 — synchronized

```java
public synchronized void createOrder() {orderCount++;
}
```

### Solution 2 — AtomicInteger

```java
private final AtomicInteger orderCount = new AtomicInteger();
public void createOrder() {
orderCount.incrementAndGet();
}
```

### Which would you choose?

If you only need a simple atomic counter:

```
AtomicInteger
```

is often appropriate.

If multiple pieces of shared state must be updated consistently:

```
synchronized
```

or an appropriate lock may be better.

---

# Practical Backend Example

Suppose your Java microservice receives an order:

```
POST /orders
       |
       ↓
Order Controller
       |
       ↓
Order Service
       |
       +---- Validate order
       |
       +---- Check inventory
       |
       +---- Calculate price
       |
       +---- Save order
       |
       +---- Publish event
```

Independent operations could potentially be executed concurrently:

```
                 Order
                   |
          +--------+--------+
          |                 |
          ↓                 ↓
    Inventory check    Price calculation
          |                 |
          +--------+--------+
                   |
                   ↓
              Save Order
```

For example, with `CompletableFuture`:

```java
CompletableFuture<Inventory> inventory = CompletableFuture.supplyAsync(
        () ->inventoryService.check(order));
 
CompletableFuture<Price> price = CompletableFuture.supplyAsync(
        () ->pricingService.calculate(order));
        
CompletableFuture<OrderResult> result = inventory.thenCombine(price,
        (inventoryResult,priceResult) ->createOrder(order,inventoryResult,priceResult)
    );
```

But concurrency should only be introduced when the operations are genuinely independent and the downstream systems can handle the parallelism.