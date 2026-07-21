# Builder — Full Study Note

> ### 🥊 The Punch Line
> **"A creational pattern that builds a complex object step by step through a separate builder, so the object is only handed over when it's complete and valid."**

---

Same company, continuing the story. **Full code at every stage, including the caller.**

> **Where we left off:** Abstract Factory. `NotificationFactory` produces a matched family per vendor. Mixing vendors became impossible.
> **New problem:** NotifyCorp needs to model the people who receive notifications — the `User`. A user has a first name, last name, email, phone... and this object is about to grow up.

---

# The Story: the User object grows up

## 📅 Day 1 — "A user has a name and age"

```java
// ============ User.java ============
public class User {
    private final String firstName;
    private final String lastName;
    private final int    age;

    public User(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName  = lastName;
        this.age       = age;
    }
}

// ============ Main.java (THE CALLER) ============
User u = new User("Mo", "Ali", 25);
```

**Verdict:** Perfect. Three params, readable, immutable. **Do not add a pattern here.**

> 🔑 **Lesson 1:** patterns solve *change*. Three fields is not a problem.

---

## 📅 Day 15 — "Just a few more fields" → telescoping constructors 😰

Six months of feature requests later: email, phone, city, country, verified flag, premium flag, avatar.

```java
// ============ User.java ============
public class User {
    private final String  firstName, lastName, email, phone, city, country, avatarUrl;
    private final int     age;
    private final boolean isVerified, isPremium;

    // ❌ THE TELESCOPE — each ctor calls the next, growing forever
    public User(String fn, String ln) { this(fn, ln, 0); }
    public User(String fn, String ln, int age) { this(fn, ln, age, null); }
    public User(String fn, String ln, int age, String email) { this(fn, ln, age, email, null); }
    public User(String fn, String ln, int age, String email, String phone) {
        this(fn, ln, age, email, phone, null);
    }
    public User(String fn, String ln, int age, String email, String phone, String city) {
        this(fn, ln, age, email, phone, city, null);
    }
    public User(String fn, String ln, int age, String email, String phone, String city, String country) {
        this(fn, ln, age, email, phone, city, country, false);
    }
    public User(String fn, String ln, int age, String email, String phone, String city,
                String country, boolean verified) {
        this(fn, ln, age, email, phone, city, country, verified, false);
    }
    public User(String fn, String ln, int age, String email, String phone, String city,
                String country, boolean verified, boolean premium) {
        this(fn, ln, age, email, phone, city, country, verified, premium, null);
    }
    // ...and the "real" one at the bottom
    public User(String fn, String ln, int age, String email, String phone, String city,
                String country, boolean verified, boolean premium, String avatar) {
        this.firstName = fn; this.lastName = ln; this.age = age; this.email = email;
        this.phone = phone; this.city = city; this.country = country;
        this.isVerified = verified; this.isPremium = premium; this.avatarUrl = avatar;
    }
    // 9 constructors. 10 fields. Adding ONE optional field = ONE more constructor. 📈
}
```

### 🔴 Now look at the caller. This is where it hurts.

```java
// ============ Main.java (THE CALLER) ============
User u = new User("Mo", "Ali", 25, "mo@corp.com", null, "Cairo", "EG", true, true, null);
//                  ↑     ↑    ↑        ↑          ↑     ↑      ↑    ↑     ↑
//  ❌ what are these? first last age  email      phone city country verified premium?
//  ❌ you CANNOT read this line. You must open User.java to decode it.

// ❌ THE BOOLEAN TRAP — which true is which? Swap them: still compiles, silently wrong.
new User("Mo", "Ali", 25, "mo@corp.com", null, "Cairo", "EG", true, false, null);
new User("Mo", "Ali", 25, "mo@corp.com", null, "Cairo", "EG", false, true, null);
//                                                             ↑ isVerified / isPremium? The compiler CANNOT tell.

// ❌ SAME-TYPE SWAP — firstName and lastName are both String. Swap them → compiles → wrong user.
new User("Ali", "Mo", 25, ...);   // every email now says "Dear Ali Mo" 💥
```

```java
// 🔺 User — a DOMAIN object (what a user IS)
public class User {
    // ❌ SRP — TWO reasons to change: (1) a new field is added, (2) user behaviour changes
    public User(String fn, String ln, int age, String email, String phone, String city,
                String country, boolean verified, boolean premium, String avatar) { ... }
    // ❌ OCP — every new optional field = EDIT this class + ADD another constructor,
    //          and every existing call site using the "full" ctor breaks.
}
```

> 🔑 **Lesson 2:** the telescope's failure isn't verbosity — it's that **the compiler stops protecting you**. Position-based arguments of the same type are unverifiable.

---

## 📅 Day 30 — First fix attempt: JavaBeans / setters

```java
// ============ User.java ============
public class User {
    private String  firstName;    // ❌ `final` had to GO — setters need mutability
    private String  lastName;
    private int     age;
    private boolean isPremium;

    public User() {}              // ← empty ctor: object born EMPTY and INVALID
    public void setFirstName(String f) { this.firstName = f; }
    public void setLastName(String l)  { this.lastName = l; }
    public void setAge(int a)          { this.age = a; }
    public void setPremium(boolean p)  { this.isPremium = p; }
}

// ============ Main.java (THE CALLER) ============
User u = new User();                 // ← ⚠️ VALID object? No. It's EMPTY.
u.setFirstName("Mo");                // ✅ readable! every field is named!
u.setLastName("Ali");
u.setAge(25);
u.setPremium(true);
repo.save(u);
```

**✅ Fixed:** readability. Boolean trap dead. Skipping optional fields trivial.

**🔴 Broken, seriously:**

```java
User u = new User();
u.setFirstName("Mo");
saveAsync(u);                 // 😱 persisted with NO last name, age 0
u.setLastName("Ali");         // too late — half-built row already in the DB

User u2 = buildUser();
auditLog.record(u2);
u2.setEmail("someone-else@corp.com");   // 💥 mutates the object the log holds
```

```java
// 🔺 User — a DOMAIN object
public class User {
    private String firstName;     // ❌ no `final` → mutable → not thread-safe
    public User() {}              // ❌ an object that exists in an INVALID state
    // ❌ SRP still broken — no validation possible, no moment "construction is finished"
}
```

> 🔑 **Lesson 3:** setters bought **readability** by selling **validity and immutability**. Telescoping gave a valid object you couldn't read; JavaBeans gives a readable object that isn't valid. **You want both.**

---

## 📅 Day 45 — Builder

> Collect the fields on a **separate mutable builder**. The real `User` is created **once, at the end**, fully formed, validated, immutable. The half-built state exists **only inside the builder**.

```java
// ============ User.java  ← THE PRODUCT (immutable, no public constructor) ============
public class User {
    private final String  firstName, lastName, email, phone, city;
    private final int     age;
    private final boolean isVerified, isPremium;

    // 🔒 PRIVATE — the ONLY way in is through the Builder. No half-built User can exist.
    private User(Builder b) {
        this.firstName = b.firstName; this.lastName = b.lastName; this.age = b.age;
        this.email = b.email; this.phone = b.phone; this.city = b.city;
        this.isVerified = b.isVerified; this.isPremium = b.isPremium;
    }

    public static class Builder {
        private final String firstName, lastName;      // required
        private int     age        = 0;                 // optional, defaulted
        private String  email      = null;
        private String  phone      = null;
        private String  city       = null;
        private boolean isVerified = false;
        private boolean isPremium  = false;

        // ⭐ REQUIRED fields go in the BUILDER'S constructor — impossible to forget
        public Builder(String firstName, String lastName) {
            this.firstName = firstName; this.lastName = lastName;
        }

        // ⭐ Optional fields: one named method each, each RETURNS `this` → chaining
        public Builder age(int age)        { this.age = age; return this; }
        public Builder email(String email) { this.email = email; return this; }
        public Builder city(String city)   { this.city = city; return this; }
        public Builder verified(boolean v) { this.isVerified = v; return this; }
        public Builder premium(boolean p)  { this.isPremium = p; return this; }

        // ⭐ build() — the ONE moment construction is "finished". Validate HERE.
        public User build() {
            if (firstName == null || firstName.isBlank())
                throw new IllegalArgumentException("First name required");
            if (age < 0 || age > 150)
                throw new IllegalArgumentException("Age must be 0-150");
            if (isPremium && email == null)
                throw new IllegalStateException("Premium users must have an email (for billing)");
            //     ↑ CROSS-FIELD validation — impossible with setters, trivial here
            return new User(this);   // built in ONE shot, fully formed
        }
    }
    // getters only — NO setters. Immutable forever.
}

// ============ Main.java (THE CALLER) ============
User u = new User.Builder("Mo", "Ali")      // required, positional, compiler-enforced
                .age(25)
                .email("mo@corp.com")        // ✅ named — no same-type swap
                .verified(true)              // ✅ named — no boolean trap
                .premium(true)
                .build();                    // ✅ validated HERE, or it throws
repo.save(u);
```

### 🎬 Flow of execution

```
Main:  new User.Builder("Mo", "Ali")
         │  ⭐ REQUIRED fields captured here — the compiler enforces "required"
         ▼
       Builder{firstName="Mo", lastName="Ali", age=0, email=null, isPremium=false, …}
         │
         ├─ .age(25)      → sets field → returns `this`  ← ⭐ THE CHAINING TRICK
         ├─ .email(...)   → sets field → returns `this`
         ├─ .premium(true)→ sets field → returns `this`
         │      ⚠️ NOTE: no User exists yet. Only the builder is mutating.
         ▼
       .build()
         ├─ validate firstName ✓
         ├─ validate age       ✓
         ├─ cross-field check  ✓   ← ONE moment the whole picture is known
         └─ new User(this)   ← ⭐ product born COMPLETE, VALID, IMMUTABLE

  💡 THE WHOLE POINT: there is NO instant a half-built `User` is reachable by anyone.
```

### 🎯 The payoff: add a field. Watch what you DON'T touch.

```java
// Builder: private String country = null; public Builder country(String c) {...; return this;}
// User:    private final String country;   // ... this.country = b.country;
```

```
FILES MODIFIED: User.java (add field + method)   ✏️ 1 file, additive only
CALL SITES BROKEN: ................... NONE       ✅
```

```java
// ✅ SOLID scorecard
public class User {
    private User(Builder b) { ... }
    // ✅ SRP — User's only job is to BE a user. Constructing it is the Builder's job.
    // ✅ OCP — a new optional field is ADDITIVE: no call site breaks
    // ✅ immutable + thread-safe: all fields `final`, no setters
    public static class Builder {
        public User build() { /* validate, then create in ONE shot */ }
        // ✅ single place for validation
    }
}
```

---

## 📅 Day 60 — the GoF Builder (with a Director)

The above is the **fluent builder** (Bloch, *Effective Java*) — what you'll write 95% of the time. GoF's original adds two ideas: a **Builder interface** (same steps build *different* products), and a **Director** (owns the *recipe*/order of steps).

```java
// ============ UserBuilder.java  ← THE BUILDER INTERFACE (the STEPS) ============
public interface UserBuilder {
    void setName(String first, String last);
    void setEmail(String email);
    void setVerified(boolean v);
}

// ============ EntityBuilder.java  ← CONCRETE BUILDER #1 ============
public class EntityBuilder implements UserBuilder {
    private final UserEntity entity = new UserEntity();
    public void setName(String f, String l) { entity.firstName = f; entity.lastName = l; }
    public void setEmail(String email)      { entity.email = email; }
    public void setVerified(boolean v)      { entity.verifiedColumn = v ? 1 : 0; }
    public UserEntity getResult() { return entity; }   // ← returns UserEntity (DB row shape)
}

// ============ JsonBuilder.java  ← CONCRETE BUILDER #2 (different product!) ============
public class JsonBuilder implements UserBuilder {
    private final StringBuilder json = new StringBuilder("{");
    public void setName(String f, String l) { json.append("\"name\":\"").append(f).append(" ").append(l).append("\","); }
    public void setEmail(String email)      { json.append("\"email\":\"").append(email).append("\","); }
    public void setVerified(boolean v)      { json.append("\"verified\":").append(v).append(","); }
    public String getResult() { return json.append("}").toString(); }   // ← returns String, NOT UserEntity!
}

// ============ UserDirector.java  ← THE DIRECTOR (the RECIPE) ============
public class UserDirector {
    public void constructAdmin(UserBuilder builder, String first, String last, String email) {
        builder.setName(first, last);
        builder.setEmail(email);
        builder.setVerified(true);          // admins are always verified
    }
    // 🔺 Director knows the ORDER. It does NOT know UserEntity or the JSON String exist.
}

// ============ Main.java (THE CALLER) ============
UserDirector director = new UserDirector();

EntityBuilder entityBuilder = new EntityBuilder();
director.constructAdmin(entityBuilder, "Mo", "Ali", "mo@corp.com");
UserEntity entity = entityBuilder.getResult();   // ⭐ result comes from the BUILDER, not the director

JsonBuilder jsonBuilder = new JsonBuilder();
director.constructAdmin(jsonBuilder, "Mo", "Ali", "mo@corp.com");
String json = jsonBuilder.getResult();           // ⭐ a DIFFERENT TYPE entirely
```

**Why doesn't the Director return the product?** It works with builders **only via their common interface**, so it's **unaware of concrete builders and products** — `UserEntity` and the JSON `String` share no common type it could return. You ask the builder you handed it.

**The quotable fact:** unlike Factory Method or Abstract Factory, **Builder doesn't require the products to have a common interface** — that's exactly what lets one construction process yield completely different products.

---

# 🆚 Builder vs Named Arguments

```python
# Python — named arguments + defaults, no pattern at all
User(first_name="Mo", last_name="Ali", age=25, email="mo@corp.com", premium=True)
#   named ✅  optional ✅  any order ✅  @dataclass(frozen=True) → immutable ✅
```

For **one object, one call, all values known** — named args win outright; the fluent Builder is largely **a workaround for Java's missing named/default parameters**. Builder still earns its place when:
- construction is **incremental/conditional** (fields added across code paths, sealed at `build()`)
- the **same steps must produce different product types** (the GoF form)
- a **Director** reuses a recipe across builders
- you want a **staged fluent DSL** (query builders)

> **Rule of thumb:** one call, all values known, one product → **named args**. Built up over time, many products, or a DSL → **Builder**.

---

# ⚖️ Advantages & Disadvantages (straight talk)

## ✅ Advantages

```
1. Readable calls    → .email(x).premium(true) instead of positional true/true/null
2. Never half-built  → the messy state lives INSIDE the builder, discarded after build()
3. Immutable product → all fields final, no setters, thread-safe with zero locking
4. One validation gate → build() is the only moment cross-field rules can be checked
5. Additive growth   → a new field is a new builder method, zero call sites break
```

## ❌ Disadvantages

```
1. Boilerplate        → every field is written twice (User + Builder) plus a method
2. More classes       → especially the GoF form: interface + N builders + director
3. Runtime enforcement→ if required fields are chained instead of in the ctor, errors move to runtime
4. Overkill for small objects → 3-4 fields, mostly required? a plain constructor is clearer
5. Builder is mutable → not thread-safe itself; build per-thread, don't share a builder
```

---

# 🚧 Common beginner mistakes (simple, memorizable)

**1. Using it for 3 fields.**
A plain constructor was enough — this is ceremony.

**2. Reaching for it in Python/JS.**
Use keyword args + a frozen dataclass instead; Builder is a Java workaround, not a universal rule.

**3. Required fields as chained methods instead of the constructor.**
`new User.Builder().build()` compiles fine and throws at **runtime** — the compile-time guarantee is gone.

**4. Skipping validation in `build()`.**
If `build()` just copies fields, you bought readability and threw away the main prize.

**5. Leaving a public constructor or setter on the product.**
Either one is a back door around the builder — half-built or mutable objects become possible again.

**6. Making the Director return the product.**
It can't — it only knows the shared builder interface, not the concrete product type.

**7. Reusing one builder for two products without resetting.**
The second product silently inherits leftover state from the first `build()` call.

---

# 🔗 Cross Topic

**→ Factory Method / Abstract Factory**
All three are creational and hide `new`, but each answers a different question: Factory Method — *which class?* Abstract Factory — *which family?* Builder — *how do I assemble THIS ONE object, step by step?* And uniquely, Builder doesn't need its products to share an interface.

**→ Multithreading (your module)**
Builder's immutable product is thread-safe with **zero locking** — the JavaBeans alternative is mutable shared state, the exact race conditions you already studied. Immutability is the cheapest concurrency strategy there is, and Builder is how you get it when the constructor would be unbearable.

---

# 💬 Interview Kit

**Q: What is the Builder pattern?**
> A **creational** pattern that constructs **complex objects step by step**, separating **construction from representation**. Fields are collected on a **builder** and the product is produced in one shot at **`build()`**, so it's **immutable** and never observable half-built. Unusually for a creational pattern, **Builder doesn't require a common product interface**, which lets the same steps yield different products.

**Q: What problem does it solve — the telescoping constructor problem?**
> When a class has many mostly-optional fields, Java forces a chain of **telescoping constructors**. Call sites become a wall of positional values — unreadable, and dangerous because two adjacent `boolean`s or `String`s can be **swapped and still compile**, turning into a **runtime** bug instead of a compile error.

**Q: Why not just use setters?**
> Setters fix readability but break three things: the object is **half-built and invalid** between `new` and the last setter, **required fields aren't enforced**, and losing `final` means it's no longer **immutable or thread-safe**. Builder keeps the mutation **inside the builder** and releases a fully-formed, validated, immutable product.

**Q: Where do you validate, and why does that matter?**
> In **`build()`** — the single moment construction is finished and the whole object is known. That's the only place **cross-field rules** are expressible, like "premium users must have an email." Setters have no such moment, so cross-field validation is impossible with them.

**Q: What is the Director, and why can't it return the product?**
> The **Director** owns the **order of construction** as a reusable recipe. It works with builders **only through their shared interface**, so it's unaware of concrete builders and products — since two builders can return **completely unrelated types**, there's no type it could return. You fetch the result **from the builder itself**. The Director is also **optional** — Builder works fine without one.

**Q: Builder vs named arguments — when is Builder actually worth it?**
> For one object built in one call with all values known, **named args and defaults win** — the fluent Builder mostly compensates for languages without them, like Java. Builder earns its place for **incremental or conditional construction**, when the **same steps must produce different product types**, or for a **staged fluent DSL** like a query builder.

**Q: What are the disadvantages?**
> **Boilerplate** — every field is declared twice plus a method. **More classes**, especially the GoF form. It's **over-engineering** for small objects or in languages with named arguments. And the compile-time required-field guarantee **only exists if you design it in** — putting required fields in chained methods instead of the constructor pushes the error to runtime.

### 🗣️ Say this in an interview

> "Builder solves the telescoping constructor problem — a class with many optional fields forces either an unreadable wall of positional constructor calls, or setters that leave the object half-built and mutable. Builder puts required fields in the **builder's constructor** so the compiler enforces them, optional fields as **named chained methods**, and validates everything — including cross-field rules — at **`build()`**, the one moment the whole object is known. The product itself has a **private constructor** and no setters, so it's **immutable** and never observable in an invalid state. I'd also flag that in Python or JS, **keyword arguments and a frozen dataclass** give me the same guarantees for free — Builder is largely a Java workaround, and I'd only reach for it there for incremental construction or a query-builder-style DSL."

---

# 📌 The whole story in one table

| Stage | Call site | Readable? | Always valid? | Immutable? |
|---|---|---|---|---|
| **Day 1** — 3-param ctor | `new User("Mo","Ali",25)` | ✅ | ✅ | ✅ |
| **Day 15** — telescoping | `new User("Mo","Ali",25,"e",null,"Cairo","EG",true,true,null)` | ❌ | ✅ | ✅ |
| **Day 30** — JavaBeans setters | `new User(); u.setX(); u.setY();` | ✅ | ❌ half-built window | ❌ |
| **Day 45** — Fluent Builder | `new Builder("Mo","Ali").age(25).build()` | ✅ | ✅ validated in build() | ✅ |
| **Day 60** — GoF + Director | `director.constructAdmin(builder); builder.getResult()` | ✅ | ✅ | ✅ |

---

# 📝 Summary

**The roles:**
| Role | In our story |
|---|---|
| **Builder** (interface) | `UserBuilder` — declares the construction steps |
| **Concrete Builder** | `EntityBuilder`, `JsonBuilder` |
| **Director** *(optional)* | `UserDirector` — owns the order, unaware of concrete products |
| **Product** | `UserEntity`, JSON `String` — need no common interface |
| *(Fluent form)* | just **Product** + a **static nested Builder** whose methods return `this` |

**Quick revision triggers:**
- `new Thing("Mo","Ali",null,null,0,true,false,null)` → telescoping → Builder.
- Two adjacent `boolean`/`String` params → swap compiles silently → Builder.
- You're in Python/JS and it's one call → keyword args + frozen dataclass, not Builder.
