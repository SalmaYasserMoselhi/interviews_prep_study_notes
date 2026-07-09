# Threading vs Multiprocessing — Notes

Goal: understand **how they work** + **when to use each** + the **Python GIL**.

---

## 1. The core idea

- **Program running** = a **process** (the app while it's alive).
- Inside a process there are **threads** = workers that run code line by line.
- **1 thread** = one worker. **Multi-threading** = many workers at the same time.
- **Multi-processing** = many separate processes = many separate copies of the app, each with its own memory.

**Kitchen analogy:**
- **Threads** = many cooks in **ONE kitchen**, sharing the same fridge (shared memory).
- **Processes** = many **separate kitchens**, each with its own fridge (separate memory).

---

## 2. Thread vs Process

| | Threads | Processes |
|--|---------|-----------|
| Memory | **Shared** (light, fast) | **Separate** (heavier, more RAM) |
| Same app copy? | Yes, one copy | No, separate copies |
| Talk to each other | Easy (shared memory) | Harder (needs IPC) |
| Race conditions | Possible → need a lock | No (isolated) |
| Startup cost | Cheap | Expensive |
| Best for | **I/O-bound** work | **CPU-bound** work |

---

## 3. When to use which (the #1 rule)

| Type | Spends its time... | Real example | Use |
|------|--------------------|--------------|-----|
| **I/O-bound** | **Waiting** (network, DB, files) | Fetching prices from 4 airline APIs | **Multi-threading** |
| **CPU-bound** | **Calculating** (math, images, video) | Applying a heavy filter to 4 images | **Multi-processing** |

**Why in one line:** a thread helps when work **waits a lot** (while one waits, another runs). A process helps when work is **heavy calculation** (needs real CPU cores running together).

---

## 4. Multi-threading example — flight booking app "Rehla" (I/O-bound)

**Idea:** user searches a ticket. We must get the price from **4 airlines**, each API call takes ~2s **waiting on the network**.
- Without threads: one after another = 2+2+2+2 = **8 seconds**.
- With threads: all 4 together = **~2 seconds** (while waiting for EgyptAir, we call Emirates).

```python
import threading, time

def fetch_price(airline):
    print(f"asking {airline}...")
    time.sleep(2)                 # waiting for the API response (I/O — no CPU work)
    print(f"{airline} replied")

airlines = ["EgyptAir", "Emirates", "Qatar", "Turkish"]

threads = []
for a in airlines:
    t = threading.Thread(target=fetch_price, args=(a,))
    t.start()                     # start the worker
    threads.append(t)

for t in threads:
    t.join()                      # wait for all to finish

# ~2 seconds instead of 8 — while one thread waits, another runs
```

**Why threads here?** The work **waits** on the network. A waiting thread uses no CPU, so the CPU is given to another thread. Also threads share memory → light and fast.

> Note: threads do NOT have to run the same function. In Uber, one thread runs `get_nearby_drivers(ahmed)`, another runs `calculate_fare(mona)`, another runs `send_notification(sara)` — all at the same time, sharing the same memory.

---

## 5. Multi-processing example — photo studio (CPU-bound)

**Idea:** 4 images uploaded, apply a **heavy filter** to each (pure CPU work, no waiting).
- With threads in Python: **no speedup** (because of the GIL — see section 7).
- With processes: each image on its **own CPU core** → **true parallel** → faster.

```python
from multiprocessing import Pool

def process_image(img):
    total = 0
    for i in range(50_000_000):   # heavy CPU work (resize / filter / compress)
        total += i
    return f"{img} done"

images = ["img1", "img2", "img3", "img4"]

if __name__ == "__main__":        # required for multiprocessing
    with Pool(4) as p:            # 4 processes = 4 cores
        results = p.map(process_image, images)
    print(results)

# each image on a separate core → real parallel execution
```

**Why processes here?** Heavy CPU work needs real cores running at once. Threads can't (GIL). Each **process has its own Python interpreter + its own GIL** → runs truly in parallel.

---

## 6. Race condition + lock (shared memory danger)

Threads share memory, so if two threads **modify the same data** at the same time → wrong result. This is a **race condition**.

**Uber example:** Mona and Khaled both request a motorbike, and only driver "Sami" is free nearby.

| Time | Thread B (Mona) | Thread C (Khaled) |
|------|-----------------|-------------------|
| t1 | Is Sami free? ✅ | |
| t2 | | Is Sami free? ✅ (Mona hasn't taken him yet) |
| t3 | give Sami to Mona | |
| t4 | | give Sami to Khaled ❌ |

Sami got assigned to **two people**! Because both **checked before either took**.

**Fix = lock** the critical section (the small dangerous part), briefly:

```python
import threading
lock = threading.Lock()

def assign_driver(driver, user):
    with lock:                    # only one thread inside at a time
        if driver.is_free:
            driver.is_free = False
            print(f"{driver.name} assigned to {user}")
        else:
            print("driver taken, find another")
```

**Key rules about locks:**
- Lock only when you **write/modify** shared data. **Reading** (e.g. showing the driver list) needs no lock.
- Lock the **smallest** part, for the **shortest** time (while locked, other threads wait → too much locking = slow).
- **thread-safe** = code works correctly under many threads. A lock is *one* way to get it (read-only / immutable data is thread-safe with no lock).

---

## 7. The Python GIL (Global Interpreter Lock)

### GIL in one line
> **Only one thread can execute Python bytecode at a time.**

### The problem it solves — reference counting
Python manages memory with **reference counting**: every object has a counter of how many references point to it. When the counter hits **0**, Python frees it.

```python
a = [1, 2, 3]   # list refcount = 1
b = a           # refcount = 2
del a           # refcount = 1
b = None        # refcount = 0 → object deleted
```

Updating that counter is itself **3 steps** (read → +1 → write). If two threads update the **same object's** counter at once → the counter goes wrong:
- Counter too **high** → object never freed → **memory leak**.
- Counter too **low** → object freed while still used → **crash** (use-after-free).

This corrupts Python's internal memory management.

### The solution — the GIL
The GIL is a **real lock (mutex)** inside the interpreter. **Rule:** to run any Python bytecode, a thread must hold the GIL, and only one thread holds it at a time. So two threads can never update a refcount simultaneously → the interpreter stays safe.

**Why one big lock instead of a tiny lock per object?** Simpler and faster for normal (single-threaded) programs. The price: CPU-bound threads can't run in parallel.

### How threads take turns (the mechanism)
A thread holding the GIL releases it when:
- **~5ms pass** (a time-slice; then it's forced to hand over), OR
- **it starts I/O** (network, DB, file, sleep) → releases the GIL **immediately** while waiting.

That second point is the key: during I/O the GIL is released, so **another thread really runs** → this is why threading speeds up I/O-bound work.

### ⚠️ The trap — GIL does NOT make YOUR code thread-safe
The GIL protects the **interpreter** (refcounts), NOT your data. Your operation `counter += 1` is **multiple bytecodes**, and the GIL can switch threads **between** them → race condition still happens.

```python
counter = 0
def bad():
    global counter
    for _ in range(100000):
        counter += 1   # NOT atomic → race despite the GIL

# two threads → final counter is LESS than 200000 (wrong)
```
So you **still need your own lock** for shared writes. The GIL affects **speed** (no CPU parallelism), not **data safety**.

- **GIL is invisible** — you never write it or see it in code. It lives inside CPython.
- The only lock you write is **your own** `threading.Lock()`.

---

## 8. Interview one-liners

- **Thread vs Process:** "الـ **threads** بيشاركوا نفس الـ **memory** جوّه process واحد؛ الـ **processes** كل واحد memory منفصلة. threads خفيفة بس فيها خطر **race conditions**، processes معزولة بس أتقل."

- **When to use:** "بستخدم **threads** للشغل الـ **I/O-bound** (network, DB, files) — وقت الاستنى thread يشتغل بدل التاني. وبستخدم **processes** للشغل الـ **CPU-bound** (حسابات، صور، فيديو) عشان cores حقيقية تشتغل سوا."

- **GIL:** "الـ **GIL** بيخلّي **thread واحد بس ينفّذ Python في اللحظة** — عشان يحمي الـ **reference counting** الداخلي بتاع Python من التخريب. ده بيمنع CPU threads تشتغل parallel، بس بيتساب وقت الـ **I/O** فالـ threads بتنفع للـ I/O. مهم: الـ GIL **مش** بيخلّي كودي thread-safe، فبرضو بحط **Lock** بتاعي على الـ shared writes."

- **race condition:** "لما اتنين thread يعدّلوا نفس الـ **shared data** في نفس الوقت → نتيجة غلط. بحلّها بـ **lock** على الـ **critical section** بس، وبأقصر وقت."
