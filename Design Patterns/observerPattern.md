# Observer — Full Study Note

> ### 🥊 The Punch Line
> **"A behavioral pattern where an object (the subject) keeps a list of subscribers and notifies them all automatically when its state changes — so the subject never needs to know who's listening."**

---

Same company, continuing the story. **Full code at every stage, including the caller.**

> **Where we left off:** Builder. `User` and message payloads are now built cleanly, immutable, and valid.
> **New problem:** NotifyCorp is launching a video platform. When a **channel uploads a new video**, every subscriber needs to be notified. That subscriber list keeps growing.

---

# The Story: one channel, growing subscribers

## 📅 Day 1 — "When MO's channel uploads, notify Ahmed"

```java
// ============ Video.java ============
public class Video {
    private final String title;
    public Video(String title) { this.title = title; }
    public String getTitle() { return title; }
}

// ============ Channel.java ============
public class Channel {
    private String channelName;
    private User ahmed;                      // ❌ hardcoded — one specific user

    public Channel(String name, User ahmed) {
        this.channelName = name;
        this.ahmed = ahmed;
    }

    public void uploadVideo(Video video) {
        System.out.println(channelName + " uploaded: " + video.getTitle());
        ahmed.onNewVideo(video);       // one direct call
    }
}
```

**Verdict:** Fine. One subscriber, one call. **No pattern needed.**

> 🔑 **Lesson 1:** patterns solve *change*. One listener is not a problem.

---

## 📅 Day 15 — "Also notify Sara. Also Omar. Also everyone who subscribes." 😰

```java
// ============ Channel.java ============
public class Channel {
    private String channelName;
    private User ahmed;      // ❌
    private User sara;       // ❌ every new subscriber = a new field
    private User omar;       // ❌ every new subscriber = a new constructor param

    public Channel(String name, User ahmed, User sara, User omar) {
        this.channelName = name;
        this.ahmed = ahmed; this.sara = sara; this.omar = omar;
    }

    public void uploadVideo(Video video) {
        System.out.println(channelName + " uploaded: " + video.getTitle());
        ahmed.onNewVideo(video);   // ❌ OCP — edit this method
        sara.onNewVideo(video);    // ❌ every new subscriber
        omar.onNewVideo(video);    // ❌ means editing THIS class
    }
}
```

### 🔴 SOLID breakage

```java
// 🔺 Channel — HIGH-LEVEL: "I upload videos"
public class Channel {
    // ❌ SRP — Channel now has as many reasons to change as it has subscribers
    private User ahmed;   // 🔻 a concrete, specific low-level object
    private User sara;    // 🔻
    private User omar;    // 🔻
    // ❌ DIP — Channel (HIGH-level: "publish content") is WELDED to
    //          SPECIFIC low-level users. It must know each one exists by name.

    public void uploadVideo(Video video) {
        ahmed.onNewVideo(video);
        sara.onNewVideo(video);
        omar.onNewVideo(video);
        // ❌ OCP — 10,000th subscriber signs up?
        //          OPEN this file. ADD a field. ADD a constructor param.
        //          ADD a line here. This literally cannot scale.
    }
}
```

```
┌───────────────────────────────────────────┐
│  10,000 SUBSCRIBERS SIGN UP                │
│  ├── edit Channel constructor  ✏️          │
│  ├── add 10,000 fields          ✏️         │
│  ├── add 10,000 lines           ✏️         │
│  └── ...this is obviously insane   💥      │
└───────────────────────────────────────────┘
```

> 🔑 **Lesson 2:** `Channel` shouldn't need to **know each subscriber by name**. It should just **announce "new video"**, and let whoever's subscribed react.

---

## 📅 Day 30 — **Observer**

The idea: **flip who depends on whom.** Instead of `Channel` calling every subscriber by name, users **subscribe themselves**, and `Channel` just says *"new video"* to whoever's on the list — without knowing who that is or how many there are.

**The mental flip:**

```
BEFORE:  Channel KNOWS Ahmed + Sara + Omar by name
         → calls each one BY NAME → new subscriber = edit Channel

AFTER:   Channel knows ONE thing: "a list of Observers"
         → loops the list, calls the SAME method on each → new subscriber = SUBSCRIBE, edit nothing
```

This is exactly YouTube: **you subscribe to a channel** — the channel never subscribes to you. The subscriber list lives on the **Channel** (the Subject), because the Channel is the one announcing.

### The full code

```java
// ============ Observer.java  ← THE OBSERVER interface ============
public interface Observer {
    void onNewVideo(Video video);
}

// ============ User.java  ← CONCRETE OBSERVER ============
public class User implements Observer {
    private String name;
    public User(String name) { this.name = name; }

    @Override
    public void onNewVideo(Video video) {
        System.out.println("📩 Notifying " + name + ": new video \"" + video.getTitle() + "\"");
    }
}


// ============ Channel.java  ← 🔺 THE SUBJECT ============
public class Channel {
    private String channelName;
    private final List<Observer> subscribers = new ArrayList<>();   // ⭐ the subscriber list

    public Channel(String channelName) { this.channelName = channelName; }

    // ⭐ subscribe / unsubscribe — the ONLY thing the subject exposes to observers
    public void subscribe(Observer user)   { subscribers.add(user); }     // USER subscribes TO channel
    public void unsubscribe(Observer user) { subscribers.remove(user); }

    public void uploadVideo(Video video) {
        System.out.println(channelName + " uploaded: " + video.getTitle());
        notifySubscribers(video);        // ⭐ announce — doesn't know WHO's listening
    }

    private void notifySubscribers(Video video) {
        for (Observer o : subscribers) {   // loop the list, call the SAME method on each
            o.onNewVideo(video);
        }
    }
}


// ============ Main.java (THE CALLER) ============
public class Main {
    public static void main(String[] args) {
        Channel channel = new Channel("MO's Tech Channel");

        User ahmed = new User("Ahmed");
        User sara  = new User("Sara");
        User omar  = new User("Omar");

        channel.subscribe(ahmed);     // ✅ registering, not editing Channel
        channel.subscribe(sara);
        channel.subscribe(omar);

        channel.uploadVideo(new Video("Observer Pattern Explained"));
    }
}

// OUTPUT:
// MO's Tech Channel uploaded: Observer Pattern Explained
// 📩 Notifying Ahmed: new video "Observer Pattern Explained"
// 📩 Notifying Sara: new video "Observer Pattern Explained"
// 📩 Notifying Omar: new video "Observer Pattern Explained"
```

### 🎬 Flow of execution — trace it slowly

```
Main:  channel.subscribe(ahmed)
         │  subscribers = [Ahmed]
       channel.subscribe(sara)
         │  subscribers = [Ahmed, Sara]
       channel.subscribe(omar)
         │  subscribers = [Ahmed, Sara, Omar]
         ▼
       channel.uploadVideo(video)
         │
         ├─ print "uploaded: ..."
         ├─ notifySubscribers(video)
         │      │
         │      │  ⭐ THE MAGIC MOMENT ⭐
         │      │  Channel loops its list and calls onNewVideo() on EACH —
         │      │  it never types the word "Ahmed" or "Sara" or "Omar".
         │      ▼
         │   subscribers[0].onNewVideo(video) → 📩 Ahmed...
         │   subscribers[1].onNewVideo(video) → 📩 Sara...
         │   subscribers[2].onNewVideo(video) → 📩 Omar...
         │
         └─ done

  💡 THE WHOLE POINT: Channel.uploadVideo() is IDENTICAL whether 0, 3, or 10,000 subscribers exist.
```

### 🎯 The payoff: 10,000th subscriber signs up. Watch what you DON'T touch.

```java
// ============ Main.java ============
channel.subscribe(new User("Fatima"));   // ← one line, at the composition point
```

```
FILES CREATED:  none — just a new User instance   ✅
FILES MODIFIED: Channel.java .................. NONE ✅
```

**That's Open/Closed made real again** — same shape you already proved with Factory Method, now on the *behavioral* side: extend by **subscribing**, not by editing the subject.

### ✅ SOLID scorecard

```java
// 🔺 Channel — THE SUBJECT
public class Channel {
    private final List<Observer> subscribers = new ArrayList<>();
    // ✅ OCP — new subscriber = subscribe(), Channel.java never changes
    // ✅ DIP — Channel depends on the 🔷 Observer ABSTRACTION,
    //          never on 🔻 specific User objects by name
    // ✅ SRP — Channel's job is to publish videos + announce.
    //          Reacting to that (sending an email, a push, whatever) is each User's OWN job.

    public void uploadVideo(Video video) {
        System.out.println(channelName + " uploaded: " + video.getTitle());
        notifySubscribers(video);   // announces, doesn't orchestrate reactions
    }
}
```

---

# 🔬 Push vs Pull — the two ways to notify

**Push model** (what we just built): the subject sends the **full data** with the event.
```java
void onNewVideo(Video video);   // observer gets the whole Video, whether it needs it or not
```
✅ Simple, one call. ❌ Couples the observer interface to the subject's full shape; wasteful if some observers only need the title.

**Pull model:** the subject sends **just a signal**; the observer **pulls** what it needs.
```java
void update(Channel channel);       // observer calls channel.getLatestVideo() itself
```
✅ Observer takes only what it needs, subject can change internals more freely. ❌ Observer needs a reference back to the subject, one extra hop.

**Interview one-liner:** *"Push is simpler and faster for small payloads; pull scales better when different observers need different slices of state."*

### 🤔 Does the Observer need to know the Subject?

In **pure push** (what we built), no — `User` never holds a `Channel` reference; everything arrives as a parameter. But two situations force the Observer to **keep a reference to the Subject**, usually passed into its constructor:

```java
public class User implements Observer {
    private String name;
    private Channel channel;   // ⭐ held ONLY because it's needed below

    public User(String name, Channel channel) {
        this.name = name;
        this.channel = channel;
    }

    @Override
    public void onNewVideo(Video video) {
        // 1️⃣ PULL — ask the subject for more than the event gave you
        String subCount = channel.getSubscriberCount();
        System.out.println(name + " sees " + subCount + " total subs");

        // 2️⃣ SELF-UNSUBSCRIBE — the observer decides to stop listening
        if (video.getTitle().contains("Farewell")) {
            channel.unsubscribe(this);   // needs the reference to remove itself
        }
    }
}
```

**In short:** pure push = no reference needed, data comes in as a parameter. Reference to the Subject only shows up when the Observer needs to **pull extra state** or **unsubscribe itself** — and even then, it's passed in explicitly (constructor or the callback itself), not something the pattern requires by default.

> 🔑 **The general rule:** **no reference needed if you only react using data passed in as a parameter (pure push).** A reference is needed **the moment you call any method on the Subject — get or set, doesn't matter.** Reading extra state, mutating it, or unsubscribing are all just different reasons to call a method — the need for the reference is the same either way.

---

# 🔗 Is this Pub/Sub?

**Yes, at the smallest scale — Observer *is* pub/sub with the broker removed.** `Channel` is the publisher, `User`s are subscribers, `uploadVideo()` is publishing an event.

| | **Observer (this pattern)** | **Pub/Sub (Kafka/RabbitMQ)** |
|---|---|---|
| Who talks to whom | `Channel` holds `List<Observer>` and calls them **directly** | A **broker** sits between publisher and subscribers |
| Coupling | loosely coupled via interface, but **same process** | **fully decoupled** — publisher doesn't know subscribers exist |
| Timing | usually **synchronous** — `uploadVideo()` blocks until all notified | usually **asynchronous** — publish and move on |

**One line:** *Observer is pub/sub zoomed in to two objects in one program; pub/sub is Observer zoomed out to a distributed system with a broker in the middle.*

---

# ⚖️ Advantages & Disadvantages (straight talk)

## ✅ Advantages

```
1. Open/Closed        → new subscriber = one subscribe() call, Channel.java untouched
2. Loose coupling     → Channel only knows the Observer interface, never a user's internals
3. Runtime flexible   → subscribers can be added/removed while the app is running
4. Broadcast for free → Channel loops the list once, no manual fan-out code anywhere
```

## ❌ Disadvantages

```
1. Order not guaranteed → nothing enforces observer #2 running before #3
2. Cascading updates    → an observer triggering more events causes a hard-to-trace chain
3. Memory leak          → subscribe() with no unsubscribe() keeps the observer alive forever
4. Harder to trace      → "who's listening?" isn't visible — must grep every subscribe() call
5. One bad observer     → a naive loop stops on the first throw, skipping everyone after it
```

---

# 🚧 Common beginner mistakes (simple, memorizable)

**1. Forget to unsubscribe → memory leak.**
Subscribe without ever unsubscribing = the channel keeps that user alive forever, even after it should be gone.

**2. Observer edits the list while it's being looped → crash.**
A user's `onNewVideo()` calling `channel.subscribe(this)` while the channel is mid-loop blows up.

**3. Assume subscribers get notified in a fixed order.**
Nothing guarantees it unless you build that in yourself.

**4. One giant method for every kind of event.**
`update(String eventType, Object data)` forces every observer to `if/else` on type — split into specific methods instead (`onNewVideo`, `sendCancellation`, etc.).

**5. One bad subscriber blocks everyone after it.**
A plain loop stops at the first exception — decide up front: skip and continue, or fail the whole batch.

**6. Confusing Observer with Pub/Sub.**
Observer = same process, direct call, usually sync. Pub/Sub = broker in the middle, decoupled, usually async. Not the same thing — see the section above.

---

# 🔗 Cross Topic

**→ Strategy (next pattern)**
Both are **behavioral** and both hide something behind an interface — but the question each one answers is different. Strategy is about **how one thing does its job** ("which algorithm should I run right now?"). Observer is about **who else needs to know** something happened ("who's listening for this event?"). Same shape, opposite direction: Strategy picks behavior *for itself*; Observer broadcasts *outward* to others.

**→ SOLID (Open/Closed + Dependency Inversion)**
This is the same win you already saw with Factory Method, just applied to *behavior* instead of *creation*. Before Observer, `Channel` had to know every subscriber **by name** — adding one meant editing `Channel`'s code. After Observer, `Channel` only knows the **`Observer` interface**, so a new subscriber is just a `subscribe()` call, zero edits. "Depend on an interface, not a concrete list of people" is the whole principle in one sentence.

---

# 💬 Interview Kit

**Q: What is the Observer pattern?**
> A **behavioral** pattern where a **subject** maintains a list of **observers** and **notifies them automatically** when its state changes, without knowing their concrete types. It exists for **loose coupling** — the subject depends only on an **interface**, never on specific listeners by name.

**Q: What problem does it solve?**
> Without it, the subject must **call every interested party by name**, so adding a new listener means **editing the subject's code** — an **Open/Closed** and **Dependency Inversion** violation. Observer **inverts** that dependency: observers **subscribe** to the subject, and the subject only depends on the observer **interface**.

**Q: Push vs Pull — what's the difference?**
> In the **push** model the subject sends the **full event data** with the callback — simple, one call. In the **pull** model the subject sends a bare signal and the observer **pulls** whatever it needs by calling back into the subject — more flexible, but couples the observer to the subject's shape.

**Q: What are the disadvantages?**
> **Notification order isn't guaranteed** unless you enforce it. An observer that triggers more events can cause **cascading updates** that are hard to trace. A forgotten **unsubscribe()** is a classic **memory leak**, since the subject keeps the observer alive forever. And a **throwing observer** can silently block everyone after it in a naive loop.

**Q: Observer vs Pub/Sub — aren't they the same?**
> Observer usually runs **in-process and synchronously**, with subject and observer sharing a **common interface**. Pub/Sub (Kafka, RabbitMQ) adds a **broker** between them, fully decoupling publisher from subscriber — delivery is typically **asynchronous** and often cross-process. Pub/Sub is Observer's distributed evolution.

**Q: Observer vs Strategy — same shape, different intent.**
> **Strategy** is about the subject choosing **how it does its own job** — one algorithm, swappable at runtime. **Observer** is about the subject **broadcasting outward** to independent listeners it doesn't use itself. "How do I do this?" vs "who else needs to know this happened?"

**Q: How do you avoid the memory leak, and keep it thread-safe?**
> Always pair **subscribe()** with **unsubscribe()** — some ecosystems use **weak references** so a forgotten observer can still be garbage-collected. For concurrency, use a **thread-safe collection** (like `CopyOnWriteArrayList`) for the subscriber list, since subscribing/unsubscribing while a notify loop is running is a classic **race condition**.

### 🗣️ Say this in an interview

> "Observer solves the problem of a subject having to **know every listener by name**. Without it, adding a new listener means **editing the subject's code** — an **OCP** and **DIP** violation. Observer **inverts** that: listeners **subscribe** through a shared interface, and the subject just **announces** — extend by subscribing, not editing.
>
> There's a **push vs pull** choice for what the callback carries, and real tradeoffs: **notification order isn't guaranteed**, a triggering observer can cause **cascading updates**, and a forgotten **unsubscribe()** is a classic **memory leak**. I'd also distinguish it from **Pub/Sub** — Observer is usually in-process and synchronous; Pub/Sub adds a broker and is typically async and cross-process."

---

# 📌 The whole story in one table

| Stage | Code shape | Add a subscriber | OCP | DIP | Verdict |
|---|---|---|---|---|---|
| **Day 1** | one direct call | — | — | ✅ | fine for ONE listener |
| **Day 15** | Channel calls 3 users directly | ✏️ edit constructor + every call | ❌ | ❌ | doesn't scale to 10,000 |
| **Day 30** | Observer — subject + subscriber list | ✅ subscribe(), 0 edits | ✅ | ✅ | scales to N subscribers |

---

# 📝 Summary

**The roles:**
| Role | In our story |
|---|---|
| **Subject** (Observable) | `Channel` — holds the subscriber list, fires notifications |
| **Observer** (interface) | `Observer` — the contract every subscriber implements |
| **Concrete Observer** | `User` (Ahmed, Sara, Omar, …) |

**Quick revision triggers:**
- A class calls a growing list of specific objects by name → Observer candidate.
- "Who's listening for this?" isn't grep-able → expected cost of Observer, not a bug.
- Long-lived subject, short-lived listener → check `unsubscribe()` first.

