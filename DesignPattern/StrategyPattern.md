Strategy Pattern:

Strategy Pattern is a Behavioral Design Pattern.

👉 Definition (simple words):

> Defines a family of algorithms,
each algorithm will be kept class,
decide in runtime which algorithm will be used.



means:

if-else / switch will not be required 

Behavior can change dynamically

Open–Closed Principle followed



---

🔹 Real Life Example 🛣️ (Navigation App)

lets say we have an app like Google Maps.

User has options:

🚗 Car route

🚲 Bike route

🚶 Walk route


Logic for Route calculate is different in each case.

If we use if-else:

if(type == "CAR") { ... }
else if(type == "BIKE") { ... }
else if(type == "WALK") { ... }

❌ Bad design, hard to extend


---

🔹 Strategy Pattern Solution

1️⃣ Strategy Interface

interface RouteStrategy {
    void buildRoute();
}


---

2️⃣ Concrete Strategies

class CarRouteStrategy implements RouteStrategy {
    public void buildRoute() {
        System.out.println("Building car route");
    }
}

class BikeRouteStrategy implements RouteStrategy {
    public void buildRoute() {
        System.out.println("Building bike route");
    }
}

class WalkRouteStrategy implements RouteStrategy {
    public void buildRoute() {
        System.out.println("Building walking route");
    }
}


---

3️⃣ Context Class

class NavigationApp {
    private RouteStrategy strategy;

    void setStrategy(RouteStrategy strategy) {
        this.strategy = strategy;
    }

    void buildRoute() {
        strategy.buildRoute();
    }
}


---

4️⃣ Client Code (Runtime Decision)

NavigationApp app = new NavigationApp();

app.setStrategy(new CarRouteStrategy());
app.buildRoute();

app.setStrategy(new WalkRouteStrategy());
app.buildRoute();


---

🔁 How this works (Flow)

Client
 ↓
Context (NavigationApp)
 ↓
Selected Strategy (Car / Bike / Walk)


---

🔹 One More Real Software Example 💳 (Payment System)

Payment methods:

Credit Card

UPI

PayPal


Each has different logic, same interface:

interface PaymentStrategy {
    void pay(int amount);
}

Switch payment method at runtime — perfect Strategy use case.


---

🔹 Strategy vs Decorator (Quick Difference)

Strategy	Decorator

Algorithm choose करना	Behavior add करना
One strategy at a time	Multiple decorators chain
Replace behavior	Enhance behavior



---

🔹 When to use Strategy Pattern?

✔ :

Multiple algorithms

Runtime selection needed

want to remove if-else 


❌ :

Logic is very simple

only 1 algorithm



---

🎯 Interview One-Liner ⭐

> Strategy Pattern allows selecting an algorithm’s behavior at runtime by encapsulating each algorithm in a separate class.




---
