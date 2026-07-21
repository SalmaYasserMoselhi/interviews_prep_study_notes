# Interview Prep Master Checklist (Backend-leaning Full-Stack)

This is a complete topic checklist, not a schedule. Every section is split into:
- **Must Know** — high-frequency, asked constantly, ordered highest-priority first.
- **Good to Know** — less common but still comes up, ordered highest-priority first.

Check items off as you cover them. Build your own daily plan from this by picking however much fits your available time — Must Know first, always, across every section, before touching any section's Good to Know.

---

## Suggested 12-Day Schedule

One way to sequence the checklist — 9 full days + 3 half days. Must Know items first on each day; Good to Know only if time remains. **GP Review** = quick recap of previous day's topics before starting the new one.

| Day | Length | Focus | Pattern | DSA | Recap |
|---|---|---|---|---|---|
| 1 | Full | §2 OOP & SOLID | Singleton | Arrays + Two Pointers | [ ] |
| 2 | Full | §4 Databases | Factory | Strings + Sliding Window | [ ] |
| 3 | Full | §5 System Design (part 1) | Builder | Sorting + Searching | [ ] |
| 4 | Full | §5 System Design (part 2) | Observer | Binary Search | [ ] |
| 5 | Full | §6 Networking/OS/Linux + §7 Security | Strategy | Hashmaps | [ ] |
| 6 | Full | §8 Language Internals | Decorator | Linked List + Prefix Sum | [ ] |
| 7 | Full | Backend Framework Comparison | Adapter | Queue + Stack | [ ] |
| 8 | Full | §10 Frontend | Facade | Bit Manipulation + Recursion | [ ] |
| 9 | Full | §11 DevOps + §12 AI/ML/Data Science | DAO / Repository | Trees | [ ] |
| 10 | Half | §13 Testing + §14 Git | Abstract Factory | *Awareness:* Graphs + Tries | [ ] |
| 11 | Half | Good-to-Know sweep + full pattern quiz | — | *Awareness:* DP + Monotonic Stack + Priority Queue | [ ] |
| 12 | Half | §15 Behavioral & CS Trivia + full Must Know review | — | Cumulative review (Must Know only) | [ ] |

---

## Contents

**Topic checklist**
1. [Data Structures & Algorithms](#1-data-structures--algorithms)
2. [Object-Oriented Programming & SOLID](#2-object-oriented-programming--solid)
3. [Design Patterns](#3-design-patterns)
4. [Databases (SQL & NoSQL)](#4-databases-sql--nosql)
5. [System Design](#5-system-design) — incl. [the design framework](#51--the-design-framework-the-way-of-thinking), [practice designs](#54--practice-designs-ordered-by-how-often-theyre-asked), [worked "Design YouTube" walkthrough](#55--worked-walkthrough-design-youtube)
6. [Networking, OS & Linux](#6-networking-os--linux)
7. [Security](#7-security)
8. [Programming Language Internals](#8-programming-language-internals)
9. [Backend Frameworks & APIs](#9-backend-frameworks--apis)
10. [Frontend](#10-frontend)
11. [DevOps](#11-devops)
12. [AI/ML, Data Science & Data Analysis](#12-aiml-data-science--data-analysis)
13. [Testing](#13-testing)
14. [Git & Version Control](#14-git--version-control)
15. [Behavioral & CS Trivia](#15-behavioral--cs-trivia)

**Framework revision**
- [Backend Framework Comparison (Day 7)](#backend-framework-comparison-day-7--concept-first)
- [Frontend Framework Comparison (Day 8)](#frontend-framework-comparison-day-8--concept-first)
- [Stack-Specific Revision (T-3 Days)](#stack-specific-revision-t-3-days)

**Planning**
- [Suggested 12-Day Schedule](#suggested-12-day-schedule) *(top of file)*
- [Using the 2-week interview period](#using-the-2-week-interview-period-23-days-per-company)

---

## 1. Data Structures & Algorithms

### Must Know
- [ ] **Time & space complexity (Big-O)** — analyzing any solution, common classes (O(1)/O(log n)/O(n)/O(n log n)/O(n²)), the tradeoff you'll be asked to state on every problem
- [ ] **Big-O vs. Theta vs. Big-Omega** — upper vs. tight vs. lower bound (the notation distinction)
- [ ] **Structure comparisons** — Stack vs. Queue, Array vs. Linked List, types of linked lists (singly/doubly/circular), BST operations incl. removing the root
- [ ] **Arrays** — traversal, in-place manipulation, common patterns
- [ ] **Strings** — manipulation, comparison, common patterns
- [ ] **Two Pointers** — converging/diverging pointers on sorted/unsorted arrays
- [ ] **Sliding Window** — fixed and variable window size problems
- [ ] **Sorting** — how merge sort/quick sort work, stability, when each is used
- [ ] **Searching** — linear search, general search patterns
- [ ] **Binary Search** — classic + on rotated arrays + search-space reduction
- [ ] **Hashmaps / Hash Tables** — collision handling, frequency counting, two-sum family
- [ ] **Linked List** — reversal, cycle detection (fast/slow pointers), merging
- [ ] **Prefix Sum** — range sum queries, subarray sum problems
- [ ] **Queue** — FIFO usage, BFS support
- [ ] **Stack** — LIFO usage, parentheses/expression validation
- [ ] **Bit Manipulation** — XOR tricks, bit masks, common interview shortcuts
- [ ] **Recursion** — base case/recursive case design, recursion tree thinking
- [ ] **Trees** — traversals (in/pre/post-order), BFS/DFS, BST properties

### Good to Know
- [ ] **Graphs** — BFS/DFS, what a shortest-path problem looks like
- [ ] **Tries** — prefix matching, autocomplete use case
- [ ] **Dynamic Programming** — memoization vs. tabulation, recognizing DP problems
- [ ] **Monotonic Stack** — next-greater-element style problems
- [ ] **Priority Queue / Heap** — top-K problems, scheduling
- [ ] **Data structures mapped to backend use cases** — e.g. hashmap for caching/dedup, queue for job processing, tree/index for lookups, set for membership

---

## 2. Object-Oriented Programming & SOLID

### Must Know
- [ ] **Four OOP pillars** — encapsulation, inheritance, polymorphism, abstraction
- [ ] **Why use OOP / what it solves** — the problem OOP addresses vs. procedural
- [ ] **Class vs. Instance/Object** — the blueprint-vs-instance distinction
- [ ] **Composition vs. Aggregation vs. Association vs. Inheritance** — the four relationship types and how they differ
- [ ] **Overloading vs. Overriding** — compile-time vs. runtime, signature vs. behavior
- [ ] **Types of Inheritance** — single, multiple, multilevel, hierarchical, hybrid
- [ ] **Types of Polymorphism** — compile-time (overloading) vs. runtime (overriding)
- [ ] **Interfaces vs. Abstract Classes** — when to use which
- [ ] **Diamond Problem** — multiple-inheritance ambiguity and how languages resolve it
- [ ] **Constructor vs. Destructor + constructor types** — default/parameterized/copy
- [ ] **Access modifiers** — public/private/protected and their scope
- [ ] **`final` vs. `static`** — and the aim of `this`/`self`
- [ ] **Composition vs. Inheritance** — "favor composition over inheritance," why
- [ ] **Is everything in Python an object?** — the "yes, including functions/classes" answer
- [ ] **Single Responsibility Principle** — one reason to change
- [ ] **Open/Closed Principle** — open for extension, closed for modification
- [ ] **Liskov Substitution Principle** — subtypes must be substitutable for base types
- [ ] **Interface Segregation Principle** — many specific interfaces over one general one
- [ ] **Dependency Inversion Principle** — depend on abstractions, not concretions
- [ ] **Dependency Inversion vs. Dependency Injection** — the principle vs. the technique (commonly confused)

### Good to Know
- [ ] **Object vs. Struct** — behavior + data vs. plain data
- [ ] **Data hiding vs. data binding** — the distinction
- [ ] **Private constructors** — use cases (Singleton, factory-only creation)
- [ ] **Coupling & Cohesion** — high cohesion, low coupling as a design goal
- [ ] **DRY / KISS / YAGNI** — as design heuristics, not just acronyms
- [ ] **Law of Demeter** — "don't talk to strangers" object interaction rule
- [ ] **Object-Oriented Design (OOD) process** — basic UML familiarity
- [ ] **Mixins / Traits** — language-specific composition mechanisms

---

## 3. Design Patterns

### Must Know
- [ ] **Singleton** (Creational) — single instance, when it's an anti-pattern
- [ ] **Factory Method** (Creational) — delegate object creation to subclasses
- [ ] **Builder** (Creational) — step-by-step construction of complex objects
- [ ] **Observer** (Behavioral) — subscribe/notify, event systems
- [ ] **Strategy** (Behavioral) — interchangeable algorithms behind one interface
- [ ] **Adapter** (Structural) — making incompatible interfaces work together
- [ ] **Decorator** (Structural) — adding behavior without subclassing
- [ ] **Facade** (Structural) — simplified interface over a complex subsystem
- [ ] **Repository** (Architectural) — abstraction over data access, works with domain objects
- [ ] **DAO (Data Access Object)** (Architectural) — abstraction over raw queries; how it differs from Repository
- [ ] **MVC** (Architectural) — Model/View/Controller separation

### Good to Know
- [ ] **Abstract Factory** (Creational) — families of related objects, vs. plain Factory
- [ ] **Prototype** (Creational) — cloning existing objects
- [ ] **Proxy** (Structural) — placeholder controlling access to another object
- [ ] **Composite** (Structural) — tree structures of objects treated uniformly
- [ ] **Bridge** (Structural) — decoupling abstraction from implementation
- [ ] **Flyweight** (Structural) — sharing to minimize memory use
- [ ] **Command** (Behavioral) — encapsulating a request as an object
- [ ] **Chain of Responsibility** (Behavioral) — passing a request along a handler chain
- [ ] **Iterator** (Behavioral) — sequential access without exposing structure
- [ ] **Template Method** (Behavioral) — skeleton algorithm, subclasses fill in steps
- [ ] **State** (Behavioral) — behavior changes based on internal state
- [ ] **Mediator** (Behavioral) — centralizing complex communication between objects
- [ ] **Memento** (Behavioral) — capturing/restoring object state (undo functionality)
- [ ] **Visitor** (Behavioral) — separating an algorithm from the objects it operates on

---

## 4. Databases (SQL & NoSQL)

### Must Know
- [ ] **SQL fundamentals (RDBMS)** — SELECT, WHERE, GROUP BY, HAVING, ORDER BY, subqueries
- [ ] **SQL command categories** — DML vs. DDL vs. DCL (and TCL)
- [ ] **HAVING vs. WHERE** — filtering before vs. after aggregation
- [ ] **Finding/removing duplicates** — GROUP BY/HAVING, DISTINCT, window functions
- [ ] **UNION vs. UNION ALL** — dedup vs. keep-all
- [ ] **DELETE vs. TRUNCATE vs. DROP** — rows-with-log vs. fast-empty vs. remove-table
- [ ] **Views** — what they are, when to use them
- [ ] **One-to-many query example** — joining and aggregating across a relationship
- [ ] **Joins** — INNER, LEFT, RIGHT, FULL, SELF, CROSS, with concrete examples
- [ ] **Keys** — primary key, foreign key, unique key, composite key
- [ ] **ACID properties** — Atomicity, Consistency, Isolation, Durability, each defined individually
- [ ] **Normalization** — 1NF, 2NF, 3NF, and why/when you'd denormalize
- [ ] **Indexing** — how a B-tree index works, when to add one, composite/covering indexes, the write-cost tradeoff
- [ ] **Index types** — primary vs. clustered vs. non-clustered, composite index column order
- [ ] **Validations vs. constraints** — application-level vs. database-level enforcement
- [ ] **Transactions & isolation levels** — Read Uncommitted / Read Committed / Repeatable Read / Serializable, plus dirty reads, non-repeatable reads, phantom reads
- [ ] **SQL vs. NoSQL** — when to choose which, concrete decision criteria
- [ ] **NoSQL document model (MongoDB)** — collections/documents, BSON, schema flexibility
- [ ] **ORM vs. ODM** — Sequelize/Mongoose/Eloquent/Active Record/Django ORM/SQLAlchemy
- [ ] **N+1 query problem** — what causes it, how to avoid it (eager loading)
- [ ] **Query optimization basics** — reading an EXPLAIN/query plan
- [ ] **Connection pooling** — why it matters, how it works

### Good to Know
- [ ] **Window functions & CTEs** — the WITH clause, ranking/running-total style queries
- [ ] **Aggregate vs. window functions** — collapsing rows vs. keeping them
- [ ] **LIKE vs. ILIKE** — case-sensitive vs. case-insensitive pattern matching
- [ ] **Solutions to speed up DB retrieval** — indexing, caching, denormalization, query tuning, read replicas
- [ ] **Data warehousing** — OLTP vs. OLAP, what a warehouse is for
- [ ] **MongoDB operations** — `.explain()`, index types, aggregation pipeline, pagination, upsert, `updateOne` vs. `findOneAndUpdate`, `$set` vs. `$unset`, embedding vs. referencing, `.populate()`, projection, cursor, ObjectID structure
- [ ] **BASE vs. ACID** — NoSQL's eventual-consistency model vs. relational guarantees
- [ ] **Primary vs. secondary node; replication vs. sharding** — MongoDB cluster concepts
- [ ] **BCNF and higher normal forms** — beyond 3NF
- [ ] **Locking** — optimistic vs. pessimistic, what causes deadlocks
- [ ] **Replication** — primary-replica setup, read replicas
- [ ] **Sharding / horizontal partitioning** — strategies and tradeoffs
- [ ] **Redis & key-value stores** — caching patterns, TTL, common data structures
- [ ] **Other NoSQL categories** — column-family (Cassandra), graph databases (Neo4j) at awareness level
- [ ] **Stored procedures, triggers, views** — what they're for
- [ ] **Database migrations / schema versioning** — tooling concept (Flyway, Liquibase, framework-native migrations)
- [ ] **Full-text search indexing** — concept level
- [ ] **Referential integrity & cascading deletes**
- [ ] **Backup & recovery basics**
- [ ] **CAP theorem as it applies to database choice** — cross-reference with System Design
- [ ] **PostgreSQL-specific features** — JSONB, array/composite types, full-text search (tsvector), extensions

---

## 5. System Design

> **Read this section in order.** The framework is the transferable skill; the concept lists are the vocabulary you plug into it; the practice designs are the reps; the YouTube walkthrough shows the whole thing assembled.

### 5.1 — The Design Framework (the way of thinking)

**The one thing every source agrees on:** there is no single right answer. Interviewers score *how you reason*, not what you memorised. Internalise this loop and you can attack any prompt — including one you've never seen.

#### The 5 steps — timings for a 40–45 min round

| # | Step | Time | What you do |
|---|---|---|---|
| 1 | **Clarify & scope** | ~5 min | Narrow the vague prompt. Functional + non-functional requirements. State what's **out** of scope. |
| 2 | **Data model & API** | ~10 min | Core entities, keys, data characteristics, API contracts, DB choice **with justification** |
| 3 | **High-level design** | ~15 min | Components + data flow for reads and writes. Keep it broad before deep. |
| 4 | **Bottlenecks & reliability** | ~5 min | What breaks at scale, single points of failure, graceful degradation, monitoring |
| 5 | **Trade-offs & extensions** | ~5 min | Summarise what you prioritised and sacrificed, and why |

#### What interviewers actually score

| Dimension | What they're looking for |
|---|---|
| **Communication & collaboration** | Do you ask clarifying questions and treat it as a conversation? |
| **Structured thinking** | Can you break down ambiguity systematically instead of jumping around? |
| **Technical judgment** | Can you explain *why this and not the alternative* for every choice? |
| **Practical depth** | Do you think about failure modes, or only the happy path? |
| **Time management** | Can you cover breadth *and* depth in the time given? |

#### Framework checklist

- [ ] **Functional vs. non-functional requirements** — *what it does* (features, use cases, users) vs. *how it must behave* (scale, latency, consistency, availability). Non-functional requirements drive nearly every technical decision.
- [ ] **Scope reduction** — "Design Instagram" is impossible; "design photo sharing" is doable. Ask which feature the interviewer actually cares about.
- [ ] **Read-heavy vs. write-heavy** — establish this early; it shapes caching, replication, and DB choice.
- [ ] **Back-of-envelope capacity estimation** — DAU, requests/sec, read:write ratio, storage growth (~1 year), bandwidth. Quantify before you design.
- [ ] **Checkpoint with the interviewer** — after each step, confirm the direction. It's a conversation, not a monologue.
- [ ] **Justify every choice** — a "correct" design you can't defend is worthless. For each component, be ready with *why this and not the alternative*.
- [ ] **HLD vs. LLD** — High-Level Design = overall architecture, major components, how they communicate. Low-Level Design = classes, objects, methods, internal logic. Know which one you're being asked for.
- [ ] **Scenario/ambiguous-question framework** — clarify ambiguity → state assumptions explicitly → structure high-level first → discuss trade-offs → connect to your own past experience → communicate with a diagram

---

### 5.2 — Must Know

#### Scaling & traffic
- [ ] **Horizontal vs. vertical scaling**
- [ ] **Load balancing** — round robin, least connections, IP hash; health checks/heartbeats; sticky sessions
- [ ] **Caching strategies** — cache-aside, write-through, write-back, write-around, TTL, invalidation
- [ ] **CDN basics** — what it caches and why; edge servers
- [ ] **Reverse proxy** — what sits in front of your servers and why (Nginx/HAProxy)

#### Data & storage
- [ ] **Database choice in system design** — SQL vs. NoSQL tradeoffs in context, justified against requirements
- [ ] **Replication & partitioning in design** — master-slave vs. master-master; sharding vs. partitioning; object storage for blobs/media
- [ ] **CAP theorem** — Consistency, Availability, Partition tolerance tradeoffs; different parts of one system may choose differently

#### Communication & APIs
- [ ] **Client-server model & REST API design principles**
- [ ] **API design essentials** — pagination, rate limiting, versioning, idempotency
- [ ] **Message queues / brokers** — what problem Kafka/RabbitMQ solve; async decoupling
- [ ] **Synchronous vs. asynchronous communication** — blocking + tightly coupled vs. resilient + eventually consistent
- [ ] **Communication protocols** — polling vs. long polling vs. WebSockets vs. SSE, and when each fits

#### Architecture
- [ ] **MVC architecture**
- [ ] **Microservices vs. monolith** — tradeoffs, when to pick which (rule of thumb: start monolithic if small, split as it grows)

#### Reliability & operations
- [ ] **Latency, throughput & availability** — the three core performance metrics; availability = uptime / (uptime + downtime); p50/p95/p99 latency
- [ ] **High availability & fault tolerance** — redundancy, replication, failover, eliminating single points of failure, graceful degradation
- [ ] **Monitoring, logging & alerting in a design** — structured logs, correlation/request IDs, error rates, queue depth; the thing that separates junior from senior answers

---

### 5.3 — Good to Know

#### Distributed systems
- [ ] **Consistent hashing** — minimising data movement when nodes are added/removed; virtual nodes
- [ ] **Leader election** — and what happens when the leader fails (Raft/Paxos at awareness level)
- [ ] **Service discovery** — how services find each other without hardcoded addresses
- [ ] **Distributed locking**
- [ ] **Eventual consistency vs. strong consistency**
- [ ] **Concurrency control in design** — locking, isolation levels, MVCC (cross-ref §4)

#### Architectural patterns
- [ ] **API Gateway** — single entry point pattern; routing, auth, rate limiting, aggregation
- [ ] **Event-driven architecture**
- [ ] **CQRS pattern**
- [ ] **Saga pattern** — distributed transactions
- [ ] **Two-phase commit (2PC)** — vs. sagas for cross-service atomicity
- [ ] **Circuit breaker pattern**
- [ ] **GraphQL vs. REST** — tradeoffs
- [ ] **Containerization in architecture** — why Docker/K8s fit microservices (cross-ref §11)

#### Caching deep-dive
- [ ] **Cache miss, cache stampede & cache avalanche** — what they are; fixes: staggered TTLs, request coalescing/mutex, early expiration, pre-warming
- [ ] **Cache eviction policies** — LRU/LFU, how a cache decides what to drop
- [ ] **Distributed cache** — Redis Cluster style, why it beats a single-node cache

#### Scale & resilience
- [ ] **Rate limiting algorithms** — token bucket, leaky bucket, sliding window
- [ ] **Auto-scaling** — metric-driven scale-out/scale-in
- [ ] **Disaster recovery** — backups, geo-replication, RPO vs. RTO
- [ ] **Clustering** — running multiple process instances, load distribution
- [ ] **Data privacy & protection in design** — encryption at rest/in transit, RBAC/ABAC, masking, audit logs, compliance

#### Problem-specific techniques
- [ ] **Fan-out on write vs. fan-out on read** — the core feed-design tradeoff (cheap for normal users vs. the celebrity problem)
- [ ] **Geospatial indexing** — quadtrees / geohash for "find nearby" problems
- [ ] **Conflict resolution** — Operational Transformation (OT) vs. CRDTs for collaborative editing
- [ ] **Adaptive bitrate streaming** — transcoding to multiple renditions, HLS/MPEG-DASH

#### Practical / troubleshooting
- [ ] **Optimizing a slow endpoint** — profiling, caching, query tuning, N+1, pagination
- [ ] **Handling DB/API bottlenecks** — indexing, connection pooling, caching, async processing
- [ ] **Dev vs. prod environments** — config, debugging, feature flags, why "works on my machine" happens
- [ ] **Cross-browser / cross-device debugging** — feature detection, dev tools, responsive testing
- [ ] **JIT vs. AOT compilation** — just-in-time vs. ahead-of-time, tradeoffs (Angular, JVM, V8)
- [ ] **Choosing frameworks** — React vs. Angular, Node vs. Laravel, SQL vs. NoSQL (including speed/perf tradeoffs) in a design context

#### Diagramming & LLD
- [ ] **UML diagrams** — class, sequence, use case, activity, state (LLD-flavoured rounds)
- [ ] **Drawing an ERD** — entities, relationships, cardinality, keys
- [ ] **Modular design, tight vs. loose coupling** — the LLD quality bar (cross-ref §2)
- [ ] **Whiteboarding practice** — sketching architecture with a mouse is harder than it sounds; practise on Excalidraw

---

### 5.4 — Practice Designs (ordered by how often they're asked)

Work through these *using the framework in 5.1*, not by memorising solutions. Concepts transfer heavily between them — after ~5, you should be able to attempt a new one cold.

| # | System | Core concepts it tests |
|---|---|---|
| 1 | **URL shortener (bit.ly)** | Hashing/Base62, NoSQL for lookups, caching hot URLs, custom-alias collisions |
| 2 | **Rate limiter** | Token bucket, distributed counting with Redis, 429 responses |
| 3 | **Chat system (WhatsApp/Slack/Telegram)** | WebSockets, message storage, group fan-out via pub/sub, delivery guarantees |
| 4 | **News feed / timeline (Twitter/Facebook)** | Fan-out on write vs. read, ranking, hot-feed caching |
| 5 | **Notification system (push/SMS/email)** | Queue ingestion, channel handlers, templates, retries, delivery tracking |
| 6 | **Video streaming (YouTube/Netflix)** | See the worked walkthrough in 5.5 |
| 7 | **E-commerce cart/checkout** | Inventory locking, saga/2PC for cart+payment+inventory consistency, async fulfilment |
| 8 | **File sharing (Dropbox/Google Drive)** | Chunking, dedup, metadata vs. blob storage, sync |
| 9 | **Ride sharing (Uber/Lyft)** | Geospatial indexing, matching service, surge, shard by city |
| 10 | **Search autocomplete** | Trie/inverted index, prefix caching, fuzzy matching |
| 11 | **Payment system (Stripe/PayPal)** | Idempotency keys, fraud detection, PCI compliance, retries |
| 12 | **Recommendation system (Netflix/Amazon)** | Collaborative filtering, batch vs. real-time layers |
| 13 | **Web crawler (Google)** | Frontier queue, politeness, dedup |
| 14 | **Collaborative doc editing (Google Docs)** | OT/CRDT, WebSockets, pub/sub |
| 15 | **Parking lot / elevator / BookMyShow** | The classic **LLD** prompts (classes, relationships, state) |

Progress: `[ ]` 1 &nbsp; `[ ]` 2 &nbsp; `[ ]` 3 &nbsp; `[ ]` 4 &nbsp; `[ ]` 5 &nbsp; `[ ]` 6 &nbsp; `[ ]` 7 &nbsp; `[ ]` 8 &nbsp; `[ ]` 9 &nbsp; `[ ]` 10 &nbsp; `[ ]` 11 &nbsp; `[ ]` 12 &nbsp; `[ ]` 13 &nbsp; `[ ]` 14 &nbsp; `[ ]` 15

---

### 5.5 — Worked Walkthrough: "Design YouTube"

> Use this as the template for reasoning about *any* large system from scratch. The point isn't the answer — it's the sequence of questions you ask yourself.

#### Step 1 — Clarify & scope *(~5 min)*

Don't design all of YouTube. Ask what matters:

- **Which features?** Upload + watch is the core. Comments, recommendations, live streaming, monetisation, subscriptions — confirm what's in scope and *explicitly park the rest*.
- **Who are the users, and where?** Global users → CDN is non-negotiable.

| Requirement type | What you land on |
|---|---|
| **Functional** | Upload a video, transcode it, stream it, search it |
| **Non-functional** | Massively **read-heavy** (views ≫ uploads, easily 100:1+) · **availability over strict consistency** — a new video appearing seconds late is fine, so this is an **AP** system · smooth playback (buffering is the real failure mode) · durability (never lose an uploaded video) |

#### Step 2 — Estimate *(~2 min)*

Put numbers on it:

| Metric | Rough figure |
|---|---|
| DAU | ~100M |
| Views | ~5/user/day → ~500M/day → **~6K views/sec** average, peak 3–5× |
| Uploads | ~500K/day |
| Storage | ~100MB raw/video, ×3–5 for renditions (240p→4K) → **petabyte-scale/year** |

**These numbers justify every later choice** — they're why you need object storage and a CDN, not a bigger database.

#### Step 3 — Data model & API *(~8 min)*

- **Entities:** User, Video (metadata: id, title, uploader, duration, status, rendition URLs), Comment, View/Watch event.

> **🔑 Key insight:** *video metadata and video bytes are completely different storage problems.* Metadata is small, queried, relational-ish → SQL or a sharded NoSQL store. Video files are huge blobs → **object storage (S3-style)**, never the database.

- **APIs:**
  - `POST /videos` — initiate upload, returns a pre-signed URL
  - `GET /videos/{id}` — metadata
  - `GET /videos/{id}/stream` — manifest
  - `GET /search?q=` — search
  - Add pagination on listings.
- **DB choice, justified:** metadata needs some querying but scales huge → sharded by `video_id`. Watch events are an append-only firehose → NoSQL/Cassandra or a stream, **not** your transactional DB.

#### Step 4 — High-level design *(~15 min)*

Walk the two flows separately — this is the heart of the answer.

**Upload (write path):**
1. Client uploads to object storage **directly** via pre-signed URL — don't proxy 100MB through your API servers.
2. Upload completion publishes an event to a **queue**.
3. **Transcoding workers** consume it — the classic async justification: transcoding takes minutes, so it must never block the request. Split the video into chunks, transcode in parallel to multiple resolutions/bitrates, write renditions back to object storage.
4. Update metadata status to `ready`, push renditions to the CDN.

**Watch (read path):**
1. Request → **CDN edge**. Most requests never reach your origin at all — this is the single most important design decision.
2. On miss → origin/object storage, then cache at the edge.
3. Metadata served from a cache (Redis) in front of the DB, since hot videos are read constantly.
4. Client uses **adaptive bitrate** (HLS/DASH) to switch renditions as bandwidth changes.

#### Step 5 — Bottlenecks & reliability *(~5 min)*

| Concern | Answer |
|---|---|
| **Bottleneck: bandwidth/egress** | The CDN; quantify the hit ratio you need |
| **Bottleneck: transcoding backlog** | Autoscale workers off queue depth; degrade by publishing low-res first, higher renditions later |
| **Celebrity/viral problem** | One video at 10M views/hour: CDN absorbs it, but watch for cache stampede on first publish → pre-warm |
| **Single points of failure** | Replicate metadata DB, multi-region object storage |
| **Graceful degradation** | Recommendations down? Still serve the video. Comments down? Still play. |
| **Monitoring** | p99 startup latency, rebuffer ratio, transcode queue depth, CDN hit ratio |

#### Step 6 — Trade-offs *(~5 min)*

Say them out loud:
- Chose **eventual consistency** (video appears after processing delay) to get availability and scale.
- Chose **object storage + CDN** over any DB-centric approach because of blob size.
- Chose **async transcoding**, accepting that a video isn't instantly watchable.

If asked to extend: live streaming (totally different — low-latency path, no batch transcode), recommendations, DRM.

#### 🎯 Transfer this shape

Every one of these systems is *some* combination of the same moves:

> split the write path from the read path → push heavy bytes to object storage + CDN → make slow work async behind a queue → cache the hot reads → shard the big table → name what you'd sacrifice.

| System | How it maps |
|---|---|
| **Netflix** | ≈ YouTube with less upload |
| **Instagram** | ≈ same, but images + feed fan-out |
| **Dropbox** | ≈ same, but sync + dedup instead of transcode |

---

## 6. Networking, OS & Linux

### Must Know
- [ ] **HTTP** — methods, status codes, headers
- [ ] **TCP vs. UDP** — reliability vs. speed tradeoff
- [ ] **DNS resolution flow**
- [ ] **OSI model layers**
- [ ] **HTTPS/TLS basics** — what the handshake accomplishes
- [ ] **Processes vs. Threads**
- [ ] **Concurrency vs. Parallelism**
- [ ] **Thread lifecycle & states** — new/runnable/running/blocked/terminated
- [ ] **Race conditions & synchronization** — why shared state needs protection; lock the smallest critical section
- [ ] **Deadlock** — the conditions that cause it, and avoidance (consistent lock ordering)
- [ ] **Thread pools** — reusable worker threads, why they beat thread-per-request at scale
- [ ] **Basic Linux commands & file permissions** — chmod/chown, navigation

### Good to Know
- [ ] **TCP three-way handshake** — in detail
- [ ] **HTTP/1.1 vs. HTTP/2 vs. HTTP/3** — what changed and why
- [ ] **WebSockets protocol details**
- [ ] **Sockets & ports**
- [ ] **Load balancer types** — L4 vs. L7
- [ ] **Linux process management** — ps, top, kill
- [ ] **Shell scripting basics**
- [ ] **Cron jobs / task scheduling**
- [ ] **Inter-process communication** — pipes, signals, message queues
- [ ] **OS memory management** — virtual memory, paging
- [ ] **Classic concurrency problems** — producer-consumer, readers-writers, dining philosophers (awareness level)
- [ ] **Busy spinning / busy waiting** — what it is and why it's usually wrong

---

## 7. Security

### Must Know
- [ ] **OWASP Top 10:2025** — the current list, finalized January 2026 (first update since 2021)

  | # | Category | What it means |
  |---|---|---|
  | A01 | Broken Access Control | Enforcement of who's allowed to do what fails — users can view/edit others' data or hit admin routes without authorization. Now also absorbs SSRF. |
  | A02 | Security Misconfiguration | Insecure defaults, unnecessary features left on, exposed cloud storage, default credentials. Jumped from #5 to #2. |
  | A03 | Software Supply Chain Failures | **New category.** Risk from third-party dependencies and build/deploy pipelines. |
  | A04 | Cryptographic Failures | Weak, missing, or misused encryption exposing data in transit or at rest. |
  | A05 | Injection | Untrusted input executed as code/commands (SQL injection, etc.). |
  | A06 | Insecure Design | Flaws in architecture/logic itself, not implementation bugs. |
  | A07 | Authentication Failures | Broken login/session/identity mechanisms. |
  | A08 | Software or Data Integrity Failures | Trusting updates/pipelines/serialized data without verifying integrity. |
  | A09 | Security Logging & Alerting Failures | Missing logs AND missing alerts on suspicious events. |
  | A10 | Mishandling of Exceptional Conditions | **New category.** Poor error handling exploited on unexpected states. |

- [ ] **Authentication vs. Authorization** — the distinction, clearly; how to authenticate a user; handling different authorization levels
- [ ] **Sessions vs. JWT vs. OAuth2** — how each works, tradeoffs
- [ ] **Token vs. JWT** — opaque token vs. self-contained signed token
- [ ] **JWT contents & decoding** — header/payload/signature, base64 (not encryption), how it's verified; advantages/disadvantages
- [ ] **Access token vs. refresh token** — short-lived vs. long-lived, the refresh flow
- [ ] **Hash vs. encryption vs. encoding** — one-way vs. reversible-with-key vs. reversible-no-key; types of hashing
- [ ] **Login flow & where to save tokens** — localStorage vs. cookies (httpOnly), the XSS/CSRF tradeoff
- [ ] **Local storage vs. session storage vs. cookies** — persistence, scope, security
- [ ] **Cookie types & attributes** — cookies are key-value pairs the browser sends in HTTP headers; behavior is set by attributes:
  - **Session cookie** — no Expires/Max-Age; lives only while the browser tab is open
  - **Persistent cookie** — has explicit Expires or Max-Age; survives browser restarts
  - **Secure** — only sent over HTTPS, never plain HTTP
  - **HttpOnly** — inaccessible to JavaScript (`document.cookie`); mitigates XSS cookie theft
  - **SameSite** — controls cross-site sending (`Strict` / `Lax` / `None`); mitigates CSRF
  - **Third-party cookie** — set by a different domain, used for tracking; being phased out by modern browsers
- [ ] **XSS (Cross-Site Scripting)** — what it is, how to prevent it
- [ ] **CSRF (Cross-Site Request Forgery)** — what it is, how to prevent it
- [ ] **SQL Injection** — how it works, parameterized queries as the fix
- [ ] **Securing a login app** — the full checklist: hashing, HTTPS, rate limiting, secure token storage, input validation
- [ ] **API versioning** — URL vs. header versioning, why and when
- [ ] **Password hashing** — bcrypt, salting, why plain hashing isn't enough

### Good to Know
- [ ] **CORS in depth** — preflight requests, allowed origins/headers
- [ ] **Content Security Policy (CSP)**
- [ ] **Rate limiting for security** — brute-force protection
- [ ] **Secrets management** — environment variables, vaults
- [ ] **Security headers** — HSTS, X-Frame-Options, etc.
- [ ] **Encryption** — symmetric vs. asymmetric
- [ ] **API security** — API keys, mutual TLS
- [ ] **Webhooks** — what they are, how they differ from polling, securing them
- [ ] **Controlling permissions** — role-based / attribute-based access control
- [ ] **Preventing duplicate payment on double-click** — idempotency keys, debouncing, locking

---

## 8. Programming Language Internals

### JavaScript / TypeScript / Node.js

**Must Know**
- [ ] **Event loop & call stack** — including Node's libuv phases
- [ ] **Closures** — including callbacks that close over variables
- [ ] **Prototypal inheritance** — prototype chain, `prototype` vs. `__proto__`
- [ ] **`this` binding** — rules across contexts
- [ ] **Hoisting** — and the Temporal Dead Zone
- [ ] **`var` vs. `let` vs. `const`** — scope, hoisting, reassignment
- [ ] **Promises & async/await** — states, `Promise.all` vs. `allSettled`, callback hell
- [ ] **`==` vs. `===`** — coercion vs. strict equality; coercion vs. casting
- [ ] **`null` vs. `undefined`** — and undefined vs. not-defined vs. ReferenceError
- [ ] **`for...of` vs. `for...in`** — values vs. keys
- [ ] **Array methods** — `filter` vs. `map` vs. `reduce` vs. `forEach`, and `for` vs. `forEach`
- [ ] **Call vs. Apply vs. Bind** — explicit `this` binding
- [ ] **Higher-order functions & first-class functions**
- [ ] **Scope** — lexical scope, scope chain, function vs. block scope
- [ ] **TypeScript structural typing** — how it sits on top of JS

**Good to Know**
- [ ] **Spread vs. Rest** — same `...` syntax, opposite jobs
- [ ] **How to clone an object** — spread, `Object.assign`, `structuredClone`, deep vs. shallow
- [ ] **Primitive vs. reference types** — the built-in primitives and copy-by-value behavior
- [ ] **Object.create vs. Object.assign; Object.freeze vs. Object.seal**
- [ ] **Arrow functions & IIFE (self-invoked functions)**
- [ ] **Currying**
- [ ] **Event bubbling / delegation**
- [ ] **Axios vs. Fetch**
- [ ] **Debounce/throttle** — implementation, not just definition
- [ ] **WeakMap/WeakSet**
- [ ] **Generators & iterators**
- [ ] **Node streams & buffers; spawn vs. fork; blocking vs. non-blocking; I/O-bound vs. CPU-bound**
- [ ] **Node multi-threading** — `worker_threads` and `cluster`, when a single-threaded runtime needs them
- [ ] **Using `.env` in Node** — `process.env`, dotenv, keeping config out of code
- [ ] **CommonJS vs. ESM; module.exports vs. exports**
- [ ] **TypeScript: Any vs. Unknown, Type vs. Interface, Union vs. Intersection, `||` vs. `??`**
- [ ] **V8 garbage collection / memory leaks**

### Python

**Must Know**
- [ ] **GIL (Global Interpreter Lock)**
- [ ] **Memory management** — reference counting
- [ ] **Data types** — the built-in types and their categories
- [ ] **List vs. Tuple** — mutability, use cases, performance
- [ ] **Mutable vs. Immutable** — which types are which, and why it matters
- [ ] **Shallow copy vs. Deep copy** — `copy` vs. `deepcopy`, nested-object behavior
- [ ] **`*args` vs. `**kwargs`** — positional vs. keyword variadic arguments
- [ ] **`==` vs. `is`** — value equality vs. identity
- [ ] **Does Python have `===`?** — no, and why (the `==`/`is` answer)
- [ ] **Pass by value vs. reference vs. object** — Python's "pass by object reference" model
- [ ] **Multithreading vs. Multiprocessing vs. Async** — when each helps, tied to the GIL
- [ ] **Generators**
- [ ] **Decorators**
- [ ] **Closures**
- [ ] **List/dict comprehensions**
- [ ] **Exception handling** — try/except/else/finally, `pass` in except, catching specifics
- [ ] **Mutability gotchas** — the mutable default argument trap
- [ ] **Recursion vs. loop** — tradeoffs, Python's recursion limit

**Good to Know**
- [ ] **Object caching / interning** — small-int and string interning behavior
- [ ] **Object chaining & reference sharing** — how assignment shares references
- [ ] **Context managers** — `with` statement mechanics
- [ ] **Static vs. dynamically typed** — where Python sits, type hints
- [ ] **Metaclasses**
- [ ] **Async/await (asyncio)**
- [ ] **Magic/dunder methods**

### Java

> **📘 Mandatory reading:** [Java interview-prep book](https://drive.google.com/file/d/1gBXH-8JFVrMtwY2-ZH6T-S_32Isj1tIC/view?usp=sharing) — work through this alongside the checklist items below.

**Must Know**
- [ ] **JVM architecture**
- [ ] **Stack vs. heap**
- [ ] **Garbage collection basics** — and what causes a memory leak in a GC language
- [ ] **How OOP dispatch works under the hood** — polymorphism mechanics
- [ ] **Abstract class vs. interface** — and interface default methods, functional interfaces
- [ ] **Overriding vs. overloading** — and can you override a static method? (no — hiding, not overriding)
- [ ] **Checked vs. unchecked exceptions** — and `throw` vs. `throws`, try-with-resources
- [ ] **`final` vs. immutable** — a final reference vs. a truly immutable object
- [ ] **StringBuffer vs. StringBuilder** — thread-safety vs. speed
- [ ] **Handling multiple threads** — `Thread`/`Runnable`, `synchronized`, thread safety
- [ ] **`start()` vs. `run()`; `wait()`/`notify()` vs. `sleep()` vs. `join()`** — the classic distinctions (and which release locks)
- [ ] **`volatile` vs. atomic classes** — visibility only vs. visibility + atomicity
- [ ] **Class-level vs. object-level locks** — `static synchronized` vs. `synchronized`
- [ ] **ExecutorService & thread pools** — vs. managing raw threads

**Good to Know**
- [ ] **JIT compilation**
- [ ] **Immutability in Java** — how to build an immutable class
- [ ] **What is ORM** — from the Java/Hibernate perspective
- [ ] **ACID transactions & isolation levels** — from the JDBC/JPA perspective (cross-ref §4)
- [ ] **Interface Segregation example** — a concrete violation-and-fix (cross-ref §2)
- [ ] **Life before OOP** — procedural limitations Java's OOP addressed
- [ ] **Collections framework internals**
- [ ] **Generics & type erasure**

### C / C++ (with Java comparison)

**Must Know**
- [ ] **Manual memory management** — `malloc`/`free`, `new`/`delete` vs. Java's garbage collector
- [ ] **Pointers vs. references**
- [ ] **Compiled-to-machine-code vs. JIT/bytecode** — the control-vs-safety tradeoff

**Good to Know**
- [ ] **Stack vs. heap allocation in C**
- [ ] **Undefined behavior** — the concept and why it matters
- [ ] **RAII (C++)**
- [ ] **Smart pointers (C++)**
- [ ] **Const-correctness**

### Ruby

**Must Know**
- [ ] **Blocks, procs & lambdas** — the differences between them
- [ ] **Everything-is-an-object model** — including numbers and nil
- [ ] **Duck typing** — Ruby's dynamic typing philosophy
- [ ] **Mixins (modules)** — Ruby's alternative to multiple inheritance

**Good to Know**
- [ ] **Metaprogramming** — `method_missing`, `define_method`
- [ ] **Symbols vs. strings** — why symbols exist
- [ ] **Ruby's garbage collector**

### PHP

**Must Know**
- [ ] **Request lifecycle** — superglobals, session handling
- [ ] **Type juggling / loose typing** — common gotchas
- [ ] **Arrays as PHP's core structure** — ordered map behavior
- [ ] **Value vs. reference** — assignment and `&` reference semantics
- [ ] **Magic methods** — `__construct`, `__get`/`__set`, `__call`, `__invoke`

**Good to Know**
- [ ] **Shared-nothing architecture** — how PHP-FPM handles each request
- [ ] **Namespaces & autoloading** — PSR-4
- [ ] **Traits** — PHP's mixin mechanism
- [ ] **Static vs. late static binding** — `self::` vs. `static::`
- [ ] **Closures & `use`** — capturing variables into anonymous functions
- [ ] **Generators / `yield`** — lazy iteration, memory efficiency

---

## 9. Backend Frameworks & APIs

### API Fundamentals (Must Know)
- [ ] **HTTP methods & semantics** — GET/POST/PUT/PATCH/DELETE, when each is correct, PUT vs. PATCH (full vs. partial update)
- [ ] **Status code groups** — 2xx success, 3xx redirect, 4xx client error, 5xx server error, and the common specific codes (200/201/204/400/401/403/404/409/422/500)
- [ ] **Idempotency** — which methods are idempotent and why it matters for retries
- [ ] **Idempotent vs. safe methods** — safe = doesn't change state (GET/HEAD/OPTIONS); idempotent = same result on repeat (GET/PUT/DELETE/HEAD/OPTIONS); POST and PATCH are neither
- [ ] **Request/response structure** — headers, body, query params vs. path params vs. body
- [ ] **Anatomy of an HTTP request/response** — method, URI, HTTP version, headers, body
- [ ] **REST maturity & conventions** — resource naming, statelessness, HATEOAS awareness
- [ ] **Statelessness in REST** — why it makes horizontal scaling easy
- [ ] **URI vs. URL vs. URN** — identifier vs. locator vs. name
- [ ] **URI design best practices** — plural nouns, no verbs, lowercase, hierarchy via `/`, max 2–3 nesting levels
- [ ] **Payload** — what it is, why GET/DELETE don't carry one
- [ ] **REST vs. SOAP** — architectural style vs. protocol; when SOAP still wins (strict contracts, transactions, advanced security)
- [ ] **REST vs. WebSockets** — request/response + stateless vs. full-duplex + stateful
- [ ] **Content negotiation** — `Accept`/`Content-Type` headers, JSON as default
- [ ] **Data serialization** — what serialization/deserialization is, JSON vs. XML, when you'd see each
- [ ] **Testing APIs** — Postman/Swagger; what you'd actually check

### Frameworks (Must Know)
- [ ] **Express middleware & routing**
- [ ] **REST implementation details** — pagination, validation, error handling
- [ ] **JWT auth flow** — end to end
- [ ] **Django vs. Flask vs. FastAPI** — architecture differences, when to use which
- [ ] **Laravel vs. Rails** — convention-over-configuration philosophy
- [ ] **ORM usage patterns** — migrations, relationships, eager vs. lazy loading
- [ ] **Error handling & logging** — structured error responses, centralized error handling, what/how to log (and what never to log)
- [ ] **Environment variables & config management** — `.env` files, per-environment config, keeping secrets out of code

### Good to Know
- [ ] **NestJS** — dependency injection, decorators, modules
- [ ] **GraphQL implementation** — resolvers, schema design
- [ ] **WebSockets implementation** — e.g. socket.io
- [ ] **File upload handling**
- [ ] **Background jobs / task queues** — Celery, Bull
- [ ] **API versioning strategies**
- [ ] **Django specifics** — request lifecycle, `select_related` vs. `prefetch_related` (N+1 fix), serialization, types of views, `makemigrations` vs. `migrate` + migration conflicts, CSRF & CORS, signals, ASGI vs. WSGI, Django vs. Express, `manage.py` commands
- [ ] **Stateful vs. stateless** — session state vs. token-based statelessness
- [ ] **Query params vs. route params vs. body** — where data rides on a request

---

## 10. Frontend

### Must Know
- [ ] **HTML fundamentals** — semantic elements, forms, document structure, accessibility basics; HTML4 vs. HTML5, SEO basics
- [ ] **React hooks** — useState/useEffect/useContext/custom hooks; useEffect dependency array
- [ ] **Library or framework?** — what React is and why use it
- [ ] **useState vs. useRef** — re-render vs. no re-render
- [ ] **useMemo vs. useCallback vs. React.memo** — memoizing values vs. functions vs. components
- [ ] **useContext vs. Redux** — when local context is enough vs. global store
- [ ] **Global state management** — Redux vs. useContext vs. Zustand, when a global store is worth it
- [ ] **useEffect cleanup function** — the return function, why it prevents leaks/stale subscriptions
- [ ] **Optimizing React performance** — memoization, list keys, code splitting, avoiding unnecessary re-renders
- [ ] **State vs. Props** — owned/mutable vs. passed-in/read-only
- [ ] **Props drilling** — the problem and how context solves it
- [ ] **Component lifecycle** — and replicating it with useEffect
- [ ] **JSX** — what it compiles to
- [ ] **Why `key` in lists** — reconciliation and stable identity
- [ ] **Virtual DOM & diffing** — and how React enhances performance
- [ ] **DOM vs. BOM** — document vs. browser object model
- [ ] **State management** — Context API vs. Redux/Zustand
- [ ] **Rendering strategies** — CSR vs. SSR vs. SSG vs. ISR, and hydration
- [ ] **Next.js** — the practical implementation of the above: `getServerSideProps`/`getStaticProps`, App Router, API routes
- [ ] **Core Web Vitals** — FCP, LCP, TTI, TBT, CLS
- [ ] **CSS box model & specificity**
- [ ] **Flex vs. Grid** — one-dimensional vs. two-dimensional layout
- [ ] **px vs. rem vs. em** — absolute vs. relative units
- [ ] **Inline vs. block** — element display behavior
- [ ] **Semantic tags & `alt` attribute** — accessibility and SEO
- [ ] **Script tag placement** — `async`/`defer`, why placement matters
- [ ] **Transition vs. animation** — and CSS performance (inline vs. internal vs. external), lazy loading
- [ ] **Responsive design** — media queries, Flexbox, Grid
- [ ] **Same-Origin Policy & CORS**
- [ ] **DOM manipulation & event handling** — bubbling, delegation

### Good to Know
- [ ] **Content Security Policy (CSP)** — cross-ref Security
- [ ] **Service workers / PWA**
- [ ] **Browser storage** — cookies vs. sessionStorage vs. localStorage
- [ ] **Build tools** — Webpack/Vite, tree shaking, code splitting
- [ ] **Angular vs. Vue vs. React** — comparison
- [ ] **Angular specifics** — directives, pipes, interpolation, lifecycle hooks, JIT vs. AOT, component vs. module, template-driven vs. reactive forms
- [ ] **CSS specificity & justify-content vs. align-items** — layout/cascade fundamentals
- [ ] **Accessibility (a11y)** — ARIA roles, semantic HTML, keyboard navigation
- [ ] **Browser rendering pipeline** — critical rendering path
- [ ] **SEO basics for SPAs**
- [ ] **CSS preprocessors & methodologies** — Sass/SCSS, BEM naming convention
- [ ] **CSS-in-JS** — styled-components/Emotion, tradeoffs vs. utility-first CSS
- [ ] **CSS Modules** — scoped class names, vs. CSS-in-JS vs. Tailwind
- [ ] **Tailwind CSS** — utility-first approach, tradeoffs vs. traditional/component CSS
- [ ] **Bootstrap** — component-library approach, when it's still a reasonable choice
- [ ] **Stacking context & z-index** — what creates a new stacking context
- [ ] **Block Formatting Context (BFC)** — what triggers it, why it matters for layout/float clearing
- [ ] **`position: sticky`** — how it differs from fixed/relative
- [ ] **Browser caching** — cache headers, `ETag`, cache busting
- [ ] **`requestAnimationFrame` vs. `setTimeout`** — for smooth animation
- [ ] **Preload vs. prefetch** — resource-hint differences

---

## 11. DevOps

### Must Know
- [ ] **Docker** — image vs. container, Dockerfile, docker-compose
- [ ] **Dockerfile instructions** — CMD vs. ENTRYPOINT, layering, common directives
- [ ] **CI/CD pipeline stages** — build → test → deploy
- [ ] **CI vs. CD** — continuous integration vs. delivery vs. deployment (the three distinct meanings)
- [ ] **Pipeline/workflow structure** — triggers, secrets, dependency caching, matrix builds
- [ ] **Git workflows** — branching strategies, PR/merge process
- [ ] **AWS core services** — EC2, S3, RDS, Lambda, IAM
- [ ] **Nginx / reverse proxy basics**
- [ ] **Linux troubleshooting** — `chown` vs. `chmod`, why a 405/permission error appears after a bad `chmod`, reading server logs when a server is down
- [ ] **Architecture models** — 3-tier architecture, thin vs. thick client
- [ ] **"Can you run SQL in MongoDB?"** — the relational-vs-document answer and why the question is a trap

### Good to Know
- [ ] **Docker deep-dive** — multi-stage builds, layer caching, volumes vs. bind mounts, networking, secrets, health checks
- [ ] **Zero-downtime migrations** — the expand/contract pattern
- [ ] **Deployment strategies** — rolling vs. blue-green vs. canary
- [ ] **Process managers** — PM2/systemd/supervisor for keeping services alive
- [ ] **Kubernetes basics** — pods, services, deployments
- [ ] **Infrastructure as Code** — Terraform concept
- [ ] **Monitoring & logging** — Prometheus/Grafana/ELK stack; logging vs. monitoring distinction
- [ ] **Load balancer configuration**
- [ ] **Container orchestration concepts**

---

## 12. AI/ML, Data Science & Data Analysis

### Must Know
- [ ] **Data analysis workflow** — EDA, cleaning, visualization
- [ ] **Core statistics** — mean/median/std dev/correlation
- [ ] **Supervised vs. unsupervised learning**
- [ ] **Common ML algorithms, conceptually** — regression, classification, clustering
- [ ] **What RAG is and why it exists**
- [ ] **Vector databases & embeddings** — concept level (Pinecone/Chroma/pgvector)
- [ ] **Prompt engineering** — zero/few-shot, system prompts, structured output
- [ ] **Open-source models** — awareness of STT (speech-to-text), TTS (text-to-speech), image classification model categories

### Good to Know
- [ ] **LangChain** — chains, agents, orchestration
- [ ] **n8n / workflow automation** — where it fits vs. custom code
- [ ] **Overfitting vs. underfitting**
- [ ] **Train/test/validation split**
- [ ] **Model evaluation metrics** — precision/recall/F1
- [ ] **Pandas/Numpy operations** — the ones you should be able to name
- [ ] **Neural network basics**

---

## 13. Testing

### Must Know
- [ ] **Testing pyramid** — unit / integration / E2E
- [ ] **Mocking & stubbing basics**
- [ ] **JUnit** — Java: `@Test`, `@Before`/`@After`, assertions
- [ ] **Jest/Mocha** — JS/Node
- [ ] **PyTest** — Python

### Good to Know
- [ ] **Test-Driven Development (TDD)**
- [ ] **Code coverage** — what it does and doesn't tell you
- [ ] **E2E tools** — Cypress, Selenium, Playwright
- [ ] **Load/performance testing basics**
- [ ] **Contract testing**

---

## 14. Git & Version Control

### Must Know
- [ ] **Core commands** — clone/add/commit/push/pull
- [ ] **Branching & merging**
- [ ] **Branches — what they are & when to create one** — feature/release/hotfix workflow reasoning
- [ ] **Reset vs. revert** — rewriting history vs. a new undo commit
- [ ] **Git revert to any commit** — undoing a specific past commit safely on shared history
- [ ] **Fetch vs. pull** — download only vs. download + merge
- [ ] **Git stash** — and stash apply vs. pop
- [ ] **Two ways to undo changes** — reset/revert/checkout/restore tradeoffs
- [ ] **Resolving merge conflicts**
- [ ] **`.gitignore`** — what it does, common patterns
- [ ] **Pull request workflow**
- [ ] **Rebase vs. merge** — linear history vs. merge commit

### Good to Know
- [ ] **Branch not found locally / can't checkout to main** — fetching remotes, tracking branches
- [ ] **Interactive rebase** — `rebase -i` for squashing/reordering/editing commits
- [ ] **Branching strategies** — Git Flow vs. Trunk-Based vs. GitHub Flow
- [ ] **Git hooks**
- [ ] **Cherry-picking**
- [ ] **Git bisect**

---

## 15. Behavioral & CS Trivia

### Must Know
- [ ] **GP Review (daily habit)** — before starting each new day's topic, spend 15–20 min recapping the previous day: skim your notes, re-explain one concept out loud, redo one problem from memory.
- [ ] **STAR format** — Situation, Task, Action, Result
- [ ] **Introduce yourself / walk through a project / a problem you faced** — the three near-universal openers, rehearsed
- [ ] **5–6 prepared stories** from your real projects (OasisFund, ITI Grading System, WyrmHole) covering: a technical challenge, a disagreement, a mistake you learned from, a time you took initiative
- [ ] **Common scenario questions** — where to place logic (backend vs. frontend), explaining atomicity through a real feature, handling conflict with a colleague, pushing an idea the lead disagrees with, being asked to work during vacation
- [ ] **Practical tip** — have demo credentials ready when asked to show a project live
- [ ] **Warm-up coding problems** — rehearse the approach out loud, not just the code:
  - Best Time to Buy and Sell Stock
  - Maximum Altitude (running prefix sum)
  - Two Sum
  - Reverse a string
  - Convert an array to an object
  - Remove duplicates from an array / find unique elements
  - Character count / frequency map
  - Print prime numbers from an array
- [ ] **SDLC phases**
- [ ] **SDLC models** — Waterfall, Incremental, Spiral, Agile; when each fits and their drawbacks
- [ ] **Agile principles**
- [ ] **Functional vs. non-functional requirements** — cross-ref §5; who specifies each

### Good to Know
- [ ] **Testing terminology** — alpha vs. beta, black-box vs. white-box
- [ ] **Cohesion vs. coupling** — within-module vs. between-module; cross-ref OOP section
- [ ] **Verification vs. validation** — building it right vs. building the right thing
- [ ] **QA vs. QC** — preventive/process vs. corrective/product
- [ ] **SRS (Software Requirements Specification)** — what it is and why it exists
- [ ] **Modularization** — breaking a system into independent modules
- [ ] **Data Flow Diagram (DFD)** — modelling how data moves through a system
- [ ] **Feasibility study** — economic/technical/operational/legal/schedule
- [ ] **Prototyping vs. POC** — a limited working model vs. validating an idea is possible at all
- [ ] **Software Configuration Management (SCM)** — controlled change management
- [ ] **Software metrics** — what they measure and why
- [ ] **Characteristics of software** — functionality, efficiency, reliability, usability, maintainability, portability
- [ ] **Baseline & software re-engineering** — milestone reference points; modernising existing software
- [ ] **Software estimation techniques** — awareness only (COCOMO, function points)
- [ ] **Pair programming** — driver/navigator roles

### Interview Tips (Delivery)
- [ ] **Show your thinking** — narrate your reasoning out loud so the interviewer follows your process, not just your final answer.
- [ ] **Always give an example** — after every answer, ground it with a concrete example. Prepare situations and test cases that can slot into any topic.
- [ ] **Steer toward your strengths** — guide the conversation toward the areas you know best.
- [ ] **Diagram your project** — if asked to explain a project idea, sketch it rather than only describing it.
- [ ] **Over-deliver** — always try to give a little more than what was asked.
- [ ] **Use STAR** — answer behavioral/experience questions in Situation → Task → Action → Result form.

---

## Backend Framework Comparison (Day 7 — Concept-First)

Worst-case prep: how **all seven** frameworks handle the same core concepts. Read each table down by concept. Frameworks: **Express, NestJS, Django, Flask, FastAPI, Laravel, Rails.**

**One-line identity:**
- **Express** — minimal, unopinionated Node HTTP layer.
- **NestJS** — opinionated TypeScript framework (modules + DI) on top of Express/Fastify.
- **Django** — batteries-included Python; ORM, admin, auth built in.
- **Flask** — minimal Python microframework; add extensions as needed.
- **FastAPI** — modern async Python; type hints drive validation + auto docs.
- **Laravel** — batteries-included PHP; large built-in feature set.
- **Rails** — batteries-included Ruby; convention-over-configuration taken furthest.

### Routing
| Framework | How routing works |
|---|---|
| Express | Imperative — `app.get('/path', handler)` |
| NestJS | Decorator-based on controllers — `@Get()`, `@Post()` |
| Django | Central `urls.py`; `include()` for app-level routing |
| Flask | Decorator on the view — `@app.route('/path')` |
| FastAPI | Decorator with typed params — `@app.get('/path')` |
| Laravel | Route files; `Route::get()`, resource routes |
| Rails | `config/routes.rb` with `resources :x` |

### Middleware / Request Pipeline
| Framework | Mechanism |
|---|---|
| Express | `app.use()`, ordered chain, `next()` |
| NestJS | Middleware → Guards → Interceptors → Pipes → handler |
| Django | Middleware classes in `MIDDLEWARE` setting |
| Flask | `before_request`/`after_request` hooks |
| FastAPI | Middleware + dependency injection |
| Laravel | Middleware classes, global or per-route |
| Rails | Rack middleware + `before_action`/`after_action` |

### Dependency Injection
| Framework | Approach |
|---|---|
| Express | None built in |
| NestJS | First-class DI container (constructor injection) |
| Django | No formal DI; direct imports |
| Flask | No formal DI; app context |
| FastAPI | Central — `Depends()` |
| Laravel | Service container, automatic constructor injection |
| Rails | No formal DI; convention + Ruby flexibility |

### ORM / ODM & Data Access
| Framework | Default data layer |
|---|---|
| Express | None — Sequelize/Prisma/TypeORM/Mongoose |
| NestJS | None built in; integrations for TypeORM/Prisma/Mongoose |
| Django | Django ORM (built in) |
| Flask | None — usually Flask-SQLAlchemy |
| FastAPI | None — SQLAlchemy/SQLModel/Tortoise |
| Laravel | Eloquent (Active Record) |
| Rails | Active Record |

### Authentication
| Framework | Built-in support |
|---|---|
| Express | None — Passport.js / custom JWT |
| NestJS | Passport + Guards |
| Django | Full built-in auth; DRF adds token/JWT |
| Flask | None core — Flask-Login / Flask-JWT |
| FastAPI | OAuth2 helpers; assemble the flow |
| Laravel | Sanctum / Passport / Breeze |
| Rails | Devise gem; `has_secure_password` built in |

### Validation
| Framework | Mechanism |
|---|---|
| Express | None — express-validator / Joi / Zod |
| NestJS | Pipes + class-validator on DTOs |
| Django | Form/serializer validation (DRF serializers) |
| Flask | None core — Marshmallow / WTForms |
| FastAPI | Pydantic models (from type hints) |
| Laravel | Form Request classes |
| Rails | Model-level validations |

### Configuration
| Framework | Config approach |
|---|---|
| Express | Manual — dotenv + own module |
| NestJS | `@nestjs/config`, typed |
| Django | Central `settings.py` |
| Flask | `app.config` object |
| FastAPI | Pydantic `BaseSettings` (typed) |
| Laravel | `config/*.php` reading `.env` |
| Rails | `config/` + encrypted `credentials.yml.enc` |

### Error Handling
| Framework | Mechanism |
|---|---|
| Express | 4-arg error middleware `(err, req, res, next)` |
| NestJS | Exception filters + `HttpException` |
| Django | Exception middleware; DRF structured handling |
| Flask | `@app.errorhandler(code)` |
| FastAPI | `HTTPException` + handlers |
| Laravel | Central `Handler` class |
| Rails | `rescue_from` in controllers |

### Choosing One Over Another
| Need | Lean toward |
|---|---|
| Maximum control, minimal opinion | Express, Flask |
| Structure & scale in a Node/TS team | NestJS |
| Ship a full product fast, small team | Django, Laravel, Rails |
| Modern async API + auto docs + typing | FastAPI |
| Data-heavy admin/CRUD app | Django |
| PHP shop / rich ecosystem | Laravel |
| Convention over configuration, rapid CRUD | Rails |

**Mental model:** minimal/unopinionated (Express, Flask) → modern middle (NestJS, FastAPI) → batteries-included/opinionated (Django, Laravel, Rails). Place any unfamiliar framework on that spectrum and reason from these concept axes.

---

## Frontend Framework Comparison (Day 8 — Concept-First)

Same approach for the frontend. Read each table down by concept. Frameworks: **React, Angular, Vue** (+ their meta-frameworks **Next.js / Nuxt** where relevant).

**One-line identity:**
- **React** — a library, not a framework; you assemble routing/state/etc. from the ecosystem.
- **Angular** — full opinionated framework; batteries included (DI, router, forms, HTTP).
- **Vue** — progressive framework; approachable core, official libraries for the rest.

### Component Model
| Framework | Approach |
|---|---|
| React | Function components + JSX |
| Angular | Classes + decorators + HTML templates |
| Vue | Single File Components (`.vue`: template/script/style) |

### State (Local)
| Framework | Mechanism |
|---|---|
| React | `useState` / `useReducer` |
| Angular | Class properties |
| Vue | `ref` / `reactive` |

### State (Global)
| Framework | Options |
|---|---|
| React | Context API, Redux, Zustand |
| Angular | Services + RxJS, NgRx |
| Vue | Pinia (Vuex predecessor) |

### Reactivity Model
| Framework | How updates propagate |
|---|---|
| React | Manual re-render via state setters; virtual DOM diffing |
| Angular | Zone.js change detection (or `OnPush`) |
| Vue | Automatic dependency-tracked reactivity |

### Data Binding
| Framework | Mechanism |
|---|---|
| React | One-way; explicit `onChange` handlers |
| Angular | Two-way `[(ngModel)]`, property `[ ]`, event `( )` |
| Vue | `v-model` two-way; `v-bind` / `v-on` |

### Templating
| Framework | Syntax |
|---|---|
| React | JSX (JS-in-markup) |
| Angular | HTML templates + directives (`*ngIf`, `*ngFor`) |
| Vue | HTML templates + directives (`v-if`, `v-for`) |

### Routing
| Framework | Solution |
|---|---|
| React | React Router (external) |
| Angular | Built-in `RouterModule` |
| Vue | Vue Router (official) |

### Dependency Injection
| Framework | Approach |
|---|---|
| React | None — props / context / hooks |
| Angular | First-class DI system |
| Vue | `provide`/`inject` (lightweight) |

### HTTP / Data Fetching
| Framework | Typical approach |
|---|---|
| React | `fetch`/axios in effects; React Query/SWR |
| Angular | Built-in `HttpClient` + RxJS |
| Vue | `fetch`/axios; often with Pinia |

### Forms
| Framework | Mechanism |
|---|---|
| React | Controlled/uncontrolled inputs; libraries (React Hook Form) |
| Angular | Reactive Forms vs. Template-driven Forms |
| Vue | `v-model` binding; VeeValidate for complex cases |

### SSR / Meta-Framework
| Framework | Meta-framework |
|---|---|
| React | Next.js |
| Angular | Angular Universal (built-in SSR) |
| Vue | Nuxt |

### Choosing One Over Another
| Need | Lean toward |
|---|---|
| Flexibility, huge ecosystem, hiring pool | React |
| Large enterprise app, structure enforced | Angular |
| Gentle learning curve, progressive adoption | Vue |

**Mental model:** library-you-assemble (React) → progressive middle (Vue) → full opinionated framework (Angular). Same spectrum logic as backend — place an unfamiliar frontend tool on it and reason from these concept axes.

---

## Using the 2-week interview period (2–3 days per company)

- **T-3 days:** Research their actual tech stack. Re-check the Must Know items for whichever backend/frontend framework they use.
- **T-2 days:** One full mock interview. Re-drill DSA from your weakest Must Know pattern.
- **T-1 day:** Light review only — behavioral stories, questions to ask them, sleep well.

---

## Stack-Specific Revision (T-3 Days)

Split by individual framework — pull up the two relevant to your target company (one backend, one frontend), then check the **Pairing Notes** at the end for how they specifically connect. React/Next.js fundamentals already in §10 aren't repeated here; this adds the implementation-level depth §9/§10 didn't cover.

### Backend Frameworks

#### Django

**Must Know**
- [ ] Django REST Framework — serializers (`ModelSerializer` vs. `Serializer`), ViewSets & Routers, generic views
- [ ] Django ORM — QuerySet laziness, `select_related` vs. `prefetch_related` (the N+1 fix), FK/M2M/O2O relationships
- [ ] Migrations — `makemigrations`/`migrate` workflow, what a migration file contains
- [ ] URL routing — `urls.py`, path converters, `include()` for app-level routing
- [ ] Middleware — the request/response processing chain, where auth/CORS middleware sits
- [ ] Authentication — built-in User model, session auth vs. token auth (djangorestframework-simplejwt)
- [ ] Settings structure — `INSTALLED_APPS`, environment-based config
- [ ] Django forms — `ModelForm`, validation, `clean()` methods

**Good to Know**
- [ ] Django admin — auto-generated CRUD, customizing `ModelAdmin`
- [ ] Signals — `pre_save`/`post_save`, why they can complicate debugging
- [ ] DRF permissions & throttling classes
- [ ] Django Channels — WebSockets/async support
- [ ] Celery — background task integration
- [ ] Class-based vs. function-based views — tradeoffs
- [ ] Django's built-in caching framework

#### Laravel

**Must Know**
- [ ] Eloquent ORM — relationships (`hasMany`, `belongsTo`, `belongsToMany`), eager loading with `with()` to avoid N+1
- [ ] Migrations & seeders — schema versioning, `php artisan migrate`
- [ ] Routing & controllers — `routes/web.php` vs. `routes/api.php`, resource controllers
- [ ] API Resources — transforming models into JSON responses
- [ ] Sanctum/Passport — token-based API auth, SPA authentication
- [ ] Middleware — `auth`, `throttle`, registering custom middleware
- [ ] Service container & dependency injection — Laravel's IoC container

**Good to Know**
- [ ] Artisan CLI — `make:model`, `make:controller`, `tinker`
- [ ] Queues & jobs — dispatching background work
- [ ] Events & listeners — decoupled side effects
- [ ] Form Request validation classes
- [ ] Query builder vs. raw Eloquent
- [ ] Blade templating basics (if the company isn't API-only)
- [ ] Facades — the static-proxy pattern and how they resolve to the container
- [ ] Accessors & mutators / attribute casting — transforming model attributes
- [ ] Query scopes — reusable local/global query constraints
- [ ] Mass assignment — `$fillable` vs. `$guarded`, why it matters for security
- [ ] Model events & observers — hooking into create/update/delete
- [ ] Caching — `remember()`, cache tags, drivers; config/route caching
- [ ] Task scheduling — the scheduler vs. cron
- [ ] Testing — Feature vs. Unit tests, `RefreshDatabase` trait

#### Rails + ERB / Hotwire

**Must Know**
- [ ] MVC conventions — convention over configuration, Rails' file/folder structure
- [ ] Active Record — associations (`has_many`, `belongs_to`, `has_and_belongs_to_many`), validations, callbacks
- [ ] Migrations — `rails generate migration`, `schema.rb`
- [ ] ERB templating — `<%= %>` vs. `<% %>`, partials (`render partial:`), layouts
- [ ] Routing — RESTful `resources`, nested routes
- [ ] Form helpers — `form_with`, strong parameters (`params.require().permit()`)
- [ ] Asset pipeline — Sprockets (older) vs. Propshaft (newer Rails)

**Good to Know**
- [ ] Hotwire (Turbo + Stimulus) — Rails' modern SPA-like interactivity without a JS framework; Turbo Drive/Frames/Streams
- [ ] Action Cable — Rails' WebSockets solution
- [ ] Active Job + Sidekiq — background job processing
- [ ] RSpec/Minitest — Rails testing conventions
- [ ] Full callback lifecycle order — common gotchas
- [ ] Concerns — Rails' module-based code reuse pattern

#### NestJS

**Must Know**
- [ ] Modules, Controllers, Providers — Nest's core building blocks
- [ ] Dependency Injection — how Nest's IoC container resolves providers
- [ ] Decorators — `@Injectable()`, `@Controller()`, `@Get()`/`@Post()`/`@Body()`/`@Param()`
- [ ] Guards — route-level authorization (`CanActivate`)
- [ ] Interceptors — wrapping request/response (logging, transformation)
- [ ] Pipes — validation/transformation of incoming data (`ValidationPipe` + class-validator)
- [ ] Request lifecycle order — Guards → Interceptors → Pipes → Handler

**Good to Know**
- [ ] Built-in microservices support — TCP/Redis/Kafka transport layers
- [ ] Nest + TypeORM/Prisma integration patterns
- [ ] Custom decorators
- [ ] Exception filters — centralized error handling
- [ ] Nest CLI & module generation
- [ ] Monorepo tooling (Nx) — common in larger Nest codebases

#### Node.js / Express

**Must Know**
- [ ] Middleware chain — `app.use()`, ordering, `next()`
- [ ] Routing — route params, query params, route-level middleware
- [ ] Error handling — centralized error-handling middleware (4-arg signature)
- [ ] REST implementation — status codes, pagination, request validation
- [ ] JWT auth flow — issuing, verifying, refresh-token pattern
- [ ] Async error handling — why unhandled promise rejections crash naive Express apps

**Good to Know**
- [ ] Express alternatives awareness — Koa, Fastify, and why someone might pick them
- [ ] Clustering / PM2 — scaling a single-threaded Node process across cores
- [ ] Rate limiting middleware (`express-rate-limit`)
- [ ] File uploads — Multer
- [ ] Environment config layering — dotenv and friends

### Frontend Frameworks

#### React
*(core hooks/virtual-DOM/state-management fundamentals are in §10 — not repeated)*

**Must Know**
- [ ] Component composition patterns — container/presentational split, compound components
- [ ] `useEffect` dependency array gotchas — stale closures, infinite loops
- [ ] Controlled vs. uncontrolled form inputs
- [ ] Key prop in lists — why it matters for reconciliation
- [ ] Rules of Hooks — why hooks can't be conditional/nested
- [ ] Lifting state up & Context pitfalls — prop-drilling fix vs. over-using Context (re-render cost)
- [ ] React Router — nested, dynamic, protected, and lazy routes
- [ ] Server state vs. client state — why React Query/SWR exist separately from Redux

**Good to Know**
- [ ] React Query/SWR — server-state vs. client-state separation
- [ ] Error boundaries
- [ ] `useReducer` for complex local state
- [ ] React DevTools profiling
- [ ] `useLayoutEffect` vs. `useEffect` — timing difference
- [ ] StrictMode double-invoke — why effects run twice in dev
- [ ] Fiber architecture & automatic batching — how React schedules renders
- [ ] Concurrent features — `startTransition`, `useDeferredValue`
- [ ] List virtualization — rendering large lists efficiently
- [ ] Render props & HOCs — older reuse patterns vs. hooks
- [ ] `forwardRef` + `useImperativeHandle` — exposing imperative APIs
- [ ] Portals — rendering outside the parent DOM tree
- [ ] Redux Toolkit vs. Zustand — modern global-state options
- [ ] Testing — React Testing Library philosophy, Jest mocking/spies, writing an interaction test
- [ ] TypeScript in React — typing props/state/hooks/events, generics, `Partial`/`Pick`/`Omit`

#### Vue

**Must Know**
- [ ] Reactivity system — `ref` vs. `reactive`, how Vue tracks dependencies
- [ ] Composition API vs. Options API — structural difference, when each is used
- [ ] Computed properties vs. methods vs. watchers — when to use which
- [ ] Component communication — props down, emits up, `v-model` for two-way binding
- [ ] Directives — `v-if`/`v-for`/`v-bind`/`v-on` shorthand syntax

**Good to Know**
- [ ] Pinia — Vue's current state management (successor to Vuex)
- [ ] Vue Router — navigation guards, dynamic routes
- [ ] Single File Components (`.vue`) — template/script/style structure
- [ ] Nuxt.js — Vue's answer to Next.js (SSR/SSG for Vue)
- [ ] Teleport & Suspense (Vue 3 features)

#### Angular

**Must Know**
- [ ] Architecture — modules (`NgModule`), components, services, dependency injection
- [ ] Data binding — interpolation, property binding `[ ]`, event binding `( )`, two-way binding `[( )]`/`ngModel`
- [ ] RxJS observables — `subscribe`, basic operators (`map`, `filter`, `switchMap`), why Angular leans on RxJS
- [ ] `HttpClient` — API calls, interceptors
- [ ] Routing — `RouterModule`, route guards, lazy-loaded modules
- [ ] Component lifecycle hooks — `ngOnInit`, `ngOnChanges`, `ngOnDestroy`

**Good to Know**
- [ ] Change detection — default (zone.js-driven) vs. `OnPush`
- [ ] NgRx — Redux-style state management for Angular
- [ ] Reactive Forms vs. Template-driven Forms
- [ ] Angular CLI — `ng build`, `ng serve`, schematics
- [ ] Standalone components — modern Angular, moving away from NgModules

#### Next.js
*(rendering-strategy fundamentals are in §10 — not repeated)*

**Must Know**
- [ ] `getServerSideProps` vs. `getStaticProps` vs. client-side fetching — when each runs, what each returns
- [ ] App Router vs. Pages Router — know which one a given codebase uses
- [ ] API routes — building backend endpoints inside a Next.js app
- [ ] Server Components vs. Client Components (App Router) — the `'use client'` boundary

**Good to Know**
- [ ] Incremental Static Regeneration — `revalidate`, on-demand revalidation
- [ ] Middleware — running code before a request completes
- [ ] Image/font optimization built-ins
- [ ] Dynamic imports & code splitting, Next.js-specific patterns

### Pairing Notes — the integration-specific bits that don't belong to either framework alone

- **Django + React:** CORS config (`django-cors-headers`), token auth flow from React's side (storing/refreshing JWTs), separate deployments (Django API + React static build).
- **Laravel + Vue:** confirm whether the company runs API-only (Sanctum + separate Vue SPA) or Inertia.js (server-routed, SPA feel without a separate API) — auth and data flow differ completely between the two.
- **NestJS + Next.js:** shared TypeScript types between backend and frontend (often a shared package in a monorepo); whether Next.js calls the Nest API directly or proxies through its own API routes.
- **Node + Angular:** trace one full request/response cycle — Angular's `HttpClient`+RxJS on the way out, Express middleware chain + JWT verification on the way in.
- **Rails + ERB:** the odd one out — no separate frontend framework, Rails renders HTML directly. Know Hotwire (Turbo + Stimulus) if the company wants SPA-like interactivity without a JS framework — that's modern Rails' answer to "why isn't this React."

---

## Pick your DSA language early

Python, JS, or Java — pick **one** and stick with it. Switching mid-prep slows pattern recognition.

## Key resources referenced

**System design**
- [Tech Interview Handbook — System Design Guide](https://www.techinterviewhandbook.org/system-design/) (interview types, the canonical question list)
- [System Design Newsletter — How to Prepare for System Design Interview](https://newsletter.systemdesign.one/p/how-to-prepare-for-system-design-interview) (the 5-step framework + what interviewers score)
- [A Beginner's Guide to System Design (Medium — Aritra Sen)](https://medium.com/@sentalkssane/a-beginners-guide-to-system-design-76d64689788b) (the template, estimation, practice strategy)
- [roadmap.sh — Top 30 System Design Questions](https://roadmap.sh/questions/system-design) (beginner→advanced, worked designs, scenario framework)
- [GeeksforGeeks — System Design Interview Questions](https://www.geeksforgeeks.org/system-design/top-10-system-design-interview-questions-and-answers/) (HLD/LLD split, 100 questions)
- [InterviewBit — System Design Questions](https://www.interviewbit.com/system-design-interview-questions/) (CAP, performance metrics, sharding)
- [System Design Primer](https://github.com/donnemartin/system-design-primer) (the deepest free reference)
- [roadmap.sh — System Design Roadmap](https://roadmap.sh/system-design)

**General interview prep**
- [Tech Interview Handbook — Algorithms Study Cheatsheet](https://www.techinterviewhandbook.org/algorithms/study-cheatsheet/)
- [Tech Interview Handbook — SWE Interview Guide](https://www.techinterviewhandbook.org/software-engineering-interview-guide/)
- [Grind 75](https://www.techinterviewhandbook.org/grind75/)
- [roadmap.sh — Top 50 Full Stack Interview Questions](https://roadmap.sh/questions/full-stack)
- [roadmap.sh — Top 30 Front End Interview Questions](https://roadmap.sh/questions/frontend)
- [InterviewBit — REST API Questions](https://www.interviewbit.com/rest-api-interview-questions/)
- [InterviewBit — Multithreading Questions](https://www.interviewbit.com/multithreading-interview-questions/)
- [InterviewBit — Full Stack Developer Questions](https://www.interviewbit.com/full-stack-developer-interview-questions/)
- [InterviewBit — Software Engineering Questions](https://www.interviewbit.com/software-engineering-interview-questions/)
- [InterviewBit — Technical Interview Questions](https://www.interviewbit.com/technical-interview-questions/)
- [GeeksforGeeks — Full Stack Developer Interview Questions](https://www.geeksforgeeks.org/html/full-stack-developer-interview-questions-and-answers/)
- [GeeksforGeeks — Software Engineering Interview Questions](https://www.geeksforgeeks.org/software-engineering/software-engineering-interview-questions-and-answers/)
- [backend-cheats](https://github.com/cheatsnake/backend-cheats)
- [Full-stack Developer Interview Questions (indy256)](https://github.com/indy256/Full-stack-Developer-Interview-Questions-and-Answers)
- [Agilemania — Software Developer Interview Questions](https://agilemania.com/software-developer-interview-questions)
- [OWASP Top 10:2025 — official list](https://owasp.org/Top10/2025/) (finalized January 2026)

*DevOps, AI/ML, and Data Science content draws on general industry knowledge rather than the SWE-interview-specific resources above.*
