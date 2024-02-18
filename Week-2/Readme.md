# Week 2

## Critical Sections in Concurrent Programming

In concurrent programming, managing access to shared resources is a key challenge. While locks are commonly used, a higher-level construct called a critical section, often referred to as an isolated construct, provides a more abstract and convenient approach.

### Example: Money Transfer Scenario

Consider a scenario involving a shared bank account where money transfers occur concurrently between two threads. Thread T1 deducts $100 from the parent's balance and adds $100 to the family balance. Thread T2 deducts $100 from the family balance and adds $100 to the daughter's balance.

```java
// Thread T1
isolate {
    // critical section for T1
    family.balance += 100;
    parent.balance -= 100;
}

// Thread T2
isolate {
    // critical section for T2
    family.balance -= 100;
    daughter.balance += 100;
}
```

In this scenario, without proper synchronization, interleaved reads and writes to the shared variable "family" can result in incorrect outcomes. For instance, if the read from T2 (R2) precedes the read from T1 (R1), and the subsequent writes (W2 and W1) occur in a specific order, the family balance might be incorrectly updated.

### Isolated Constructs

The isolated construct ensures that critical sections are executed in mutual exclusion. The specific implementation of mutual exclusion, whether through locks or other mechanisms like transactional memory, is abstracted away from the programmer. The notation isolate indicates that the two blocks of code within it cannot be interleaved.

```
// Thread T1
isolate {
    // critical section for T1
    family.balance += 100;
    parent.balance -= 100;
}

// Thread T2
isolate {
    // critical section for T2
    family.balance -= 100;
    daughter.balance += 100;
}
```

In Java, the isolate construct is not a standard language feature, and the example provided earlier is a conceptual representation. However, you can achieve similar functionality using locks or other synchronization mechanisms. Below is an example using the ReentrantLock class from the java.util.concurrent.locks package:

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class BankAccount {
    int balance;

    public BankAccount(int balance) {
        this.balance = balance;
    }
}

public class MoneyTransferExample {
    static Lock lock = new ReentrantLock();
    static BankAccount parent = new BankAccount(500);
    static BankAccount family = new BankAccount(1000);
    static BankAccount daughter = new BankAccount(0);

    public static void main(String[] args) {
        // Thread T1
        new Thread(() -> {
            lock.lock();
            try {
                family.balance += 100;
                parent.balance -= 100;
            } finally {
                lock.unlock();
            }
        }).start();

        // Thread T2
        new Thread(() -> {
            lock.lock();
            try {
                family.balance -= 100;
                daughter.balance += 100;
            } finally {
                lock.unlock();
            }
        }).start();
    }
}


```
