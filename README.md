# Parallel Programming in Java

## Java Threads

In the context of concurrent programming, threads serve as fundamental building blocks. Java, unlike some earlier mainstream programming languages, incorporated the concept of threads into its language definition from its inception.

When an instance of the Thread class in Java is created using the new operation, it doesn't immediately start executing. Instead, it can commence execution only when its `start()` method is invoked. The specific statement or computation to be executed by the thread is provided as a parameter to the constructor.

The Thread class also provides a waiting operation through its `join()` method. When a thread t0 initiates a `t1.join()` call, t0 will be required to wait until thread t1 completes. Afterward, it can safely access any values computed by t1. It's worth noting that there are no restrictions on which thread can perform a join on another thread. However, it's essential for a programmer to be cautious, as incorrectly using join operations can lead to the creation of a deadlock cycle. (A deadlock occurs when two threads wait for each other indefinitely, preventing any progress from being made.)

## Structured Locks

Imagine a shared buffer between a producer and a consumer, a simple yet common scenario in concurrent programming. The `SharedBuffer` class encapsulates this shared resource, and we employ the `synchronized` keyword to manage access. The `produce` method represents a producer adding data to the buffer, while the `consume` method signifies a consumer retrieving data. Both methods are synchronized to ensure exclusive access, preventing conflicts that might arise if multiple threads attempt to modify the buffer simultaneously.

Within each method, there's a crucial aspect of coordination using `wait()` and `notify()`. When the producer enters the `produce` method and finds data already available (`dataAvailable` is true), it patiently waits by invoking `wait()`. This temporarily releases the lock, allowing other threads to execute. On the flip side, the consumer, when attempting to consume data from an empty buffer, also calls `wait()` to wait for the producer to fill it.

The magic happens when data is produced or consumed. In the `produce` method, once data is added to the buffer, we set `dataAvailable` to true and notify the waiting consumer using `notify()`. Conversely, in the `consume` method, after data is consumed, `dataAvailable` is set to false, and a notification is sent to the producer.

This dance of `wait()` and `notify()` ensures that the producer and consumer synchronize their actions. If the buffer is full, the producer waits; if it's empty, the consumer waits. As they produce and consume, they signal each other, allowing for a seamless, coordinated interaction, avoiding conflicts and ensuring the integrity of the shared resource. This encapsulates the essence of synchronization and communication in concurrent Java programming.

```java
class SharedBuffer {
    private int data;
    private boolean dataAvailable = false;

    // Producer adds data to the buffer
    public synchronized void produce(int value) {
        while (dataAvailable) {
            try {
                wait(); // Wait if data is available
            } catch (InterruptedException e) {
                // Handle interruption
            }
        }
        data = value;
        dataAvailable = true;
        System.out.println("Produced: " + value);
        notify(); // Notify consumer that data is available
    }

    // Consumer retrieves data from the buffer
    public synchronized int consume() {
        while (!dataAvailable) {
            try {
                wait(); // Wait if data is not available
            } catch (InterruptedException e) {
                // Handle interruption
            }
        }
        int consumedData = data;
        dataAvailable = false;
        System.out.println("Consumed: " + consumedData);
        notify(); // Notify producer that buffer is empty
        return consumedData;
    }
}


```

## Unstructured Locks

### Hand over hand locking
**Hand-over-hand locking** is a synchronization technique used in concurrent programming where threads acquire and release locks in a sequential, linked manner. This approach is often applied to scenarios involving linked data structures, such as linked lists, where threads need to perform operations on adjacent nodes in a specific order.

The key characteristic of hand-over-hand locking is that a thread acquires a lock for one node, performs some operation, releases that lock, and then acquires the lock for the next node in the sequence. This pattern continues as the thread moves through the linked structure.

Here's a high-level description of how hand-over-hand locking works:

1. **Initialization:** Each node in the linked structure is associated with a lock. These locks are distinct objects and need to be explicitly allocated.

2. **Lock Acquisition and Release:** As a thread traverses the linked structure, it acquires and releases locks for adjacent nodes in a sequential manner.

3. **Coordination:** This approach ensures that at any given time, a thread holds locks for two adjacent nodes, allowing it to work with the data associated with those nodes while avoiding conflicts with other threads.

4. **Explicit Lock Management:** Unlike structured locks, where locks are automatically acquired and released based on synchronized blocks or methods, hand-over-hand locking requires explicit calls to lock and unlock individual locks.

Here's a simplified example in pseudo-code to illustrate hand-over-hand locking:

```java
class Node {
    // Node-related data and methods
}

class LinkedListProcessor {
    private final Object lockN1 = new Object();
    private final Object lockN2 = new Object();
    private final Object lockN3 = new Object();

    public void processNodes() {
        // Thread 1
        synchronized (lockN1) {
            // Work with Node N1
        }
        synchronized (lockN2) {
            // Work with Node N2
        }

        // Thread 2
        synchronized (lockN2) {
            // Work with Node N2
        }
        synchronized (lockN3) {
            // Work with Node N3
        }

        // More threads and nodes can be added following the same pattern
    }
}
```

### tryLock()

The **tryLock** method is a mechanism in concurrent programming that attempts to acquire a lock and returns a boolean value indicating whether the lock acquisition was successful. Unlike traditional lock acquisition methods that might block if the lock is not available, tryLock provides a non-blocking alternative, allowing threads to take alternative actions if the lock is unavailable.

```java
class Example {
    private final Object lock = new Object();

    public void performTask() {
        boolean success = false;
        try {
            success = tryLock(); // tryLock returns true if the lock is acquired
            if (success) {
                // Code when lock is acquired
                System.out.println("Lock acquired. Performing task...");
            } else {
                // Code when lock is unavailable
                System.out.println("Lock unavailable. Performing alternative task...");
            }
        } finally {
            if (success) {
                unlock(); // Release the lock if acquired
            }
        }
    }
}
```

### Read Write Lock

A *read-write lock* is a synchronization mechanism used in concurrent programming to control access to a shared resource, allowing multiple threads to read the resource simultaneously while providing exclusive access for write operations. This type of lock is designed to improve performance in scenarios where the majority of operations are reads, and write operations are less frequent.

A read-write lock typically consists of two locks:

1. **Read Lock:** Multiple threads can simultaneously acquire the read lock, allowing them to perform read operations on the shared resource concurrently. Reading operations do not modify the shared resource, so they can be performed concurrently without conflicting with each other.

2. **Write Lock:** Only one thread at a time can acquire the write lock. This exclusive access ensures that write operations are performed atomically and that no other thread is reading or writing to the resource during a write operation.

The main advantage of a read-write lock is that it maximizes concurrency for read operations while still providing exclusive access for write operations when needed. This can significantly improve performance in situations where reads are more frequent than writes.

In the context of a typical read-write lock implementation, when one thread holds a write lock (exclusive access for writing), no other thread is allowed to acquire a read lock (shared access for reading). This exclusion ensures that while a thread is performing a write operation, no other threads can concurrently perform read operations, and vice versa.

The purpose of this restriction is to maintain data consistency. A write operation can modify the shared resource, and allowing other threads to read from the resource while it's being modified could lead to inconsistent or incorrect results. Therefore, the read-write lock ensures that when a thread is performing a write operation, it has exclusive access to the resource.



Here's a simple example in Java using the `ReentrantReadWriteLock` class, which is part of the `java.util.concurrent.locks` package:

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class SharedResource {
    private int data;
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();

    public int readData() {
        rwLock.readLock().lock();
        try {
            // Read operation on the shared resource
            return data;
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public void writeData(int newData) {
        rwLock.writeLock().lock();
        try {
            // Write operation on the shared resource
            data = newData;
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}
