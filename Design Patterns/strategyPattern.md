# Strategy — Full Study Note

> ### 🥊 The Punch Line
> **"A behavioral pattern where you pull an algorithm out into its own interchangeable object, so you can swap how something is done without touching the code that uses it."**

---

Same company, continuing the story. **Full code at every stage, including the caller.**

> **Where we left off:** Observer. `Channel` announces new videos; `User`s subscribe and react without the channel knowing who they are.
> **New problem:** NotifyCorp now sells premium subscriptions, and checkout needs to **process payment** — which depends on how the customer wants to pay.

---

# The Story: one payment method becomes five

## 📅 Day 1 — "Everyone pays by credit card"

```java
// ============ Checkout.java ============
public class Checkout {
    public void pay(double amount) {
        System.out.println("💳 Charging $" + amount + " to credit card");
    }
}
```

**Verdict:** Fine. One payment method, no variation. **No pattern needed.**

> 🔑 **Lesson 1:** patterns solve *change*. One method is not a problem.

---

## 📅 Day 15 — "Also support PayPal. Also wallet balance." 😰

```java
// ============ Checkout.java ============
public class Checkout {
    public void pay(double amount, String method) {
        if (method.equals("CREDIT_CARD")) {                // ❌ OCP — new method = EDIT this if/else
            System.out.println("💳 Charging $" + amount + " to credit card");
        } else if (method.equals("PAYPAL")) {              // ❌ this chain grows forever
            System.out.println("🅿️ Redirecting $" + amount + " through PayPal");
        } else if (method.equals("WALLET")) {
            System.out.println("👛 Deducting $" + amount + " from wallet balance");
        }
    }
}

// ============ Main.java (THE CALLER) ============
Checkout checkout = new Checkout();
checkout.pay(100.0, "PAYPAL");   // ❌ magic string — typo-prone
```

Three months later, finance wants **bank transfer** and **crypto** support too:

```java
public class Checkout {
    // 🔺 HIGH-LEVEL: "process payment for checkout"
    public void pay(double amount, String method) {
        if (method.equals("CREDIT_CARD")) {                // ❌ OCP — edit this method AGAIN
            chargeCard(amount);
        } else if (method.equals("PAYPAL")) {
            redirectToPaypal(amount);
        } else if (method.equals("WALLET")) {
            deductWallet(amount);
        } else if (method.equals("BANK_TRANSFER")) {
            initiateBankTransfer(amount);
        } else if (method.equals("CRYPTO")) {
            broadcastCryptoTx(amount);
        }
        // ❌ SRP — Checkout now owns FIVE different payment integrations' worth of logic,
        //          each with its own reason to change (Stripe SDK updates, PayPal API
        //          changes, wallet balance rules, bank compliance, crypto node config...)
        // ❌ DIP — Checkout is directly wired to EVERY payment provider's specific API calls.
        //          Testing "does wallet deduction work" means instantiating the WHOLE
        //          Checkout class and threading a magic string through it.
    }
}
```

```
┌───────────────────────────────────────────┐
│  ADD "APPLE PAY" SUPPORT                   │
│  ├── edit pay()                    ✏️      │
│  ├── hope you didn't break CREDIT_CARD ⚠️  │
│  ├── hope you didn't break PAYPAL  ⚠️       │
│  └── one giant method, five providers 💥   │
└───────────────────────────────────────────┘
```

> 🔑 **Lesson 2:** `Checkout` shouldn't need to **know every payment provider's integration details**. It should just say *"process payment using whatever method I was given,"* and let each method own its own logic.

---

## 📅 Day 30 — **Strategy**

The idea: **pull each payment method out into its own class behind a shared interface**, and hand `Checkout` whichever one it should use. `Checkout` calls the interface; it never writes the provider-specific logic itself.

**The mental flip:**

```
BEFORE:  Checkout KNOWS every payment provider's INTEGRATION
         → if/else picks the logic → new provider = edit Checkout

AFTER:   Checkout knows ONE thing: "a PaymentStrategy"
         → calls strategy.pay(amount) → new provider = NEW CLASS, edit nothing
```

### The full code

```java
// ============ PaymentStrategy.java  ← THE STRATEGY interface ============
public interface PaymentStrategy {
    void pay(double amount);
}

// ============ CreditCardPayment.java  ← CONCRETE STRATEGY #1 ============
public class CreditCardPayment implements PaymentStrategy {
    @Override public void pay(double amount) {
        System.out.println("💳 Charging $" + amount + " to credit card");
    }
}

// ============ PaypalPayment.java  ← CONCRETE STRATEGY #2 ============
public class PaypalPayment implements PaymentStrategy {
    @Override public void pay(double amount) {
        System.out.println("🅿️ Redirecting $" + amount + " through PayPal");
    }
}

// ============ WalletPayment.java  ← CONCRETE STRATEGY #3 ============
public class WalletPayment implements PaymentStrategy {
    @Override public void pay(double amount) {
        System.out.println("👛 Deducting $" + amount + " from wallet balance");
    }
}


// ============ Checkout.java  ← 🔺 HIGH-LEVEL — the CONTEXT ============
public class Checkout {
    private PaymentStrategy strategy;   // 🔷 holds the ABSTRACTION, not a provider's SDK

    public Checkout(PaymentStrategy strategy) {   // ← injected, swappable
        this.strategy = strategy;
    }

    public void setStrategy(PaymentStrategy strategy) {   // ⭐ can swap at RUNTIME
        this.strategy = strategy;
    }

    public void pay(double amount) {
        strategy.pay(amount);   // ⭐ delegates — doesn't know WHICH provider this is
    }
}


// ============ Main.java (THE CALLER) ============
public class Main {
    public static void main(String[] args) {
        Checkout checkout = new Checkout(new PaypalPayment());
        checkout.pay(100.0);                          // 🅿️ Redirecting $100.0 through PayPal

        checkout.setStrategy(new CreditCardPayment()); // ⭐ swap at runtime
        checkout.pay(100.0);                          // 💳 Charging $100.0 to credit card
    }
}

// OUTPUT:
// 🅿️ Redirecting $100.0 through PayPal
// 💳 Charging $100.0 to credit card
```

### 🎬 Flow of execution

```
Main:  new Checkout(new PaypalPayment())
         │  strategy = PaypalPayment instance
         ▼
       checkout.pay(100.0)
         │
         ├─ strategy.pay(100.0)
         │      │
         │      │  ⭐ THE MAGIC MOMENT ⭐
         │      │  Checkout calls the INTERFACE method — Java looks at the ACTUAL
         │      │  object → it's a PaypalPayment → runs ITS pay()
         │      ▼
         │   PaypalPayment.pay(100.0) → 🅿️ Redirecting $100.0 through PayPal
         │
         └─ done

       checkout.setStrategy(new CreditCardPayment())
         │  strategy is REPLACED — Checkout itself never changes
       checkout.pay(100.0) → strategy.pay(100.0) → 💳 Charging $100.0 to credit card

  💡 THE WHOLE POINT: Checkout.pay() is IDENTICAL code no matter which provider is
     plugged in — it never contains the word "Paypal" or "CreditCard".
```

### 🎯 The payoff: add Apple Pay. Watch what you DON'T touch.

```java
// ============ ApplePayPayment.java  ← NEW FILE ============
public class ApplePayPayment implements PaymentStrategy {
    @Override public void pay(double amount) {
        System.out.println("🍎 Authorizing $" + amount + " via Apple Pay");
    }
}

// ============ Main.java ============
checkout.setStrategy(new ApplePayPayment());   // one line, at the composition point
```

```
FILES CREATED:  ApplePayPayment.java            ✅ 1 new file
FILES MODIFIED: Checkout.java ............. NONE ✅
```

```java
// ✅ SOLID scorecard
public class Checkout {
    private PaymentStrategy strategy;
    // ✅ OCP — new provider = new class, Checkout.java never changes
    // ✅ DIP — Checkout depends on the 🔷 PaymentStrategy ABSTRACTION,
    //          never on 🔻 PaypalPayment/CreditCardPayment concretes
    // ✅ SRP — Checkout's job is to RUN the checkout process.
    //          Each provider's integration is that strategy's OWN, single responsibility.

    public void pay(double amount) {
        strategy.pay(amount);   // delegates, doesn't integrate
    }
}
```

---

# ⚖️ Advantages & Disadvantages (straight talk)

## ✅ Advantages

```
1. Open/Closed         → new provider = new class, Checkout.java untouched
2. Testable in isolation → test PaypalPayment.pay() alone, no Checkout needed
3. Runtime swapping     → setStrategy() changes provider while the app is running
4. Kills duplicated if/else → the branching logic disappears entirely, not just centralized
```

## ❌ Disadvantages

```
1. More classes         → one class per payment provider, even simple ones
2. Caller must know which strategy to pick → the if/else doesn't vanish, it moves to the composition point
3. Overkill for 2 stable options → a boolean flag or single if/else may be simpler
4. Strategies can't easily share partial logic → common code needs a shared base class or helper
```

---

# 🚧 Common beginner mistakes (simple, memorizable)

**1. Using it for one payment method that never changes.**
If there's only ever one way to pay, Strategy is pure ceremony — just write the method.

**2. Forgetting the caller still has to choose a strategy.**
The if/else doesn't disappear — it moves to `main`/config/DI. Someone still decides `new PaypalPayment()`.

**3. Putting shared logic in every concrete strategy.**
If `CreditCardPayment` and `PaypalPayment` both need fraud-check logic, duplicating it in each class is a smell — extract a shared helper or abstract base.

**4. Confusing Strategy with Factory Method.**
Factory Method decides **which object to create**. Strategy decides **which algorithm an already-built object should run**. Different questions, similar shape.

**5. A strategy interface with too many methods.**
If `PaymentStrategy` grows `pay()`, `refund()`, `getFees()`, `validateCard()`... concrete strategies that don't need all of them signal the interface is doing too much (an ISP smell).

---

# 🔗 Cross Topic

**→ Factory Method (creational family)**
Same shape — an interface with swappable implementations — but different intent. Factory Method answers **"which object do I create?"** Strategy answers **"which algorithm should this already-built object run?"** If you can say "swap what gets built" vs "swap what gets done," you've nailed the distinction interviewers ask for.

**→ Open/Closed + Dependency Inversion (SOLID)**
The exact same win you saw with Observer and Factory Method, now applied to **algorithms**: `Checkout` depends only on the `PaymentStrategy` interface, so a new provider is a new class, not an edited method. "Depend on an interface, not the integration" is the whole principle.

---

# 💬 Interview Kit

**Q: What is the Strategy pattern?**
> A **behavioral** pattern that pulls an algorithm out into its own class behind a shared **interface**, so the algorithm can be **swapped at runtime** without changing the code that uses it. The class that uses the algorithm — the **context** — depends only on the interface, never on a specific implementation.

**Q: What problem does it solve?**
> Without it, a class like `Checkout` ends up with a **growing if/else** picking between payment providers internally, which means every new provider **edits the same method** — an **Open/Closed** violation — and gives that class **many reasons to change**, one per provider, a **Single Responsibility** violation.

**Q: How is behavior swapped at runtime?**
> The context holds a reference to the strategy **interface**, usually injected through the constructor or a setter. Calling `setStrategy()` replaces the object behind that reference, so the very next call runs different behavior — the context's own code never changes.

**Q: Strategy vs Factory Method — same shape, different intent?**
> Both hide something behind an interface with multiple implementations. **Factory Method** decides **which object to create** — a creational concern. **Strategy** decides **which algorithm an already-built object should run** — a behavioral concern. "What do I build?" vs "how do I behave?"

**Q: What are the disadvantages?**
> It adds a **class per algorithm**, even trivial ones. The decision of **which strategy to use still exists** — it just moves to the composition point (`main`, config, or a DI container) instead of living inside the context. And for **two stable options**, a simple flag or one if/else is often clearer than the ceremony of an interface and two classes.

**Q: Give a real-world example.**
> Payment processing (credit card, PayPal, wallet, crypto) behind one `PaymentStrategy`, sorting algorithms behind one `Comparator`, or compression algorithms behind one `CompressionStrategy` — the caller picks an implementation, the calling code never changes regardless of which one is active.

### 🗣️ Say this in an interview

> "Strategy solves the problem of a class hardcoding a **growing if/else between algorithms** — here, payment providers — which forces every new provider to **edit that same method**, an Open/Closed violation, and piles up unrelated integrations in one class. Strategy pulls each provider into its own class behind a shared **interface**; the context holds that interface and **delegates** to it, so swapping behavior is just swapping the object behind the reference, even at **runtime** via a setter. I'd distinguish it from **Factory Method**, which decides what to construct — Strategy decides what an already-built object should **do**. The tradeoff is more classes, and the decision of which strategy to use doesn't disappear, it just **moves to the composition point** — so for two stable, simple options I'd just use a flag instead."

---

# 📌 The whole story in one table

| Stage | Code shape | Add a payment provider | OCP | DIP |
|---|---|---|---|---|
| **Day 1** | one hardcoded method | — | — | — |
| **Day 15** | one if/else in `pay()` | ✏️ edit the same method | ❌ | — |
| **Day 30** | Strategy — context + swappable interface | ✅ new class, 0 edits | ✅ | ✅ |

---

# 📝 Summary

**The roles:**
| Role | In our story |
|---|---|
| **Strategy** (interface) | `PaymentStrategy` — the contract every provider implements |
| **Concrete Strategy** | `CreditCardPayment`, `PaypalPayment`, `WalletPayment`, `ApplePayPayment` |
| **Context** | `Checkout` — holds a strategy, delegates to it, swappable via `setStrategy()` |

**Quick revision triggers:**
- A method has a growing if/else picking between ways of doing the same thing → Strategy candidate.
- You need to change behavior **while the app is running**, not just at startup → Strategy, not Factory Method.
- Only one algorithm, ever → don't; just write the method.
