# SOLID Principles — Notes

5 rules to write code that's easy to change, test, and extend without breaking things.

| Letter | Principle | One line |
|--------|-----------|----------|
| **S** | Single Responsibility | One class = one job |
| **O** | Open/Closed | Extend, don't edit |
| **L** | Liskov Substitution | Child must replace parent safely |
| **I** | Interface Segregation | Small interfaces, not fat ones |
| **D** | Dependency Inversion | Depend on interfaces, not concrete classes |

---

## Interview one-liners

- **S — Single Responsibility:** "الـ class يكون ليه **only one reason to change**، يعني ميكونش فيه أكتر من functionality. لو عدّلت في واحدة، بنسبة كبيرة الباقي هيتكسر وهبقى عرضة لـ **bugs**."

- **O — Open/Closed:** "الـ class يكون **open for extension, closed for modification**. لو عندي service ليها أكتر من behaviour، لما أحتاج أضيف behaviour جديد مروحش أعدّل في نفس الـ class — أضيف **class جديد** بالـ behaviour الجديد. مثال: عندي shapes وكل واحد ليه حسبة area مختلفة، مخليهمش كلهم في class واحد وأفضل أعمل if/else كتير — أعمل **Shape** فيه **abstract method area** وكل shape يعملها **implement** بطريقته."

- **L — Liskov Substitution:** "الـ **child class يقدر يحلّ محل الـ parent** من غير ما يكسر أي حاجة — زي إنه ميـ raise-ش **exception** أو يغيّر السلوك المتوقّع. لو أقدر أبعت الـ child مكان الـ parent في أي مكان ومفيش حاجة تتكسر، يبقى Liskov محترم."

- **I — Interface Segregation:** "بدل ما أعمل **one fat interface** وأجبر الـ classes تعمل implement لـ methods هي مش مهتمة بيها، أعمل أكتر من **interface صغيّر** يحتوي بس على الـ **related methods**. ودا كمان بيخليني مكسرش الـ Liskov وإن الـ child يطلّع **unexpected behavior** مع method معيّنة."

- **D — Dependency Inversion:** "الـ class يعتمد على **interface not concrete class**، عشان لو احتجت أعدّل في الـ tool اللي بتخدم service معيّنة مروحش أعدّل في الـ dependency كل شوية. مثال: عندي payment methods زي **PayPal, Credit, Debit**، والـ **OrderPlacementService** مجيّش جوّاها أستخدم payment method معيّن — أخليها تعتمد على الـ **PaymentService interface**، وأيًا كان الـ payment method اللي هديها ليها تتعامل عادي."

---

## S — Single Responsibility Principle (SRP)

**Idea:** A class should have **one reason to change** — it does one job only.

**Why:** If a class does 3 things, changing one thing can break the other two. Hard to test, hard to maintain.

```java
// BAD — User class does 3 different jobs
class User {
    String name;
    void saveToDatabase() { /* SQL code */ }   // database job
    void sendEmail()      { /* SMTP code */ }   // email job
}
// Change email provider → edit User. Change database → edit User. Fragile.

// GOOD — split each job into its own class
class User { String name; }                          // holds data only
class UserRepository { void save(User u) { } }       // database only
class EmailService   { void send(User u) { } }       // email only
```

**Interview answer (English):** "Each class should have only one responsibility, one reason to change. I separate data, persistence in db, and email service into different classes so a change in one doesn't affect the others."

---

## O — Open/Closed Principle (OCP)

**Idea:** Code should be **open for extension, closed for modification** — add new behavior by adding new code, not by editing old (working) code.

**Why:** Editing tested code risks breaking it. Adding a new class is safer.

```java
// BAD — every new shape forces you to edit this method
class AreaCalculator {
    double area(Object shape) {
        if (shape instanceof Circle) return /* ... */;
        else if (shape instanceof Square) return /* ... */;
        // Add Triangle? edit here AGAIN. breaks Open/Closed.
    }
}

// GOOD — each shape knows its own area; calculator never changes
interface Shape { double area(); }
class Circle implements Shape { public double area() { return Math.PI * r * r; } }
class Square implements Shape { public double area() { return s * s; } }
// New shape = new class. Old code stays untouched.
```

**Interview answer:** "New requirements should mean writing a new class that implements an interface, not modifying existing classes. This prevents breaking code that already works."

---

## L — Liskov Substitution Principle (LSP)

**Idea:** A **child class must be usable anywhere its parent is used** — without breaking behavior or surprising anyone.

**Why:** If a subclass breaks the parent's promise, polymorphism becomes a lie and code that trusts the parent crashes.

```java
// BAD #1 — Square breaks Rectangle's contract
class Rectangle { void setWidth(int w){} void setHeight(int h){} }
class Square extends Rectangle {
    // Square forces width == height, so setWidth(5) secretly changes height too.
    // Code expecting normal Rectangle behavior breaks.
}

// BAD #2 — Ostrich is a Bird but can't fly
class Bird { void fly() {} }
class Ostrich extends Bird { void fly() { throw new RuntimeException(); } } // violates LSP

// GOOD — don't force a wrong hierarchy; split by real capability
interface Bird {}
interface FlyingBird extends Bird { void fly(); }
```

**Interview answer:** "A subtype must honor the contract of its base type. The classic violations are Square extending Rectangle, and Ostrich extending a Bird that can fly — the child breaks assumptions the parent guarantees. Rule of thumb: if you can pass the child anywhere the parent is used and nothing breaks, LSP holds."

---

## I — Interface Segregation Principle (ISP)

**Idea:** **Many small, focused interfaces are better than one big fat interface.** Don't force a class to implement methods it doesn't use.

**Why:** A fat interface forces empty/dummy methods, which is confusing and error-prone.

```java
// BAD — one fat interface forces Robot to implement eat()
interface Worker { void work(); void eat(); }
class Robot implements Worker {
    public void work() { }
    public void eat()  { }  // robot doesn't eat → forced empty/throw. bad.
}

// GOOD — split into small interfaces; implement only what you need
interface Workable { void work(); }
interface Eatable  { void eat(); }
class Robot implements Workable { }               // only work
class Human implements Workable, Eatable { }      // both
```

**Interview answer:** "Split large interfaces into smaller, role-specific ones so no class is forced to implement methods it doesn't need."

---

## D — Dependency Inversion Principle (DIP)

**Idea:** A class should depend on an **interface**, not on one specific class. Don't lock your class to a single exact tool — let it work with any tool that follows the same interface.

**Why:** Hardcoding a concrete class makes swapping or testing painful. Depend on an interface and inject the real implementation.

```java
// BAD — UserService is locked to MySQL
class UserService {
    MySQLDatabase db = new MySQLDatabase();  // hardcoded. switching DB = editing this class.
}

// GOOD — depend on an interface, inject the implementation
interface Database { void save(); }
class MySQLDatabase    implements Database { public void save() { } }
class PostgresDatabase implements Database { public void save() { } }

class UserService {
    private Database db;
    UserService(Database db) { this.db = db; }  // injected from outside (Dependency Injection)
}
// Swap DB = pass a different implementation. UserService never changes.
// Bonus: easy to test — inject a fake/mock Database.
```

**Interview answer:** "High-level modules should depend on abstractions, not concrete implementations. I inject dependencies through interfaces, which lets me swap implementations and mock them in tests. This is what Dependency Injection is built on."

---

## Quick recap

| Principle | Remember it as |
|-----------|----------------|
| **S** | One class, one job |
| **O** | Add new code, don't touch old code |
| **L** | Child works safely in place of parent |
| **I** | Small interfaces, take only what you need |
| **D** | Depend on interface, inject the real thing |

**Link to patterns (say this in interview):**
- Strategy / Factory patterns → follow **Open/Closed**.
- Dependency Injection → implements **Dependency Inversion**.

**Self-test:**
1. Which principle does Dependency Injection implement? → *D (Dependency Inversion)*
2. Square-extends-Rectangle breaks which? → *L (Liskov)*
3. A class that saves to DB *and* sends emails breaks which? → *S (Single Responsibility)*
