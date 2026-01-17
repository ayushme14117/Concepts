## 1️⃣ Blocking Queue

**BlockingQueue** is a **Java interface** which provides **thread safe queue** with specified number of permits. The main purpose of it is to solve **Produce-Consumer problem**.
It automatically handles the thread block/unblock when queue is full/empty.

* **Producer**: Produces the data.
* **Consumer**: Consumes the data.

---

## 2️⃣ Key Features

| Feature                        | Explanation                                                                                                            |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| Thread-safe                    | Multiple threads can use it simultaneously                                                                             |
| Blocking behavior              | Producer waits until queue does not have space. Consumer waits until queue is not empty.                               |
| No need for manual wait/notify | BlockingQueue internally manages synchronization।                                                                      |
| Common implementations         | `ArrayBlockingQueue`, `LinkedBlockingQueue`, `PriorityBlockingQueue`, `DelayQueue`                                     |

---

## 3️⃣ Methods of BlockingQueue

| Method                      | Behavior                                                        |
| --------------------------- | --------------------------------------------------------------- |
| `put(E e)`                  | Adds element to queue, waits if queue is full.                  |
| `take()`                    | Removes element, waits if queue is empty.                       |
| `offer(E e, timeout, unit)` | Tries to add element within timeout, otherwise fails.           |
| `poll(timeout, unit)`       | Tries to remove element within timeout, otherwise returns null. |

---

## 4️⃣ Working Example

```java
import java.util.concurrent.*;

public class BlockingQueueExample {
    public static void main(String[] args) {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);

        // Producer Thread
        Thread producer = new Thread(() -> {
            for (int i = 1; i <= 10; i++) {
                try {
                    queue.put(i); // wait if queue is full
                    System.out.println("Produced: " + i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        // Consumer Thread
        Thread consumer = new Thread(() -> {
            for (int i = 1; i <= 10; i++) {
                try {
                    int data = queue.take(); // wait if queue is empty
                    System.out.println("Consumed: " + data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        producer.start();
        consumer.start();
    }
}
```

**Explanation:**

1. Queue size: 5
2. Producer wants to add 10 items
3. Queue full → Producer waits
4. Consumer consumes 10 items 
5. Queue empty → Consumer waits
6. automatic synchronization happens, **manual wait/notify is not required**

---

💡 **Key Point:**

* BlockingQueue provides **thread-safe and blocking operations**
* Perfect for **producer-consumer patterns**।

---

