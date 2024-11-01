# AtomicInteger

The java.util.concurrent.atomic.AtomicInteger class in Java, is used for handling integer values in a way that is **safe for concurrent access by multiple threads** without requiring explicit synchronization (such as synchronized blocks).

**Key Uses of AtomicInteger in Java**
**Atomic Operations:** AtomicInteger provides atomic operations like get(), set(), incrementAndGet(), decrementAndGet(), addAndGet(), and compareAndSet(). These methods ensure that the entire operation is performed atomically, which is crucial in multi-threaded applications.

**Counters in Concurrency:** AtomicInteger is often used as a counter shared among multiple threads. For instance, if multiple threads need to increment a shared counter, AtomicInteger provides thread-safe increments via incrementAndGet() and getAndIncrement() without requiring synchronized methods.

**Efficient Thread-Safe Updates:** By using atomic operations, AtomicInteger avoids the need for explicit synchronization (like using synchronized or Lock). This reduces overhead and increases performance when compared to traditional synchronization, especially in high-concurrency scenarios.

# Using AtomicInteger
Each thread increments the counter 1,000 times, and incrementAndGet() ensures each increment is atomic.
The final output will be 30,000 (30 threads x 1,000 increments), with no race conditions or inconsistencies.
```java
import java.util.concurrent.atomic.AtomicInteger;

public class Main {
   private static AtomicInteger counter = new AtomicInteger(0);

   public static void main(String[] args) throws InterruptedException {
      // Create 30 threads that increment the counter
      Thread[] threads = new Thread[30];
      for (int i = 0; i < threads.length; i++) {
         threads[i] = new Thread(() -> {
            for (int j = 0; j < 1000; j++) {
               counter.incrementAndGet(); // Atomic increment
            }
         });
         threads[i].start();
      }
      // Wait for all threads to complete
      for (Thread t : threads) {
         t.join();
      }
      // Output final counter value
      System.out.println("Final counter value: " + counter.get()); // it will always print 30000
   }
}

```

# Understanding the issue in the multithreaded application 
In a multi-threaded environment, operations on int or Integer are not atomic. For example, counter++ (increment operation) is actually a compound operation:
1. Read the current value of counter.
2. Increment it by 1.
3. Write the new value back to counter.

**Without atomicity, two threads can read the same value at the same time, both increment it, and then write the same result back, which means some increments will be lost.**
```java
public class Main {
   private static int counter = 0;

   public static void main(String[] args) throws InterruptedException {
      Thread[] threads = new Thread[30];
      for (int i = 0; i < threads.length; i++) {
         threads[i] = new Thread(() -> {
            for (int j = 0; j < 1000; j++) {
               counter++; // Non-atomic increment
            }
         });
         threads[i].start();
      }
      for (Thread t : threads) {
         t.join();
      }

      System.out.println("Final counter value: " + counter); // final value may differ every time
   }
}

```

## Using Synchronization with int or Integer :
To avoid race conditions, we can synchronize access to the counter variable
```java
public class Main {
    private static int counter = 0;
    public static synchronized void incrementCounter() {
        counter++;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread[] threads = new Thread[30];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    incrementCounter();  // Synchronized increment
                }
            });
            threads[i].start();
        }
        for (Thread t : threads) {
            t.join();
        }

        System.out.println("Final counter value: " + counter); // this will always print 30000
    }
}
```
**Using Synchronization:** By adding synchronized to the incrementCounter() method, we ensure that only one thread can increment the counter at a time.
**Drawback:** This approach, while correct, is slower because it adds the overhead of acquiring and releasing a lock with each increment operation.

