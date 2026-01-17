## 1️⃣ Semaphore

**Definition:**  
A **Semaphore** is a **thread-synchronization construct** that controls access to a shared resource by multiple threads using a **set number of permits**.  

- Think of it as a **traffic signal** for threads.  
- Only a limited number of threads can access a resource simultaneously.  

---

## 2️⃣ Key Points

| Feature | Explanation |
|---------|------------|
| **Permits** | Semaphore has a counter representing the number of threads allowed to access the resource. |
| **Acquire** | A thread calls `acquire()` to get permission; if no permits are available, it **waits**. |
| **Release** | A thread calls `release()` after finishing, returning the permit. |
| **Fairness** | Can be **fair** (FIFO) or non-fair. |
| **Types** | `CountingSemaphore` (permits > 1), `BinarySemaphore` (permits = 1, acts like a mutex). |

---

## 3️⃣ Real-world Analogy

- Imagine a **public bathroom with 3 toilets**:  
  - **Semaphore permits = 3**  
  - If all toilets are occupied → next person **waits**.  
  - Once someone leaves → permit is released → next person enters.  

---

## 4️⃣ Java Example

```java
import java.util.concurrent.*;

public class SemaphoreExample {
    public static void main(String[] args) {
        // Semaphore with 3 permits
        Semaphore semaphore = new Semaphore(3);

        Runnable task = () -> {
            String threadName = Thread.currentThread().getName();
            try {
                System.out.println(threadName + " is waiting for a permit.");
                semaphore.acquire(); // acquire a permit
                System.out.println(threadName + " got a permit and is accessing the resource.");
                Thread.sleep(2000); // simulate resource usage
                System.out.println(threadName + " is releasing the permit.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release(); // release permit
            }
        };

        // Create 6 threads trying to access the resource
        for (int i = 1; i <= 6; i++) {
            new Thread(task, "Thread-" + i).start();
        }
    }
}
```

**Explanation:**
1. Semaphore initialized with **3 permits** → max 3 threads can access simultaneously.  
2. Other threads **wait** until a permit is released.  
3. After usage, thread calls `release()` → next waiting thread gets the permit.  

---

## 5️⃣ Sample Output

```
Thread-1 is waiting for a permit.
Thread-1 got a permit and is accessing the resource.
Thread-2 is waiting for a permit.
Thread-2 got a permit and is accessing the resource.
Thread-3 is waiting for a permit.
Thread-3 got a permit and is accessing the resource.
Thread-4 is waiting for a permit.
Thread-5 is waiting for a permit.
Thread-6 is waiting for a permit.
Thread-1 is releasing the permit.
Thread-4 got a permit and is accessing the resource.
Thread-2 is releasing the permit.
Thread-5 got a permit and is accessing the resource.
Thread-3 is releasing the permit.
Thread-6 got a permit and is accessing the resource.
...
```

**Observation:**  
- Only **3 threads** can run at the same time.  
- Remaining threads **wait** until a permit becomes available.  

---

## 6️⃣ Key Takeaways

- **Semaphore** is perfect for **resource-limiting scenarios** like DB connections, thread pools, or rate-limiting.  
- **Binary Semaphore (permits=1)** acts like a **mutex**.  
- Supports **fairness policy** to avoid starvation (`new Semaphore(3, true)`).  

---
