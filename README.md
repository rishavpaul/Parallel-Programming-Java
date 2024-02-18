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

```
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
