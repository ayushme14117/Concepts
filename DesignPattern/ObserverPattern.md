
🔹 Observer Pattern:

Observer Pattern is a Behavioral Design Pattern.

👉 Simple definition:

> When an object (Subject) changes state,
it's dependents (Observers) should automatically get notified without tight coupling.



Means:

One-to-many relationship

Loose coupling

Event-driven design



---

🔹 Real Life Example 📺 (YouTube Channel)

Subject:

👉 YouTube Channel

Observers:

👉 Subscribers

जब:

New video uploads
Then:

All subscribers will get notification. 🔔


Subscriber can be added/removeed
Channel does not know who are subscribers, it only notifies.


---

🔹 Structure (Roles)

Role	Meaning

Subject keeps State
Observer reacts on 	State change
ConcreteSubject	Actual implementation
ConcreteObserver	Actual listener



---

🔹 Simple Java Example

1️⃣ Observer Interface

interface Observer {
    void update(String message);
}


---

2️⃣ Subject Interface

interface Subject {
    void subscribe(Observer observer);
    void unsubscribe(Observer observer);
    void notifyObservers();
}


---

3️⃣ Concrete Subject

class YouTubeChannel implements Subject {

    private List<Observer> observers = new ArrayList<>();
    private String videoTitle;

    public void uploadVideo(String title) {
        this.videoTitle = title;
        notifyObservers();
    }

    public void subscribe(Observer observer) {
        observers.add(observer);
    }

    public void unsubscribe(Observer observer) {
        observers.remove(observer);
    }

    public void notifyObservers() {
        for (Observer o : observers) {
            o.update(videoTitle);
        }
    }
}


---

4️⃣ Concrete Observer

class Subscriber implements Observer {
    private String name;

    Subscriber(String name) {
        this.name = name;
    }

    public void update(String videoTitle) {
        System.out.println(
            name + " notified: New video uploaded - " + videoTitle
        );
    }
}


---

5️⃣ Client Code

YouTubeChannel channel = new YouTubeChannel();

Subscriber s1 = new Subscriber("Amit");
Subscriber s2 = new Subscriber("Neha");

channel.subscribe(s1);
channel.subscribe(s2);

channel.uploadVideo("Observer Pattern Explained");


---

🔁 Flow

Upload Video
 ↓
YouTubeChannel (Subject)
 ↓
Notify all Subscribers (Observers)


---

🌱 Observer Pattern in Spring (Real World)

1️⃣ Spring Events (BEST example)

ApplicationEventPublisher publisher;

publisher.publishEvent(new OrderCreatedEvent(orderId));

2️⃣ Observers (Listeners)

@EventListener
public void sendEmail(OrderCreatedEvent event) {
    // send email
}

@EventListener
public void updateInventory(OrderCreatedEvent event) {
    // update stock
}

👉 Multiple listeners, zero coupling.


---

🔹 Where Observer is Used in Real Systems

UI event handling (button click)

Messaging systems

Stock price updates

Notification services

Cache invalidation

Spring Application Events

Kafka consumers (conceptually)



---

🔹 Observer vs Strategy (Quick Diff)

Observer	Strategy

Event based	Algorithm selection
One-to-many	One-to-one
Push notifications	Replace behavior



---

⚠️ Drawbacks (Realistic)

❌ Memory leaks (observers not removed)
❌ Order of notification not guaranteed
❌ Debugging hard (hidden flow)


---

🎯 Interview One-Liner ⭐

> Observer Pattern defines a one-to-many dependency so that when one object changes state, all its dependents are notified automatically.




---

Next:

UML diagram

Push vs Pull observer

Java built-in Observer (deprecated)

Kafka vs Observer comparison
