# Design Patterns — Notes

For each pattern know: **problem / how / where used.** Skip UML.

## The 3 families

| Family | Answers | Patterns |
|--------|---------|----------|
| **Creational** | How to **create** objects | Singleton, Factory, Builder |
| **Structural** | How to **compose** objects | Adapter, Decorator, Facade |
| **Behavioral** | How objects **talk** | Observer, Strategy |

---

# CREATIONAL

## 1. Singleton

**Problem (2 things for the same instance):**
1. Ensure **exactly one instance** of a class — controls access to a **shared resource** (DB, config, cache).
2. Provide a **safe global access point** — reachable from anywhere, but can't be overwritten.

**How:** private constructor + one static instance + static `getInstance()`.
> Impossible with a normal constructor — `new` always returns a fresh object.

**Where:** DB connection pool, config, logger, cache.

```java
class DatabaseConnection {
    private static DatabaseConnection instance;
    private DatabaseConnection() { /* open real DB connection */ }

    public static DatabaseConnection getInstance() {
        if (instance == null) instance = new DatabaseConnection();
        return instance;                          // always same object
    }
    public void query(String sql) { }
}
```

### Thread-safety (the follow-up they LOVE)
Basic version is **not** thread-safe: two threads can pass `instance == null` together → two instances. Three fixes:

**Option 1 — `synchronized` method** (simple, but locks every call → slow)
```java
public static synchronized DatabaseConnection getInstance() {
    if (instance == null) instance = new DatabaseConnection();
    return instance;
}
```

**Option 2 — Double-Checked Locking** (fast, locks only on first creation)
```java
private static volatile DatabaseConnection instance;    // volatile = every thread sees latest value

public static DatabaseConnection getInstance() {
    if (instance == null) {                             // check 1: no lock, fast
        synchronized (DatabaseConnection.class) {
            if (instance == null) {                     // check 2: safety inside lock
                instance = new DatabaseConnection();
            }
        }
    }
    return instance;
}
```

**Option 3 — Enum Singleton** ⭐ (best in Java — thread-safe by design)
```java
enum DatabaseConnection {
    INSTANCE;
    public void query(String sql) { }
}
// usage
DatabaseConnection.INSTANCE.query("SELECT * FROM users");
```

| Option | Speed | Complexity |
|--------|-------|------------|
| synchronized method | 🐢 slow (locks every call) | simple |
| double-checked locking | 🐇 fast | complex + needs `volatile` |
| enum | ⚡ fastest | shortest + safest |

### Watch-outs
- Hard to unit-test (global state, hard to mock).
- Hides dependencies.
- Breaks **SRP** slightly (manages own instance + does real job). Overuse → anti-pattern.

**How exactly does it break SRP?** The class does 2 jobs = 2 reasons to change:
1. **Lifecycle** — managing the single instance (`instance`, `getInstance()`, thread-safety).
2. **Real work** — the actual business logic (`query()`, `connect()`).

Change the singleton strategy (e.g. synchronized → enum) OR change how queries work → both edit the same class.

> 💡 Fix: separate instance creation into a **Factory** (next pattern). Then one class handles the real work, another handles creation.

### Interview answer (polished)

"الـ **Singleton** بيحل مشكلة إني محتاجة **only one instance** من class معيّن، وبيديني **global access point** آمن ليه — أي جزء في الـ app بيشوف **نفس الـ instance**.

مثال: الـ **DB connection**. دي **shared resource**، مفيش داعي أعمل مليون connection.

الـ follow-up المشهور: **هل هو thread-safe؟** لأ، مش by default. لو اتنين thread نادوا `getInstance()` في نفس اللحظة والـ instance لسه `null`، الاتنين هيعدّوا الشيك ويعملوا **instance جديد** كل واحد → كسر الـ Singleton.

الحل: أعمل **lock** في `getInstance()`. عندي **3 options**:

1. **`synchronized` على الميثود** — أبسط حل، بس بيقفل **كل مرة** الميثود بتتنادى حتى بعد ما الـ instance اتعمل → **بطيء**.
2. **Double-checked locking مع `volatile`** — بشيك مرتين، مرة **بره القفل** (سريع) ومرة **جوّاه** (أمان). كده بقفل **بس أول مرة**، بعد كده كل الاستدعاءات تعدّي بدون قفل. الـ **`volatile`** مهمة عشان كل الـ threads يشوفوا آخر قيمة من الـ **memory** مش من الـ **CPU cache**.
3. **Enum Singleton** — الأحسن في Java. **thread-safe by design** من غير ما أكتب أي قفل، ومحمي من **reflection** و **serialization** tricks."

**Bonus lines (لو دقّق):**
- **إمتى أختار كل واحدة؟** →
    - لو الميثود بتتنادى شوية → `synchronized` بسيط ومكفّي.
    - لو كتير → double-checked.
    - لو Java → **enum أحسن**.
- **عيوب Singleton عمومًا؟** →
    - صعب في **unit-testing**.
    - بيخبّي الـ **dependencies**.
    - وبيكسر **SRP** شوية (بيدير نفسه + بيشتغل).

---

## 2. Factory Method

**Problem:** class معيّن بيبقى مسؤول عن **إنشاء objects** من كذا نوع (`new Visa()`, `new MasterCard()`...) → **بيكسر SRP + OCP**.

**Idea:** **encapsulate object creation logic** in one class. أي جزء محتاج object → يسأل الـ factory بدل ما يعمل `new` بنفسه.

**Where used:** payment methods, notification channels (email/SMS/push), shape drawing, DB drivers.

### Code (payment example)
```java
enum CardType { VISA, MASTER_CARD, AMERICAN_EXPRESS }

interface PaymentMethod {
    void authorize();
    void startMoneyTransfer();
}

class Visa implements PaymentMethod {
    public void authorize() { }
    public void startMoneyTransfer() { }
}
class MasterCard implements PaymentMethod { /* ... */ }
class AmericanExpress implements PaymentMethod { /* ... */ }

// ⭐ Factory — one class, one job: create PaymentMethod
class PaymentMethodFactory {
    public PaymentMethod createPaymentMethod(CardType type) {
        switch (type) {
            case VISA:              return new Visa();
            case MASTER_CARD:       return new MasterCard();
            case AMERICAN_EXPRESS:  return new AmericanExpress();
            default: throw new IllegalArgumentException("Unknown: " + type);
        }
    }
}

// PaymentProcessor doesn't know concrete classes — only the interface
class PaymentProcessor {
    private PaymentMethodFactory factory;
    public PaymentProcessor(PaymentMethodFactory factory) { this.factory = factory; }

    public void process(CardType type) {
        PaymentMethod payment = factory.createPaymentMethod(type);
        payment.authorize();
        payment.startMoneyTransfer();
    }
}
```

### SOLID keywords it satisfies
- **SRP** — `PaymentProcessor` does processing, factory does creation.
- **OCP** — new payment method = new class + one line in factory. `PaymentProcessor` never touched.
- **DIP** — `PaymentProcessor` depends on `PaymentMethod` interface, not concrete classes.

### 💡 Fixes the Singleton SRP issue
Singleton mixes **lifecycle** (managing instance) + **real work** (queries). Factory splits them:
```java
class DatabaseConnection {                          // ← real work only
    public void query(String sql) { }
}
class DatabaseConnectionFactory {                   // ← creation only
    private static DatabaseConnection instance;
    public static DatabaseConnection getInstance() {
        if (instance == null) instance = new DatabaseConnection();
        return instance;
    }
}
```
Now each class has **one reason to change** → SRP restored.

### Honest note (say if pressed)
Factory doesn't fully close OCP — you still edit it when adding a new type. But it **contains** the change in one small class whose only job is creation. For 100% closed → use a **registry** or **reflection**.

### Interview answer
"الـ **Factory** بـ**encapsulate object creation logic** في class واحد — بدل ما كل جزء في الكود يعمل `new` بنفسه، بيرجع للـ factory.

مثال: عندي `PaymentProcessor` بيعمل process على `PaymentMethod` interface، وعندي concrete classes زي **Visa, MasterCard, PayPal**. بدل ما `PaymentProcessor` يعرف كل الأنواع ويعمل `if/else`، بيسأل الـ **`PaymentMethodFactory`** بالـ `cardType` وترجّعله الـ **PaymentMethod instance جاهزة**.

بحل مشكلتين:
1. **SRP** — `PaymentProcessor` بس بيـ**process**، مش بيـ**create**.
2. **OCP** — payment method جديدة؟ concrete class جديد + سطر في الـ factory. **`PaymentProcessor` ما بيتلمسش**.

Honest note: الـ factory نفسه لسه بيتعدّل لما أضيف نوع، فمش OCP مثالي 100% — بس بيركّز التغيير في مكان مسؤوليته الوحيدة الإنشاء. لو محتاجة أقفله كمان → **registry** أو **reflection**."

### Bonus: Abstract Factory (the follow-up)
- **Factory Method:** creates **one object** of a type (Visa vs MasterCard — all `PaymentMethod`).
- **Abstract Factory:** creates a **whole family of related objects** that must stay consistent together.

**When to use:** you have multiple **families**, and objects inside one family must not be mixed with another (e.g. Stripe stack ≠ PayPal stack).

```java
interface Card {} interface Validator {} interface Gateway {}

// Stripe family
class StripeCard implements Card {}
class StripeValidator implements Validator {}
class StripeGateway implements Gateway {}
// PayPal family — parallel structure
class PayPalCard implements Card {}
class PayPalValidator implements Validator {}
class PayPalGateway implements Gateway {}

// ⭐ Abstract Factory — creates the whole family
interface PaymentFactory {
    Card createCard();
    Validator createValidator();
    Gateway createGateway();
}
class StripeFactory implements PaymentFactory {
    public Card createCard()           { return new StripeCard(); }
    public Validator createValidator() { return new StripeValidator(); }
    public Gateway createGateway()     { return new StripeGateway(); }
}
class PayPalFactory implements PaymentFactory { /* PayPal versions */ }

// pick the family once → all pieces stay consistent, can't mix by accident
PaymentFactory factory = new StripeFactory();
```

**Real-world:** UI themes (Windows/Mac families), DB drivers (MySQL/Postgres families), cloud SDKs (AWS/Azure).

**Interview line:** "Factory Method creates **one object**; Abstract Factory creates a **family of related objects** that must stay consistent together — like UI themes or DB drivers."

---

## 3. Builder

**Problem:** class with **many optional fields**. Two bad options:
- **Telescoping constructors** — one constructor per combination → explodes fast.
- **Setters** — object stays **half-built** after `new` (invalid state, easy to forget a required field).

**Idea:** helper class builds the object step-by-step via **method chaining** (**fluent interface**), then `build()` returns the final, complete, immutable object.

**Where used:** `StringBuilder`, `HttpRequest.Builder`, SQL query builders, Lombok's `@Builder`.

### Code
```java
class Pizza {
    private final String dough;         // final → immutable after build
    private final String sauce;
    private final String cheese;
    private final boolean pepperoni;
    private final boolean mushrooms;

    private Pizza(Builder b) {          // private → only Builder can create
        this.dough     = b.dough;
        this.sauce     = b.sauce;
        this.cheese    = b.cheese;
        this.pepperoni = b.pepperoni;
        this.mushrooms = b.mushrooms;
    }

    public static class Builder {
        private String dough;           // required
        private String sauce;           // required
        private String cheese;          // optional
        private boolean pepperoni;      // optional
        private boolean mushrooms;      // optional

        public Builder(String dough, String sauce) {   // required in constructor
            this.dough = dough;
            this.sauce = sauce;
        }

        public Builder cheese(String cheese) { this.cheese = cheese; return this; }
        public Builder pepperoni(boolean v)  { this.pepperoni = v;   return this; }
        public Builder mushrooms(boolean v)  { this.mushrooms = v;   return this; }

        public Pizza build() {
            // validation here — object never leaves incomplete
            return new Pizza(this);
        }
    }
}

// usage — reads like English
Pizza p = new Pizza.Builder("thin", "tomato")
                    .cheese("mozzarella")
                    .pepperoni(true)
                    .build();
```

### Benefits (keywords)
| Benefit | How |
|---------|-----|
| **Readability** | reads like a sentence |
| **Immutable object** | fields `final`, nothing changes after `build()` |
| **No constructor overloading** | handles optional fields cleanly |
| **Validation in `build()`** | object never exists in an invalid state |

### Builder vs Factory
| Factory | Builder |
|---------|---------|
| Chooses **which class** to create (variants) | Builds **one complex object** step-by-step |
| Returns object in **one step** | Returns after **many steps** + `build()` |
| For **variants of a type** | For **objects with many optional fields** |

### Interview answer
"الـ **Builder** بيشتغل لما عندي class فيه **كتير optional fields**. بدل ما أعمل **كذا نسخة من الـ constructor** أو أستخدم **setters** (اللي بيخلوا الـ object ناقص لحد ما تخلصي)، الـ Builder بيبنيلي الـ object step-by-step عبر **method chaining** (**fluent interface**)، وفي الآخر بينده `build()` بيرجّعلي الـ object **جاهز ومتناسق ومحصّن (immutable)**.

مثال: `new Pizza.Builder("thin", "tomato").cheese("mozzarella").pepperoni(true).build()`. أمثلة معروفة في Java: **`StringBuilder`**، **`HttpRequest.Builder`**، **Lombok's `@Builder`**."

---

# STRUCTURAL

> **Structural patterns** focus on **how to compose objects** — connecting pieces together so they work as a larger structure, without changing the pieces themselves.

## 4. Adapter

**Problem:** you have a class (or library) whose **interface is incompatible** with your code. You can't edit it (third-party or legacy).

**Idea:** a wrapper class in the middle that **translates** between the two interfaces. Like a phone charger adapter — doesn't change the socket or the phone, just connects them.

**Where used:** third-party SDK integration, legacy system wrapping, API format conversion, `Arrays.asList()` in Java (adapts array → List interface).

### Code (payment example)

```java
// Your code expects this interface
interface PaymentProcessor {
    void pay(double amount);
}

// Third-party SDK — you CAN'T edit this
class StripeAPI {
    void makePayment(int amountInCents) { /* Stripe logic */ }
}

// Adapter — translates your interface → Stripe's interface
class StripeAdapter implements PaymentProcessor {
    private StripeAPI stripe;

    public StripeAdapter(StripeAPI stripe) {
        this.stripe = stripe;
    }

    public void pay(double amount) {
        int cents = (int)(amount * 100);    // convert dollars → cents
        stripe.makePayment(cents);           // delegate to Stripe
    }
}

// usage — your code sees PaymentProcessor, doesn't know Stripe exists
PaymentProcessor processor = new StripeAdapter(new StripeAPI());
processor.pay(29.99);
```

### The analogy

```
Your Phone (USB-C) ──► Adapter ──► Wall Socket (220V)
Your Code (pay())  ──► StripeAdapter ──► StripeAPI (makePayment())
```

Adapter **doesn't change** either side — just **translates** between them.

### SOLID keywords it satisfies
- **OCP** ⭐ — new payment provider = new Adapter class. Your code **never touched**.
- **DIP** — your code depends on `PaymentProcessor` interface, not Stripe directly.
- **SRP** — Adapter's only job is **translation**.

### Interview answer

"**Adapter** solves the problem of having an **incompatible interface** — typically a third-party library or legacy class whose methods don't match what your code expects.

Example: my code uses `PaymentProcessor.pay(double)`, but Stripe SDK has `makePayment(int cents)`. I can't edit Stripe. So I create `StripeAdapter implements PaymentProcessor` — inside `pay()` it converts dollars to cents and delegates to `stripe.makePayment()`.

New provider? **New adapter class. My code never changes** — it only knows the `PaymentProcessor` interface."

---

## 5. Decorator

**Problem:** you need to **add behavior** to an object **at runtime** without modifying its class. Inheritance can't do this — it's fixed at compile time, and combining multiple extensions causes a **class explosion** (e.g. `MilkCoffee`, `SugarCoffee`, `MilkSugarCoffee`, `MilkSugarWhipCoffee`...).

**Idea:** wrap the object in another object that **adds behavior before/after** delegating to the original. Decorators implement the **same interface** as the wrapped object, so they're stackable — like layers.

**Where used:** Java I/O streams (`BufferedReader(new FileReader(...))`), middleware chains, logging/auth wrappers, UI component styling.

### Code (coffee shop example)

```java
// 1. Base interface
interface Coffee {
    double cost();
    String description();
}

// 2. Concrete base
class SimpleCoffee implements Coffee {
    public double cost() { return 5.0; }
    public String description() { return "Simple coffee"; }
}

// 3. Base decorator — implements same interface + wraps a Coffee
abstract class CoffeeDecorator implements Coffee {
    protected Coffee wrapped;
    public CoffeeDecorator(Coffee wrapped) { this.wrapped = wrapped; }
}

// 4. Concrete decorators — each adds one thing
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee wrapped) { super(wrapped); }
    public double cost() { return wrapped.cost() + 2.0; }
    public String description() { return wrapped.description() + " + milk"; }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee wrapped) { super(wrapped); }
    public double cost() { return wrapped.cost() + 1.0; }
    public String description() { return wrapped.description() + " + sugar"; }
}

class WhipDecorator extends CoffeeDecorator {
    public WhipDecorator(Coffee wrapped) { super(wrapped); }
    public double cost() { return wrapped.cost() + 3.0; }
    public String description() { return wrapped.description() + " + whip"; }
}

// usage — stack like layers
Coffee order = new SimpleCoffee();                    // 5.0
order = new MilkDecorator(order);                     // 7.0
order = new SugarDecorator(order);                    // 8.0
order = new WhipDecorator(order);                     // 11.0
System.out.println(order.description());  // "Simple coffee + milk + sugar + whip"
System.out.println(order.cost());         // 11.0
```

### Why not inheritance?

```
Inheritance → class explosion:
  MilkCoffee, SugarCoffee, MilkSugarCoffee,
  MilkWhipCoffee, SugarWhipCoffee, MilkSugarWhipCoffee...
  3 add-ons = 7 subclasses. 10 add-ons = 1023 subclasses!

Decorator → just 3 classes. Combine freely at runtime.
```

### Java I/O — the classic real-world example

```java
// each layer adds behavior: file reading → buffering → line counting
new LineNumberReader(           // adds line counting
    new BufferedReader(         // adds buffering
        new FileReader("f.txt") // base: reads file
    )
)
```

### SOLID keywords it satisfies
- **OCP** ⭐ — new behavior = new decorator class. Existing classes **never touched**.
- **SRP** — each decorator adds **one responsibility**.
- **DIP** — decorators depend on the `Coffee` interface, not concrete classes.

### Decorator vs Adapter

| | Decorator | Adapter |
|--|-----------|---------|
| **Purpose** | **add behavior** to existing interface | **convert** one interface to another |
| **Interface** | same as wrapped object | different from wrapped object |
| **Stackable** | yes — multiple layers | usually single wrapper |

### Interview answer

"**Decorator** solves the problem of needing to **add behavior at runtime** without modifying the original class. Instead of inheritance (which causes **class explosion** with combinations), I wrap the object in a decorator that implements the **same interface** and adds behavior before/after delegating.

Example: `SimpleCoffee` costs 5. Wrap it in `MilkDecorator` → 7. Wrap that in `SugarDecorator` → 8. **Stackable layers**, any combination, no new subclass needed.

Real-world: Java I/O — `BufferedReader(new FileReader(...))` — each wrapper adds one capability. New behavior? **New decorator class. Existing code untouched** — classic OCP."

---

# BEHAVIORAL

> **Behavioral patterns** focus on **how objects behave and communicate** — which algorithm to use, how they notify each other, how to distribute responsibilities. (Creational = how to make. Structural = how to compose. Behavioral = how they talk / act.)

## 5. Strategy

**Problem:** class has **multiple algorithms** to do the same thing, and the choice happens at **runtime**. Putting them all inside one class with `if/else` breaks **SRP + OCP**.

**Idea:** put each algorithm in its own class implementing a **common interface**. The **context** depends on the interface and swaps algorithms at runtime.

**Where used:** payment methods, pricing tiers (Regular / VIP / Seasonal), sorting (`Comparator`), compression (ZIP/RAR/GZIP), authentication (password / OAuth / biometric), shipping calculators.

### Code (shipping example)
```java
// 1. Strategy interface
interface ShippingStrategy {
    double calculate(Order order);
}

// 2. Concrete strategies — each algorithm isolated
class StandardShipping implements ShippingStrategy {
    public double calculate(Order order) { return order.weight * 2; }
}
class ExpressShipping implements ShippingStrategy {
    public double calculate(Order order) { return order.weight * 5 + 10; }
}
class SameDayShipping implements ShippingStrategy {
    public double calculate(Order order) { return order.weight * 10 + 30; }
}

// 3. Context — uses a strategy without knowing its details
class ShippingCalculator {
    private ShippingStrategy strategy;

    public ShippingCalculator(ShippingStrategy strategy) { this.strategy = strategy; }
    public void setStrategy(ShippingStrategy s) { this.strategy = s; }   // swap at runtime

    public double calculate(Order order) { return strategy.calculate(order); }
}

// usage — pick algorithm at runtime
ShippingCalculator calc = new ShippingCalculator(new StandardShipping());
calc.setStrategy(new ExpressShipping());
```

### SOLID keywords it satisfies
- **OCP** ⭐ — new algorithm = new class. Context never touched. Strategy is **the classic OCP example**.
- **SRP** — each algorithm is one class with one job.
- **DIP** — context depends on the interface, not concrete classes.

### Strategy vs Factory
| Factory | Strategy |
|---------|----------|
| Handles **creation** of the object | Handles **use** of the object |
| Returns object, then done | Object stays and gets used by the context |
| Solves **which class to create** | Solves **which algorithm to use** |

They **complement** each other — a Factory can create the Strategy and inject it into the Context.

### Interview answer
"الـ **Strategy** بيحل مشكلة إن class فيه **كذا algorithm** لعمل نفس الحاجة، والاختيار في الـ **runtime**. بدل ما أعمل `if/else` كبير جوّه الـ class (**كسر SRP + OCP**)، بحط كل algorithm في **class مستقل بيـ implement نفس الـ interface**، والـ context بيعتمد على الـ interface ويقدر يبدّل بينهم في الـ runtime.

مثال: `ShippingCalculator` عنده Standard, Express, SameDay strategies. algorithm جديد؟ **class جديد بس، الـ Calculator ما بيتلمسش**. ده **التطبيق الكلاسيكي لـ Open/Closed**."

---

## 6. Observer

**Problem:** an object (Subject) needs to **notify multiple other objects** when its state changes, without knowing them by name or hardcoding them — because hardcoding breaks **OCP + SRP + DIP**.

**Idea:** Subject keeps a `List<Observer>`. When state changes → loops through and calls `update()` on each one. Observers subscribe/unsubscribe from outside — **zero code changes** in Subject.

**Where used:** event systems, UI listeners (button click → multiple handlers), notifications (YouTube channel → subscribers), message brokers, reactive programming (RxJava).

### Code (YouTube channel example)

```java
// 1. Observer interface
interface Observer {
    void update(String videoTitle);
}

// 2. Concrete observers — each reacts differently
class User implements Observer {
    private String name;
    public User(String name) { this.name = name; }
    public void update(String videoTitle) {
        System.out.println(name + " got notified: " + videoTitle);
    }
}

class EmailService implements Observer {
    public void update(String videoTitle) {
        System.out.println("Sending email about: " + videoTitle);
    }
}

class Analytics implements Observer {
    public void update(String videoTitle) {
        System.out.println("Logging stats for: " + videoTitle);
    }
}

// 3. Subject — manages observer list + notifies
class Channel {
    private String name;
    private List<Observer> subscribers = new ArrayList<>();

    public Channel(String name) { this.name = name; }

    public void subscribe(Observer o)   { subscribers.add(o); }
    public void unsubscribe(Observer o) { subscribers.remove(o); }

    private void notifyAll(String videoTitle) {
        for (Observer o : subscribers) {
            o.update(videoTitle);
        }
    }

    public void uploadVideo(String title) {
        System.out.println(name + " uploaded: " + title);
        notifyAll(title);   // state changed → notify everyone
    }
}

// usage
Channel ch = new Channel("MO");
ch.subscribe(new User("Ahmed"));
ch.subscribe(new User("Sara"));
ch.subscribe(new EmailService());
ch.subscribe(new Analytics());

ch.uploadVideo("Observer Pattern");
// Ahmed got notified, Sara got notified, email sent, stats logged
// Channel doesn't know WHO they are or WHAT they do
```

### Push vs Pull

| | Push | Pull |
|--|------|------|
| **Subject sends** | all data in `update(title, desc, length)` | reference to itself `update(channel)` |
| **Observer gets** | everything — even if not needed | only what it asks for via `ch.getTitle()` |
| **Coupling** | higher | lower |

**In practice:** most implementations are hybrid — push the most important data + pass a reference for pulling the rest.

### SOLID keywords it satisfies
- **OCP** ⭐ — new observer = new class + `subscribe()`. Subject **never touched**.
- **SRP** — Subject manages state + notify. Each observer handles one reaction.
- **DIP** — Subject knows `Observer` interface only, not concrete classes.

### Strategy vs Observer

| | Strategy | Observer |
|--|----------|----------|
| **Solves** | choosing an algorithm from multiple options | notifying multiple objects when something happens |
| **Relationship** | Context ↔ **one** strategy | Subject ↔ **many** observers |
| **At runtime** | swaps algorithm | adds/removes observers |
| **Example** | Shipping calculator | YouTube notifications |

### Interview answer

"**Observer** solves the problem of an object (Subject) needing to **notify multiple other objects** when its state changes, **without knowing them by name**.

Example: a YouTube Channel uploads a video and needs to notify all subscribers — users, email service, analytics. Instead of hardcoding each one inside `uploadVideo()` (**breaks OCP + SRP**), the Channel keeps a **`List<Observer>`** and does a **loop + `update()`** on each one.

New observer? **New class implementing Observer + `subscribe()`**. The Channel is **never touched**."

---
