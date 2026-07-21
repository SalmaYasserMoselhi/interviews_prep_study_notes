# 🎬 YouTube — System Design Interview Simulation & Final Revision Guide

> **What this is:** a full mock interview for *"Design YouTube,"* walked through the **5-step framework**, with every concept that fits YouTube woven in *as it naturally arises* — plus **Traps & Wrong Answers** at each step, a night-before cheat sheet, flash cards, follow-ups, and an appendix for concepts that belong to *other* systems.
>
> **Night-before routine (~30 min):** skim each step's `📝 notes` + `⏱️ recap` + `🪤 traps`, then drill the `🃏 Flash Cards` and `❓ Follow-ups`.

---

## 🧭 The 5-Step Framework (memorize first)

> **"Clarify → Model → Design → Break → Trade."**
> *Ask · Size · Sketch · Stress · Sacrifice*

| Step | ⏱️ | Goal | Proves |
|---|---|---|---|
| **1. Clarify** | ~5m | Functional + non-functional, state **out of scope** | Structured thinking |
| **2. Model** | ~10m | Estimate → entities → API → **DB choice justified** | Technical judgment |
| **3. Design** | ~15m | **Write path**, then **read path** — no box without a reason | Practical depth |
| **4. Break** | ~5m | Bottleneck · SPOF · degradation · monitoring | ⭐ Seniority |
| **5. Trade** | ~5m | *"I bought X, I paid Y"* | ⭐ Highest signal |

> 🧠 **Mental model for the whole hour:** *Step 1 is your evidence locker. Every box you draw later must point back to something you wrote in Step 1 — or delete it.*

---
---

# 🔵 STEP 1 — Clarify & Scope

> 🎯 **Goal:** turn *"design YouTube"* (impossible in 45 min) into something buildable, and write down the constraints I'll cite all hour.

### 🎤 Interviewer
> *"Design YouTube."*

### 💭 You (thinking)
> YouTube is a decade of work. I can't build all of it — **the vagueness is the test.** They want me to **shrink the scope and say so.** I'll propose the core loop and park the rest, then confirm.

### 🗣️ You (out loud)
> *"YouTube is huge, so let me narrow it. I'll treat **uploading** and **watching** a video as the core, plus **transcoding** and **search by title**. I'll **park** comments, recommendations, live streaming, and monetization unless you want one in. Sound right?"*

### 🎤 Interviewer
> *"Good. Focus on upload and watch. Assume global users."*

---

### 🧠 The important half — Non-Functional Requirements (SCALAR-D)

> 💭 *Functional requirements are the same for everyone — every YouTube plays videos. The **non-functional requirements are where the design is decided.***

- **Scale?** ~100M daily users, global.
- **Read-heavy or write-heavy?** People **watch ≫ upload** → **read-heavy ~1000:1** ⭐
- **Latency?** Smooth playback — **buffering is the real failure.**
- **Consistency or availability?** A video appearing a minute late is fine → **eventual consistency / AP.**
- **Durability?** **Never lose an uploaded video.** ⭐

---

### 📝 Draft notes (stays on the board)

```
IN SCOPE:  upload · transcode · watch · search-by-title
PARKED:    comments · recommendations · live · monetization

SCALAR-D:
  Scale       → 100M DAU, global
  Access      → READ-HEAVY ~1000:1   ⭐ shapes everything
  Latency     → smooth playback (no buffering)
  Consistency → EVENTUAL / AP
  Durability  → NEVER lose a video   ⭐
```

### 🤔 Why
- **Scoping** is the first engineering decision; saying it aloud reads as a **choice**, not an omission.
- **Read-heavy** means optimize the **read path** (cache, replicas, CDN); the write path barely scales.
- **AP/eventual** is my **CAP decision**, made now — it lets me scale globally.
- **Durability** forces **replicated object storage** for the videos later.

### 📌 Concepts (memory bullets)

> 📌 **Functional vs non-functional** — verbs (what) vs adverbs (how well); NFRs drive the design.

> 📌 **Read-heavy vs write-heavy** — read-heavy = SAME data read MANY times → cache; it *inverts* the design.

> 📌 **CAP (AP vs CP)** — in a partition, pick stale data (AP) or an error (CP). Videos → AP; money → CP.

> 📌 **Scope reduction** — shrink the impossible prompt and *say* you're shrinking it.

### 🪤 Traps & Wrong Answers — Step 1

| Tempting wrong move | Why it's wrong | Better |
|---|---|---|
| Start drawing boxes immediately | The prompt is deliberately vague; designing blind = fail on structure | **Clarify + scope first**, always |
| Try to design *all* of YouTube | 4 min per subsystem → shallow everywhere → reads as *you're* shallow | **Park features out loud**, design the core |
| Assume read-heavy silently | Correct here, but assuming *quietly* is a failure mode | **State the assumption:** *"I'll assume 1000:1, stop me if not"* |

> ### ⏱️ 30-Second Recap — Step 1
> *"I scoped to **upload and watch**, parking the rest. It's **read-heavy at ~1000:1**, needs **smooth playback**, tolerates **eventual consistency** so it's an **AP** system, and demands **durability** — never lose a video. Those four facts will justify every decision I make."*

---
---

# 🟢 STEP 2 — Estimate, Data Model & API

> 🎯 **Goal:** put numbers on it (to *justify* components, not to be "right"), then define what's stored and how the world talks to it.

### 🎤 Interviewer
> *"Give me a rough sense of scale."*

### 💭 You (thinking)
> The point isn't the number — it's **manufacturing the justification** for the components I'm about to draw. I'll **round hard** to powers of ten.

---

### 🔢 The numbers (each: *what it is · how estimated · why it matters*)

**Traffic**
```
Reads (views):
  100M DAU × 5 videos/day = 500M views/day
  500M ÷ 100,000 sec/day  ≈ 5,000 reads/sec average
  × 3 for peak            ≈ 15,000 reads/sec   ⬅ design target
```
- **What it is:** how many video views per second at busy times.
- **How estimated:** DAU × videos-per-user ÷ ~100k seconds/day, ×3 because **traffic isn't flat** (evenings spike).
- **Why it matters:** 15K/sec of reads is the load the **read path** must survive → justifies **caching + CDN + replicas**.

```
Writes (uploads):
  ~500K uploads/day ÷ 100,000 ≈ 5 uploads/sec
```
- **What it is:** how many new videos per second.
- **How estimated:** rough industry-shape guess, ÷ seconds/day.
- **Why it matters:** **5/sec is tiny** → the write path barely needs scaling, and slow work (transcoding) can be **queued** without pressure.

**The ratio (cheapest, most valuable line)**
```
500M reads : 500K writes = 1000 : 1  → violently read-heavy ⭐
```
- **Why it matters:** confirms Step 1 — **the read path is the entire problem**; the write path is an afterthought.

**Storage**
```
500K uploads × 100 MB       = 50 TB/day (raw)
× ~4 renditions (240p→4K)   = 200 TB/day
× 365                       ≈ 70 PB/year   💀
```
- **What it is:** how much disk the videos consume per year.
- **How estimated:** uploads × avg size × number of quality versions × days.
- **Why it matters:** **70 PB is NOT a database problem** → forces **object storage (S3)**, not a DB.

**Bandwidth**
```
15,000 concurrent streams × ~1 MB/s ≈ 15 GB/s egress   💀
```
- **What it is:** data flowing *out* to viewers per second.
- **How estimated:** peak concurrent streams × bitrate per stream.
- **Why it matters:** **your servers can't push 15 GB/s** → forces a **CDN**.

> 📌 **Back-of-envelope estimation** — estimate to justify, not to be right. Round to powers of ten. The number is the bridge from requirement → component.

---

### 🎯 What the numbers *decided*

| Number | Forces |
|---|---|
| **1000:1** read-heavy | Cache + replicas + CDN; ignore write scaling |
| **70 PB/yr** | 🚫 Not a DB → **object storage** |
| **15 GB/s egress** | 🚫 Not my servers → **CDN** |
| **5 writes/sec** | Writes tiny → **async transcoding is free** |

> 🧠 *I didn't "choose" a CDN — the number chose it. I just read it out loud.*

---

### 🗂️ Data Model — entities + size/heat/growth

```
User    → id, name, email
Video   → id, uploader_id, title, duration, status, rendition_urls[]
            📏 ~2 KB/row (metadata only — the FILE is NOT here)
            🌡️ read constantly, written once → read-heavy
            📈 ~500K/day → ~180M rows/year
WatchEvent → user_id, video_id, ts   (append-only firehose → write-heavy)
```

> ⭐ **Key insight:** video *metadata* and video *bytes* are two different storage problems. Metadata is small/queried/relational → **DB**. The file is a huge blob I never query inside → **object storage**. The DB row holds only the **S3 key** — an *address*, not the *thing*.

**🔢 Unique IDs — a real sub-decision here.** Each video needs a unique ID. With a single DB, auto-increment works, but if I ever shard, two shards would generate colliding IDs, and a central ID-generating server would be a bottleneck and a single point of failure. So I'd use **Snowflake IDs** — timestamp + machine ID + counter — which are collision-free without coordination, compact, and **time-sortable**. For short share links I'd Base62-encode the ID.

> 📌 **Unique ID generation** — sharded systems can't use auto-increment; **Snowflake** = timestamp+machine+counter (unique, compact, sortable); **Base62** shortens it.

---

### 📡 API Contract
```
POST /v1/videos             → 201 + { uploadUrl }   (pre-signed URL)
GET  /v1/videos/:id         → 200 metadata
GET  /v1/videos/:id/stream  → 200 manifest
GET  /v1/search?q=&cursor=  → 200 paginated
```
Plural nouns, method = verb, versioned, **cursor pagination**.

> 📌 **Cursor pagination** — bookmark the last item's **ID**, not a position (offset). Offset **scans and discards** every earlier row (slow deep pages) and **skips/duplicates** when new videos are inserted. Cursor **seeks via the index** → constant-time + stable. Feeds don't need "jump to page."

---

### 🗄️ Database choice — justified (an NFR *output*, not a preference)

> 💭 This is an NFR output, not a preference. Metadata needs some querying and grows large but is relational-ish → SQL (sharded later if needed) or a wide-column store. The append-only watch-event firehose is write-heavy → a wide-column store (Cassandra) or a stream, not my transactional DB. And the 70 PB of video → object storage.

```
Video/User metadata → SQL (Postgres); ~180M rows/yr = fine on one box + index
Watch events        → write-heavy firehose → wide-column (Cassandra) / stream
Video bytes (70 PB) → OBJECT STORAGE (S3), never the DB
```

> 📌 **SQL vs NoSQL** — default SQL, justify NoSQL. Relational + correctness → SQL. Write-heavy firehose → wide-column (Cassandra). NoSQL is a *trade* (joins/transactions for scale), not an upgrade.

> 📌 **Object storage vs DB** — bytes → S3 (the thing); metadata → DB (the address). If you never query inside it, it doesn't belong in a DB.

### 🪤 Traps & Wrong Answers — Step 2

| Tempting wrong move | Why it's wrong | Better |
|---|---|---|
| Store videos in the DB | 70 PB won't fit; blobs flush the DB cache, bloat backups, cost 10–50×, can't CDN | **Object storage**, DB holds the **S3 key** |
| Estimate, then ignore the number | The #1 estimation failure | Every number must **change a decision** |
| Offset pagination on the feed | Slow on deep pages + skips/duplicates on insert | **Cursor pagination** |

> ### ⏱️ 30-Second Recap — Step 2
> *"~**15K reads/sec** peak vs. **5 writes/sec** — confirming **1000:1 read-heavy**. **70 PB/year** of video means **object storage, not a database**. **15 GB/s egress** means a **CDN is mandatory**. The DB holds only **2KB metadata rows** pointing at S3 keys, so even 180M rows/year is a normal Postgres problem. I'd use **Snowflake IDs** so I can shard later without ID collisions, and **cursor pagination** on listings. Every one of those numbers justifies a component I'm about to draw."*

---
---

# 🟡 STEP 3 — High-Level Design

> 🎯 **Goal:** boxes and arrows — the heart of the round.
> 🧠 **The one technique:** *split the **write path** from the **read path**.* YouTube is wildly asymmetric — uploads rare/heavy, views constant/light. I add a box **only when I can name the pressure that demands it.**

---

## ✍️ WRITE PATH (upload — rare, heavy)

### 🎤 Interviewer
> *"Walk me through an upload."*

### 💭 You (thinking)
> The naive move is to POST the 100MB file to my API server. That turns my server into a **very expensive pipe** — it buffers 100MB in RAM and holds the connection for a minute; ten concurrent uploads kill it. Better: let the client upload **directly to object storage** via a **pre-signed URL**.

**① Pre-signed upload**
```
Client → API: "I want to upload"
API → Client: PRE-SIGNED URL (expires 15 min)
Client → S3 DIRECTLY: [100 MB]   ← API never touches a byte ⚡
```
- **Why:** keep the API a **metadata service**, not a 100MB pipe.
- **Without it:** the API buffers every upload → dies under a few concurrent uploads.
- **Why not proxy through the API:** wasted RAM, held connections, zero benefit.

**② Queue the slow work**
```
Upload done → S3 event → push job to a QUEUE
```
- **Why:** transcoding takes **minutes** — it must **never block** the request.
- **Solves:** **decouples** fast upload from slow processing; **buffers** bursts.
- **Without it:** user waits minutes, or an upload burst overwhelms transcoders.

> ⭐ **The queue is at-least-once, so transcode workers must be idempotent.** A retried "transcode video 42" job must not produce duplicate renditions — I key the output by `videoId+rendition` so a re-run overwrites, not duplicates.
> 📌 **Delivery guarantees** — real world = **at-least-once** = duplicates happen → make consumers **idempotent** (a retry = same result). "Exactly-once" is mostly at-least-once + dedup.

**③ Transcoding workers**
```
Consume queue → split into chunks → transcode 240p→4K in parallel →
write renditions to S3 → push to CDN → mark video "ready"
```
- **Why:** different devices/bandwidths need **multiple renditions** for **adaptive bitrate**.
- **Without it:** everyone streams one huge file → buffering on slow connections (our #1 failure).
- **Auto-scale workers off queue depth** — backlog grows → add workers.

```
📱 ─pre-signed─→ 📦 S3 ─event─→ 📨 Queue ─→ 👷 Transcoders ─→ 📦 S3 ─→ 🌍 CDN
                                                   └─status─→ 🗄️ DB "ready"
```

---

## 📖 READ PATH (watch — constant, light)

### 🎤 Interviewer
> *"Now I click play."*

### 💭 You (thinking)
> This is the 15K/sec, 15 GB/s, global path — the whole problem. **Physics** is my enemy: a round trip from a viewer in Egypt to a US origin is ~150ms, and playback needs several round trips. **No code fixes the speed of light** — I have to **shorten the distance**. That's a **CDN**.

**① CDN at the edge**
```
📱 Watch → 🌍 CDN edge (nearby)
   ✅ HIT (~90%) → ~20ms, origin never knows ⚡
   ❌ MISS → S3 origin → cache at edge → serve
```
- **Why:** **150ms cross-continent is physics**, and **15 GB/s egress** no origin can serve.
- **Solves:** ~90% of traffic **never reaches us** → speed + massive origin offload + DDoS absorption.
- **Without it:** worldwide buffering + servers melt under 15 GB/s.
- **Why not just more servers:** you can't out-scale the speed of light — distance is the problem.

**② Load balancer + reverse proxy** (for metadata/API calls)
```
📱 → CDN (miss) → ⚖️ LB → 🖥️ app servers
```
- **Load balancer** — *distributes requests **and** drops dead servers via **health checks** (deep checks that verify DB/cache reachability, not just "process alive"). Makes horizontal scaling deliver availability. Run it redundantly — it's a SPOF otherwise.*
- **Reverse proxy (Nginx)** — *TLS termination, compression, static files done once at the server edge. An LB is one feature of a reverse proxy; often the same box.*
- **Without them:** no way to spread load or detect a dead server → one crash takes users down.

**③ Stateless app servers**
```
🖥️🖥️🖥️  interchangeable — any server, any request
```
- **Statelessness** — *no session in server memory; state lives in Redis/DB. The precondition for horizontal scaling — add/remove/lose any node freely.*
- **Without it:** sessions in RAM → users randomly logged out when routed to a different server; uploaded temp files vanish; crons run N times.

**④ Redis metadata cache**
```
metadata → 💾 Redis (hot videos) → 🗄️ DB (on miss)
```
- **Redis cache-aside** — *check cache → miss → read DB → fill. Memory ≫ disk (~1000×). Hot videos' metadata is read constantly.*
- **Without it:** every view hits the DB → DB is the bottleneck.
- **Guardrail:** **TTL** (auto-expire) **+ invalidate on update** — never cache without an expiry, or stale data lives forever.

> 📌 **TTL & invalidation** — TTL bounds staleness; invalidate (delete) on write; use both, TTL as the backstop.

**⑤ Read replicas**
```
✍️ writes → 🗄️ LEADER      📖 reads → 📑 Followers
```
- **Read replicas** — *leader writes, followers read → scales reads + gives failover. Cost: replication lag → brief staleness (fine, we're AP). Read-your-own-writes: send a user's reads to the leader briefly after they write.*
- **Without it:** cache misses all pile onto one DB.

**⑥ Adaptive bitrate playback**
```
Player switches 240p↔4K based on live bandwidth
```
- **Why:** smooth playback on any connection — buffering is our **defined failure mode**.

```
📱 → 🌍 CDN (90% stop here) → ⚖️ LB → 🖥️ stateless app → 💾 Redis → 📑 DB replica
     video bytes stream from CDN/S3, never through the app
```

---

## 🗺️ Both paths together
```
       WRITE                              READ
   📱 upload                             📱 watch
      │ pre-signed                          │
      ▼                              🌍 CDN edge (90% ⚡)
   📦 S3 ─event─► 📨 Queue                   │ miss
                    │                 ⚖️ Load Balancer (+reverse proxy)
                    ▼                        │
               👷 Transcoders        🖥️ Stateless app servers
                    │ renditions             │
                    ▼                 💾 Redis (metadata cache)
               📦 S3 ─► 🌍 CDN                │ miss
                    │ status          📑 DB read replicas ◄─writes─ 🗄️ Leader
                    ▼
               🗄️ Metadata DB (leader)

  DB = metadata + S3 keys only.  Bytes live in S3 + CDN.  AP / eventual.
```

> 🧠 **Check:** every box earned its place — CDN (physics + egress), queue (minutes-long transcode), Redis (1000:1 reads), replicas (spread reads), pre-signed URLs (don't pipe blobs). Nothing is decoration.

**On sharding / consistent hashing:** I'm **not sharding** the metadata DB — 180M rows/yr fits one box + index. *If* it grew past that, I'd shard by `video_id`, and both my **Redis Cluster** and **CDN** already distribute keys using **consistent hashing** so adding a node moves only ~1/N of keys, not everything.

> 📌 **Consistent hashing** — `% N` reshuffles everything on a node change; a hash *ring* moves only ~1/N. Used by Redis Cluster, CDNs, Cassandra.

### 📌 Concepts (memory bullets)

> 📌 **Pre-signed URL** — client uploads straight to S3; API never touches bytes.

> 📌 **Message queue** — to-do list between services; decouple, buffer, respond instantly.

> 📌 **Idempotency** — a retry = same effect as one call; required because queues are at-least-once.

> 📌 **CDN** — edge caches near users; 150ms cross-continent is physics; serves ~90%; cache by URL not user.

> 📌 **Load balancer** — distribute + detect dead servers (deep health checks); run it redundantly.

> 📌 **Reverse proxy** — TLS/compression/static at the edge; an LB is one feature of it.

> 📌 **Statelessness** — no state in the server; precondition for horizontal scaling.

> 📌 **Redis cache-aside** — check → miss → load → fill; always TTL + invalidate.

> 📌 **Read replicas** — leader writes, followers read; scales reads + failover; cost = lag.

### 🪤 Traps & Wrong Answers — Step 3

| Tempting wrong move | Why it's wrong | Better |
|---|---|---|
| POST the 100MB file to your API | API becomes a pipe: buffers RAM, holds connections, dies under load | **Pre-signed URL** → client → S3 directly |
| "Just add more servers" for the read path | Can't out-scale the speed of light or 15 GB/s egress | **CDN** — shorten distance, offload origin |
| Sessions in server memory | Users randomly logged out when LB routes elsewhere | **Stateless** servers; state in Redis |

> ### ⏱️ 30-Second Recap — Step 3
> *"I **split the write path from the read path**. Writes: **pre-signed upload straight to S3**, then a **queue** feeds **async, idempotent transcoders** that produce multiple renditions. Reads: a **CDN** absorbs ~90% of the 15 GB/s (physics — you can't out-code the speed of light), behind it a **load balancer** over **stateless app servers**, with **Redis** caching hot metadata and **read replicas** spreading the rest. The **DB holds only metadata and S3 keys** — the video bytes live in S3 and the CDN. I'm not sharding yet."*

---
---

# 🟠 STEP 4 — Bottlenecks & Reliability

> 🎯 **Goal:** stress my own design before the interviewer does. **The seniority step.**

### 🎤 Interviewer
> *"What breaks at scale?"*

**🔥 1. Bottleneck?**
```
Egress            → CDN absorbs it (quantify hit ratio)
Transcode backlog → AUTO-SCALE workers off queue depth; degrade: low-res first
DB reads          → Redis + replicas already handle it
```

**💀 2. Single points of failure?**
```
One CDN region  → multi-region CDN
One DB leader   → replicas + automatic failover
One S3 region   → multi-region object storage
One LB          → active-passive pair (redundancy)
```
- **HA = redundancy + failover + health checks.** Hunt every "one X."

**📉 3. Overload/failure → Graceful degradation**
```
Recommendations down? → STILL PLAY THE VIDEO ✅
Comments down?        → STILL PLAY THE VIDEO ✅
Transcoder backed up? → serve low-res now, HD later ✅
```
- **Graceful degradation** — *drop the non-essential feature, keep the core. Lose dessert, not dinner. #1 seniority signal.*

**🔔 4. Know it's broken → Monitoring**
```
Watch: p99 startup latency · rebuffer ratio · transcode queue depth · CDN hit ratio · error rate
Trace: correlation IDs across services
Alert: page when error rate or p99 crosses a threshold
```

**🚦 Abuse protection — rate limiting.** To stop a client hammering the upload or API tier, I'd rate-limit per user with a **token bucket** (allows short bursts), keeping the counter in **Redis** so it's shared across all servers — otherwise a user hitting 5 servers gets 5× the limit. Over-limit returns a **429**.

> 📌 **Rate limiting** — cap requests/user (token bucket, allows bursts); counter in Redis (shared); returns 429.

---

### 🌩️ Follow-up: the viral video
### 🎤 Interviewer
> *"A video gets 10M views/hour. What happens?"*
### 🗣️ You
> *"The **CDN absorbs it** — that's its job. The risk is a **cache stampede** on **first publish**: before the edge is warm, a flood of misses could hit the origin. I'd **pre-warm** the cache for expected-hot content and use **request coalescing** so one request refills a hot key while the rest wait."*

> 📌 **Cache stampede** — hot key expires → mass miss → DB/origin flood. Fix: lock/coalesce or pre-warm.

### 📝 Draft notes
```
Bottleneck  → egress (CDN) · transcode (auto-scale off queue depth)
SPOF        → multi-region CDN/S3 · DB failover · redundant LB
Degrade     → recs/comments down → still play the video
Monitor     → p99, rebuffer, queue depth, CDN hit ratio + correlation IDs
Abuse       → rate limit (token bucket, Redis, 429)
Viral       → CDN absorbs; pre-warm + coalesce (stampede)
```

### 📌 Concepts (memory bullets)

> 📌 **High availability** — enemy = single point of failure; fix = redundancy + failover + health checks.

> 📌 **Graceful degradation** — non-essential fails → drop it, keep the core.

> 📌 **Auto-scaling** — adjust capacity to load (queue depth); needs statelessness.

> 📌 **Monitoring** — 4 golden signals (latency, traffic, errors, saturation) + correlation IDs.

> 📌 **Rate limiting** — token bucket, counter in Redis, returns 429.

> 📌 **Cache stampede** — hot key expires → flood; fix = coalesce or pre-warm.

### 🪤 Traps & Wrong Answers — Step 4

| Tempting wrong move | Why it's wrong | Better |
|---|---|---|
| Only describe the happy path | Juniors describe how it works; seniors say **what breaks** | Name **bottleneck, SPOF, degradation, monitoring** |
| Leave a lone load balancer / single region | You just moved the SPOF, not removed it | **Redundant LB, multi-region CDN/S3** |
| "If recs fail, the page errors" | One non-essential service takes down playback | **Degrade** — still play the video |

> ### ⏱️ 30-Second Recap — Step 4
> *"Main bottleneck is **egress**, solved by the **CDN**; transcoding backlog I'd handle by **auto-scaling workers off queue depth**. I remove **SPOFs** with **multi-region CDN/S3, DB failover, and a redundant LB**. On failure I **degrade gracefully** — recommendations down, the **video still plays**. I'd monitor **p99, rebuffer ratio, queue depth, and CDN hit ratio** with **correlation IDs**, and **rate-limit** abuse with a token bucket in Redis. A viral video is **absorbed by the CDN**; I'd **pre-warm** to avoid a **stampede** on first publish."*

---
---

# 🔴 STEP 5 — Trade-offs

> 🎯 **Goal:** say the bill out loud. **This 60-second summary disproportionately drives the score.** Protect it.

### 🎤 Interviewer
> *"Summarize your design and its trade-offs."*

### 🗣️ You (out loud)
> *"I optimized for **scale and availability**, and here's what I paid:*
> - *Chose **eventual consistency (AP)** — a video appears after processing — to get **availability and global scale**. If **strong consistency** were required, the design changes fundamentally.*
> - *Put **video bytes in object storage + CDN**, never the DB, because of **70 PB blob size** and **15 GB/s global reads**. The DB holds only **metadata + S3 keys**.*
> - *Made **transcoding async behind a queue**, accepting a video **isn't instantly watchable** — the trade for responsive uploads and burst tolerance, with **idempotent workers** for safe retries.*
> - *Biased the whole design to the **read path** (**CDN, Redis, replicas**) because it's **1000:1 read-heavy**; the write path barely scales.*
> - ***Did not shard** the metadata DB — it fits one box; sharding would cost **joins and transactions**. **Snowflake IDs** keep sharding open as a future move.*
>
> *With more time: **live streaming** (a different low-latency path), plus **recommendations** and **comments** as independent, degradable services."*

### 📝 Draft notes
```
Optimized for: scale + availability
Paid with:
  • eventual consistency (late video)        ← for availability
  • bytes in S3+CDN, not DB                  ← blob size + global reach
  • async transcode (not instantly watchable)← responsiveness (idempotent retries)
  • read-path-biased design                  ← 1000:1 read-heavy
  • NO sharding yet                          ← keep joins/transactions
Extensions: live streaming · recommendations · comments (degradable)
```

### 🪤 Traps & Wrong Answers — Step 5

| Tempting wrong move | Why it's wrong | Better |
|---|---|---|
| Run out of time, skip trade-offs | You lose the **highest-signal minute** | **Protect the last 5 min** for this |
| "My design has no downsides" | Every design is a purchase; claiming none = you can't see them | **Name what you paid** for each choice |
| Over-engineer to sound advanced (shard, Cassandra everywhere) | Over-engineering is a **failure**, not enthusiasm | **Restraint** — justify each component, stop early |

> ### ⏱️ 30-Second Recap — Step 5
> *"I bought **scale and availability**, and I paid with **eventual consistency**, **async (delayed) video processing**, and a design **entirely biased toward reads** — which is correct because it's **1000:1 read-heavy**. I deliberately **avoided sharding** to keep joins and transactions. If the requirement flipped to **strong consistency**, the design would change fundamentally."*

---
---
---

# 📄 ONE-PAGE CHEAT SHEET

```
DESIGN YOUTUBE — 45-MIN PLAYBOOK
════════════════════════════════════════════════════════════
STEP 1 — CLARIFY (5m)
  Scope: upload + watch + transcode + search. Park the rest.
  SCALAR-D → read-heavy 1000:1 · smooth playback · AP/eventual · durable

STEP 2 — ESTIMATE + MODEL (10m)
  15K reads/s · 5 writes/s · 70 PB/yr · 15 GB/s egress
  70 PB → object storage   15 GB/s → CDN   DB = metadata + S3 keys
  Snowflake IDs · cursor pagination

STEP 3 — DESIGN (15m) ── SPLIT THE PATHS ──
  WRITE: client →(pre-signed)→ S3 → queue → idempotent transcoders → S3 → CDN → DB"ready"
  READ:  client → CDN(90%) → LB(+reverse proxy) → stateless app → Redis → DB replica
         bytes stream from CDN/S3, never through the app
  (consistent hashing if Redis Cluster / sharding ever needed)

STEP 4 — BREAK (5m)
  Egress→CDN · transcode→auto-scale off queue depth
  SPOF: multi-region CDN/S3 · DB failover · redundant LB
  Degrade: recs/comments down → STILL PLAY VIDEO
  Monitor: p99 · rebuffer · queue depth · CDN hit ratio · correlation IDs
  Rate limit: token bucket in Redis (429).  Viral → CDN + pre-warm (stampede)

STEP 5 — TRADE (5m)
  Bought scale + availability.
  Paid: eventual consistency · async transcode · read-biased · no sharding yet.
════════════════════════════════════════════════════════════
GOLDEN RULES: ask before you draw · every box needs a "because" ·
split read/write · name what you sacrificed · don't over-engineer.
```

---

# 🗺️ ONE-PAGE ARCHITECTURE DIAGRAM

```
                              🌍 GLOBAL USERS
                                    │
                    ┌───────────────┴────────────────┐
             UPLOAD │                          WATCH  │
                    ▼                                 ▼
        ┌──────────────────┐              ┌────────────────────┐
        │  🖥️ API (metadata) │              │   🌍 CDN EDGE       │
        │  returns          │              │  ~90% served ⚡     │
        │  pre-signed URL   │              └─────────┬──────────┘
        └────────┬─────────┘                        │ miss
                 │ client uploads                    ▼
                 │ DIRECT to S3            ┌────────────────────┐
                 ▼                         │ ⚖️ LOAD BALANCER    │
        ┌──────────────────┐              │ (+ reverse proxy)   │
        │  📦 OBJECT        │              └─────────┬──────────┘
        │  STORAGE (S3)     │◄─renditions            ▼
        │  70 PB · 11 nines │            ┌────────────────────────┐
        └────────┬─────────┘            │ 🖥️ STATELESS APP SERVERS │
                 │ upload event          └───────────┬────────────┘
                 ▼                                    ▼
        ┌──────────────────┐              ┌────────────────────┐
        │  📨 QUEUE         │              │ 💾 REDIS (metadata)  │
        │  buffers bursts   │              │ cache-aside + TTL    │
        └────────┬─────────┘              └──────────┬──────────┘
                 │                                    │ miss
                 ▼                         ┌──────────▼──────────┐
        ┌──────────────────┐              │ 🗄️ METADATA DB        │
        │  👷 TRANSCODERS   │─►S3─►CDN     │ leader ──► replicas  │
        │  240p→4K · idem.  │  + DB"ready" │ (writes)   (reads)   │
        │  auto-scale       │              └─────────────────────┘
        └──────────────────┘
  ▸ BYTES: S3 + CDN   ▸ METADATA: DB + Redis   ▸ AP / eventual
```

---

# 🃏 FLASH CARDS — one line per concept

**Framework & thinking**
- **5-step framework** — Clarify → Model → Design → Break → Trade (5/10/15/5/5).
- **Functional vs non-functional** — verbs vs adverbs; NFRs drive the design.
- **Scope reduction** — shrink the impossible prompt and say you're shrinking it.
- **Read-heavy vs write-heavy** — same data read many times → cache; it inverts the design.
- **Estimation** — estimate to justify, not to be right; round to powers of ten; the number picks the component.

**Data layer**
- **Object storage vs DB** — bytes → S3 (the thing); metadata → DB (the address).
- **SQL vs NoSQL** — default SQL, justify NoSQL; write-heavy firehose → wide-column (Cassandra).
- **Unique IDs** — sharded → no auto-increment; Snowflake (timestamp+machine+counter); Base62 to shorten.
- **Cursor pagination** — bookmark the last ID; fast + stable vs offset.
- **Read replicas** — leader writes, followers read; scales reads + failover; cost = lag.
- **Sharding** — splits data across machines; scales writes/storage; costs joins + transactions; delay it.
- **Consistent hashing** — `% N` reshuffles everything; a ring moves ~1/N; used by Redis Cluster/CDN/Cassandra.

**Consistency**
- **CAP** — in a partition, pick stale (AP) or error (CP); videos → AP, money → CP.
- **Strong vs eventual** — strong = correct-everywhere-but-slow; eventual = fast-but-stale; default eventual.

**Performance / caching**
- **CDN** — edge caches; 150ms cross-continent is physics; serves ~90%; cache by URL not user.
- **Cache-aside** — check → miss → load → fill; memory ≫ disk; always TTL + invalidate.
- **TTL & invalidation** — TTL bounds staleness; invalidate on write; use both.
- **Cache stampede** — hot key expires → flood; fix = coalesce or pre-warm.

**Traffic & structure**
- **Load balancer** — distribute + detect dead servers (deep health checks); run redundantly.
- **Reverse proxy** — TLS/compression/static at the edge; an LB is one feature of it.
- **Statelessness** — no state in server; precondition for horizontal scaling.
- **Horizontal vs vertical** — more boxes vs bigger box; start vertical, build stateless, scale out for availability.
- **Pre-signed URL** — client uploads directly to S3; API never touches bytes.

**Async & communication**
- **Message queue** — to-do list between services; decouple, buffer, respond instantly.
- **Sync vs async** — need the answer now → sync; slow work user shouldn't wait for → async.
- **Delivery guarantees** — real world = at-least-once → idempotent consumers.
- **Idempotency** — a retry = same effect as one call; the safety net for retries.
- **Adaptive bitrate** — multiple renditions; player switches by live bandwidth.

**Reliability**
- **High availability** — enemy = SPOF; fix = redundancy + failover + health checks.
- **Graceful degradation** — drop the non-essential feature, keep the core.
- **Auto-scaling** — adjust capacity to load (queue depth); needs statelessness.
- **Monitoring** — 4 golden signals (latency, traffic, errors, saturation) + correlation IDs.
- **Rate limiting** — token bucket, counter in Redis, returns 429.

---

# ❓ COMMON FOLLOW-UP QUESTIONS

**Q: Why not store videos in the database?**
> 70 PB won't fit; blobs **flush the DB's memory cache**, make **backups/replication crawl**, cost **10–50× per GB**, and can't sit behind a **CDN**. Metadata and bytes are **two problems** — the DB holds a **string (the S3 key)**.

**Q: How do uploads scale without killing your servers?**
> **Pre-signed URLs** — the client uploads **directly to S3**; my API handles **200 bytes, not 100 MB**.

**Q: Why is transcoding asynchronous — and how do you handle retries?**
> It takes **minutes**; doing it inline would **timeout**. A **queue** decouples and buffers it, and I **auto-scale workers off queue depth**. Queues are **at-least-once**, so workers are **idempotent** — output keyed by `videoId+rendition`, so a retry **overwrites** instead of duplicating.

**Q: How do you handle a viral video?**
> The **CDN absorbs** it. Risk is a **stampede on first publish** before the edge is warm — I'd **pre-warm** and use **request coalescing**.

**Q: Would you shard the database?**
> **Not yet** — ~180M rows/yr fits **one Postgres box + index**. I'd shard by `video_id` **only** when it outgrows that, because sharding costs **joins, transactions, and ops pain**. **Snowflake IDs** keep that option open.

**Q: Is this CP or AP?**
> **AP** — a video appearing a **minute late is fine**, so I favor **availability + eventual consistency** to scale. Payments would be **CP**.

**Q: Recommendations service goes down — what happens?**
> The **video still plays.** Recommendations are **non-essential** → **graceful degradation**.

**Q: How do slow connections avoid buffering?**
> **Adaptive bitrate** — multiple renditions (240p→4K), player switches by **live bandwidth**.

**Q: CDN vs Redis — what's the difference here?**
> **CDN** caches **video bytes** near users (egress + latency). **Redis** caches **metadata** near the app (DB offload). Different data, different layer.

**Q: How do you keep unique video IDs across a sharded system?**
> **Snowflake IDs** — timestamp + machine ID + counter → collision-free **without coordination**, compact, and **time-sortable**.

**Q: How do you protect the API from abuse?**
> **Rate limiting** — **token bucket** per user, counter in **Redis** so it's **shared across servers**; over-limit → **429**.

**Q: How do you know the system is healthy?**
> **p99 startup latency, rebuffer ratio, transcode queue depth, CDN hit ratio, error rate**, with **correlation IDs** to trace a request across services and **alerts** on breaches.

---

# 🧯 APPENDIX A — MASTER "TRAPS & WRONG ANSWERS" TABLE

*(All step-traps in one place for fast night-before review.)*

| # | Trap | Why wrong | Right move |
|---|---|---|---|
| 1 | Draw before clarifying | Prompt is vague on purpose | Clarify + scope first |
| 2 | Design all of YouTube | Shallow everywhere | Park features, design the core |
| 3 | Assume read-heavy silently | Silent assumption = failure mode | State it out loud |
| 4 | Precise numbers | Fake precision on a guess | Round to powers of ten |
| 5 | Videos in the DB | 70 PB, flushes cache, no CDN | Object storage + S3 key |
| 6 | Estimate then ignore | #1 estimation failure | Every number changes a decision |
| 7 | Auto-increment IDs | Collide when sharded | Snowflake / UUID |
| 8 | Offset pagination | Slow deep pages + skip/dupe | Cursor pagination |
| 9 | POST 100MB to API | API becomes a pipe | Pre-signed URL → S3 |
| 10 | Sync transcoding | Timeout + blocked user | Queue + async workers |
| 11 | Non-idempotent workers | At-least-once → duplicate renditions | Idempotent, keyed output |
| 12 | "More servers" for reads | Can't beat physics/egress | CDN |
| 13 | Sessions in server memory | Random logouts | Stateless + Redis |
| 14 | Shallow health check | Passes while 500-ing | Deep health check |
| 15 | Cache with no TTL | Stale forever | TTL + invalidate |
| 16 | Shard the DB now | Costs joins/transactions | Don't shard until you must |
| 17 | Only the happy path | No seniority signal | Bottleneck/SPOF/degrade/monitor |
| 18 | Lone LB / single region | SPOF remains | Redundant LB, multi-region |
| 19 | Recs down → page errors | Non-essential kills core | Graceful degradation |
| 20 | Per-server rate limit | User × N servers = N× limit | Shared counter in Redis |
| 21 | Ignore first-publish spike | Cold edge → stampede | Pre-warm + coalesce |
| 22 | Skip trade-offs (out of time) | Lose the highest-signal minute | Protect the last 5 min |
| 23 | Over-engineer to look advanced | Over-engineering is a failure | Restraint; justify each box |

---

# 🗃️ APPENDIX B — CONCEPTS NOT USED HERE (and where they belong)

*These came up in the course but don't fit a scoped YouTube read/watch design. Know them — just know they belong to other prompts.*

| Concept | Why not here | Where it IS the answer |
|---|---|---|
| **Fan-out on write/read** | We parked the subscriptions feed | **News feed / Twitter / Instagram** (celebrity problem) |
| **Saga / 2PC / compensating txns** | No multi-service atomic transaction (no money movement) | **E-commerce checkout, payments** (charge + inventory + order) |
| **Distributed lock** | No "only one worker may act on a shared resource" need | **Payment double-click, inventory reservation** |
| **Consensus / Raft / quorum** | We don't hand-build leader election (managed DB does it) | **Building a DB/coordination service; ZooKeeper/etcd** |
| **CQRS** | Read/write needs don't diverge enough to justify it | **Systems with very different read vs write models** |
| **WebSockets / SSE / long polling** | Video playback is HTTP streaming, not live push | **Chat (WebSockets), live scores/notifications (SSE)** |
| **gRPC** | Only 2–3 internal services; REST is fine | **Large microservice meshes, fast internal calls** |
| **GraphQL** | Simple, cacheable endpoints; CDN loves REST URLs | **Mobile apps assembling complex screens in one call** |
| **Geospatial indexing (geohash/quadtree)** | No "find nearby" query | **Uber / food delivery ("drivers near me")** |
| **Inverted index (Elasticsearch)** | We kept search to simple title match | **Full-text / search autocomplete** |
| **Bloom filter** | No hot "does this key exist" path to protect | **Cache penetration, "username taken?" checks** |
| **CRDT / OT** | No real-time collaborative editing | **Google Docs, collaborative editors** |
| **Service discovery / API gateway** | Only a couple of services; overkill | **Large microservice systems** |
| **Connection pooling / PgBouncer** | True in practice, but too low-level for the HLD story | **Any DB under many app servers (mention if pushed)** |
| **PACELC** | CAP was enough for the consistency call | **Deep consistency discussions** |
| **ACID vs BASE** | Implied by SQL-metadata + AP; not the focus | **DB-choice-heavy designs** |
| **Monolith vs microservices** | Not asked; we split by path, not by service | **"How would you structure the org/system" questions** |
| **DNS internals, L4 vs L7 detail, eviction policies (LRU/LFU)** | One-liners at most; not decision drivers here | **Deep-dive follow-ups if the interviewer digs** |

> 💡 **How to use this table:** if an interviewer pulls the design toward one of these (*"now add live chat during the video"* → WebSockets; *"now recommend related videos"* → fan-out / inverted index), you know exactly which tool to reach for.

---

# 🎯 TOP INTERVIEW KEYWORDS (say these naturally)

> **functional / non-functional requirements** · **read-heavy (1000:1)** · **back-of-envelope estimate** · **object storage** · **pre-signed URL** · **write path / read path** · **message queue** · **at-least-once / idempotent** · **async / decoupling** · **transcoding / renditions** · **adaptive bitrate** · **CDN / edge / hit ratio** · **load balancer / deep health checks** · **reverse proxy** · **stateless** · **Redis / cache-aside / TTL** · **read replicas / replication lag** · **eventual consistency / AP** · **consistent hashing** · **Snowflake ID** · **single point of failure** · **redundancy / failover** · **graceful degradation** · **auto-scaling** · **monitoring / p99 / correlation IDs** · **rate limiting / token bucket** · **cache stampede / pre-warm** · **don't shard until you must** · **trade-offs**

---

> ### ⚡ THE 5 THINGS THAT CARRY ANY DESIGN
> 1. **Ask before you draw** — NFRs (read/write ratio, consistency) decide everything.
> 2. **Every box needs a "because"** — trace it to a requirement or a number, or delete it.
> 3. **Split the read path from the write path** — optimized by opposite moves.
> 4. **It's all one fork** — correctness vs availability, cost vs simplicity. Name what you sacrificed.
> 5. **Climb cheapest-first and know when to STOP** — over-engineering is a failure, not enthusiasm.

---

*End of guide. Read the recaps + traps + flash cards the night before, then do this design out loud, timed to 45 min, until the framework is muscle memory. Good luck. 🚀*
