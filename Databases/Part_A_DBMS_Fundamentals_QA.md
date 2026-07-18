# Part A — DBMS Fundamentals (Interview Q&A)

Covers: DBMS vs RDBMS · Keys · ACID · Isolation Levels · Normalization · OLTP vs OLAP.
Format: each **famous interview question** + detailed answer + keyword to say. Discuss each with me before moving on.

---

# TOPIC 1 — DBMS vs RDBMS + Command Categories
📖 Resources: [DBMS vs RDBMS](https://www.geeksforgeeks.org/dbms/difference-between-rdbms-and-dbms/) · [SQL Command Types](https://www.geeksforgeeks.org/sql/sql-ddl-dql-dml-dcl-tcl-commands/)

### Q1. What is a database and a DBMS?
- **Database** = organized collection of related data.
- **DBMS** = software that stores, retrieves, and manages that data + controls access. It's the middle layer between the app and the raw data.

**answer:** الـ database هو collection منظّم من data. الـ DBMS هو الـ software اللي بيـ store و retrieve و manage الـ access للـ data — زي MySQL و MongoDB. هو الوسيط بين الـ application والـ data.

---

### Q2. DBMS vs RDBMS? ⭐
- **RDBMS** = DBMS that stores data in **tables** with **relationships** (foreign keys) and enforces **integrity constraints**.
- Every RDBMS is a DBMS; not every DBMS is relational (MongoDB = NoSQL DBMS).

```
DBMS
├── Relational (RDBMS) → MySQL, PostgreSQL, Oracle
└── NoSQL → Document (Mongo)
          → Key-Value (Redis)
          → Column (Cassandra)
          → Graph (Neo4j)
```

**answer:** الـ RDBMS هو DBMS بيخزّن الـ data في **tables** بينها **relationships** بالـ foreign keys، وبيفرض **integrity constraints**. كل RDBMS هو DBMS، بس مش كل DBMS يبقى relational — زي MongoDB اللي هو NoSQL.

---

### Q3. SQL command categories?
| Category | Job | Commands |
|----------|-----|----------|
| **DDL** | structure | `CREATE, ALTER, DROP, TRUNCATE` |
| **DML** | data | `INSERT, UPDATE, DELETE` |
| **DQL** | read | `SELECT` |
| **DCL** | permissions | `GRANT, REVOKE` |
| **TCL** | transactions | `COMMIT, ROLLBACK,` **`SAVEPOINT`** |

> `SELECT` = **DQL**, not DML.

**answer:** فيه 5 أنواع: **DDL** للـ structure (CREATE, ALTER, DROP)، **DML** للـ data (INSERT, UPDATE, DELETE)، **DQL** للقراءة (SELECT)، **DCL** للـ permissions (GRANT, REVOKE)، و **TCL** للـ transactions (COMMIT, ROLLBACK, SAVEPOINT).

---

### Q4. Why is TRUNCATE DDL not DML? + which is faster?
- **DELETE (DML)** → removes rows **one by one**, logged (can `ROLLBACK`), supports `WHERE`.
- **TRUNCATE (DDL)** → drops the **data pages** at once, resets the table → works on the **structure**, not rows. No WHERE, no rollback, resets auto-increment.
- **TRUNCATE is faster** — it doesn't log each row.

> 📄 **data page** = fixed-size block (~8KB) the DB reads/writes as one unit; rows live inside pages. DELETE works *inside* pages; TRUNCATE throws the *whole pages* away.

**answer:** الـ **TRUNCATE** بيتعامل مع الـ **structure** — بيعمل deallocate للـ data pages ويصفّر الـ table، مش بيمسح row row، عشان كده هو DDL و**أسرع**. الـ **DELETE** بيمسح الـ rows واحد واحد مع logging، فبيدعم WHERE و rollback بس أبطأ.

---

### Q5. What is a schema?
- **Blueprint** of the DB — tables, columns, types, relationships, constraints.
- **Relational → schema-on-write:** fixed, enforced at write time.
- **NoSQL → schema-on-read:** flexible, shape interpreted at read time; each document can differ.

**answer:** الـ schema هي **blueprint** للـ database. في الـ relational بتبقى **schema-on-write** — ثابتة ومفروضة وقت الكتابة. في الـ NoSQL بتبقى **schema-on-read** — flexible، كل document ممكن يختلف والشكل بيتفسّر وقت القراءة.

---

### Q6. SQL vs NoSQL + use-cases ⭐⭐
| | SQL | NoSQL |
|--|-----|-------|
| Model | Tables | Document / Key-Value / Column / Graph |
| Schema | Fixed (schema-on-write) | Flexible (schema-on-read) |
| Consistency | Strong (**ACID**) | Eventual (**BASE**) |
| Scaling | Vertical | Horizontal |
| Best for | Structured + transactions | Flexible/nested + huge scale |

**Use-cases:** 
- banking/orders → **SQL** 
- product catalog → **Mongo** 
- cache/sessions → **Redis** 
- IoT/time-series → **Cassandra** 
- social graph → **Neo4j**.

**answer:** بستخدم **SQL** لما الـ data structured والـ relationships والـ transactions مهمين — زي banking و orders. وبستخدم **NoSQL** لما الـ schema بيتغيّر أو الـ data nested أو محتاجة scale أفقي — زي product catalog (Mongo) أو caching (Redis). كتير من الأنظمة بتستخدم الاتنين — **polyglot persistence**.

---

### Q7. Vertical vs Horizontal scaling
- **Vertical (scale up)** = stronger machine (more CPU/RAM). Easy, but has a ceiling + single point of failure.
- **Horizontal (scale out)** = more machines. ~Unlimited + fault-tolerant, but harder (load balancer, sharding).
- SQL leans vertical; NoSQL is built for horizontal.

**answer:** الـ **vertical scaling** يعني جهاز أقوى (موارد أكتر) — سهل بس ليه سقف و single point of failure. الـ **horizontal scaling** يعني أجهزة أكتر — شبه لانهائي و fault-tolerant بس أعقد. الـ SQL أميل للـ vertical والـ NoSQL اتصمّمت للـ horizontal.

---

### Q8. Replication vs Sharding? (both are horizontal scaling)
Sharding is **a way to do** horizontal scaling — not a separate thing.
- **Replication** = **full copy** of the data on each server.
  → spreads **reads** + fault tolerance. Every server still holds **all** the data.
- **Sharding** = data is **split**; each server holds a **different part** (chosen by a **shard key**).
  → lets you store data **bigger than one machine** + spreads **writes**.

```
Replication:  Server1[ALL]   Server2[ALL]   Server3[ALL]
Sharding:     Server1[A–H]   Server2[I–P]   Server3[Q–Z]
```

| | Replication | Sharding |
|--|-------------|----------|
| Data per server | full copy | a slice |
| Solves | read scaling + fault tolerance | data too big + write scaling |
| If a server dies | others have everything | that slice is lost (unless also replicated) |

Often combined: each shard has its own replicas.

**answer:** الاتنين نوع من **horizontal scaling**. الـ **replication** = نسخة **كاملة** من الـ data على كل server → بيوزّع الـ **reads** ويدّي fault tolerance. الـ **sharding** = تقسيم الـ data، كل server شايل **جزء** (حسب الـ **shard key**) → بيخليني أخزّن data **أكبر من جهاز واحد** وأوزّع الـ **writes**. غالبًا بيتعملوا مع بعض: كل shard ليه replicas.

---

# TOPIC 2 — Keys
📖 Resource: [Keys in Relational Model](https://www.geeksforgeeks.org/dbms/types-of-keys-in-relational-model-candidate-super-primary-alternate-and-foreign/)

### Q1. What is a Primary Key?
- A column (or set of columns) that **uniquely identifies each row**.
- Rules: **unique + NOT NULL**, only **one per table**.

**answer:** الـ primary key هو column (أو أكتر) بيـ **uniquely identify** كل row. لازم يكون **unique + NOT NULL**، وواحد بس في الـ table.

---

### Q2. Primary Key vs Unique Key? ⭐
| | Primary Key | Unique Key |
|--|-------------|------------|
| Null | **Not allowed** | **Allowed** |
| Count per table | One | Many |
| Purpose | Main identifier | Enforce uniqueness on other columns |

**answer:** الـ **primary key** هو الـ main identifier — واحد بس، ومينفعش NULL. الـ **unique key** بيضمن uniqueness لـ column تاني (زي email)، ممكن يكون فيه كذا واحد في الـ table، ويقبل NULL.

---

### Q3. Can a Primary Key be NULL? Composite?
- **NULL?** No — must identify a row, and NULL = unknown.
- **Composite?** Yes — PK from **multiple columns together**. Ex: `order_items(order_id, product_id)`.

**answer:** لأ، الـ PK مينفعش يكون NULL لأنه لازم يـ identify الـ row. آه ممكن يكون **composite** — يعني متكوّن من أكتر من column مع بعض، زي `order_items(order_id, product_id)`.

---

### Q4. What is a Foreign Key?
- A column that **references another table's Primary Key** → enforces **referential integrity** (can't insert an order for a customer that doesn't exist).
- Ex: `orders.customer_id` → `customers.id`.

**answer:** الـ foreign key هو column بيـ **reference** الـ primary key بتاع table تاني — بيعمل الـ relationship وبيفرض **referential integrity**، يعني مينفعش أدخّل order لـ customer مش موجود.

---

### Q5. Super / Candidate / Primary / Alternate key?

Use this one example table: `users(id, email, name)` where `id` and `email` are each unique.

- **Super key** = **any** column set that identifies a row uniquely — **even if it has extra columns**.
  `{id}`, `{email}`, `{id, email}`, `{id, name}` → all super keys (any set containing something unique).
- **Candidate key** = a super key with **no extra column** (minimal). Remove any column and it stops being unique.
  `{id}`, `{email}` → candidate keys. `{id, email}` is NOT (email is redundant, `id` alone is enough).
- **Primary key** = the candidate key you **chose** → `{id}`.
- **Alternate key** = the candidate keys you **didn't** choose → `{email}`.

> ⚠️ **Two kinds of composite — don't confuse:**
> - `(order_id, product_id)` — neither is unique alone, only **together** → **candidate** key (minimal, you need both).
> - `(id, email)` — `id` is **already** unique alone, email is extra → **super key but NOT candidate**.

**answer:** الـ **super key** أي set بيـ identify الـ row، حتى لو فيه columns زيادة. الـ **candidate key** هو super key **minimal** — من غير أي عمود زيادة (لو شِلت أي عمود يبطّل unique). اللي بختاره منهم يبقى **primary key**، والباقي **alternate**. يعني في `users`: `{id}` و `{email}` candidate keys، بس `{id, email}` super بس مش candidate لأن `id` لوحده كفاية.

---

### Q6. Natural vs Surrogate key — which to prefer?
- **Natural key** — real business data (email, national ID).
- **Surrogate key** — artificial ID with no business meaning (auto-increment, UUID).
- **Prefer surrogate** — business data can change or repeat; a surrogate is stable forever.

**answer:** الـ **natural key** هو business data حقيقي زي email. الـ **surrogate key** هو ID مصطنع ملوش معنى زي auto-increment أو UUID. بفضّل الـ **surrogate** لأن الـ business data ممكن تتغيّر أو تتكرّر، بس الـ surrogate ثابت طول العمر.

---

### Q7. ON DELETE actions?
- **CASCADE** — delete the children too.
- **SET NULL** — set the FK to NULL.
- **RESTRICT / NO ACTION** — block the delete if children exist.

**answer:** لما أمسح parent، الـ **ON DELETE** بيحدّد يحصل إيه للـ children: **CASCADE** يمسحهم معاه، **SET NULL** يخلّي الـ FK بتاعهم NULL، **RESTRICT** يمنع المسح لو فيه children.

---

# TOPIC 3 — ACID + Transactions
📖 Resource: [ACID Properties](https://www.geeksforgeeks.org/dbms/acid-properties-in-dbms/)

### Q1. What is a transaction?
- A **group of operations treated as one unit** — all succeed or none do.
- Classic example: **money transfer** = debit A + credit B, both together.

**answer:** الـ transaction هو مجموعة operations بتتعامل كـ **وحدة واحدة** — يا كلها تنجح يا ولا واحدة. المثال الكلاسيكي: تحويل فلوس = debit من A و credit لـ B، لازم الاتنين مع بعض.

---

### Q2. Explain ACID ⭐⭐
- **A — Atomicity:** all-or-nothing; any step fails → whole thing rolls back.
- **C — Consistency:** DB moves from one **valid state to another** (constraints hold).
- **I — Isolation:** concurrent transactions don't see each other's half-work.
- **D — Durability:** once **committed**, survives crashes (via write-ahead log).

**answer:** الـ **ACID**: **Atomicity** كله أو لا شيء، **Consistency** الـ DB بينتقل من valid state لـ valid state، **Isolation** الـ transactions المتزامنة مبتشوفش شغل بعض النص، **Durability** بعد الـ commit التغيير بيفضل حتى لو حصل crash.

---

### Q3. BASE — the NoSQL counterpart to ACID ⭐⭐ (asked in interviews!)
BASE is the philosophy of **distributed NoSQL** databases — the opposite lean of ACID. It only makes sense with **multiple servers / replicas**.
- **B — Basically Available:** the system **always responds**, even during a partial failure (answer might be slightly stale).
- **S — Soft state:** state **can change over time on its own** as replicas sync in the background.
- **E — Eventual consistency:** if writes stop, all replicas **eventually** agree — not instantly.

**ACID vs BASE:**
| | ACID | BASE |
|--|------|------|
| Question | "Is it **correct right now**?" | "Is it **available**, and correct soon?" |
| Consistency | Strong (immediate) | Eventual |
| Used by | RDBMS (banking, orders) | Distributed NoSQL (feeds, counters) |
| Example | Money transfer | Likes counter |

> **Why BASE exists → replicas.** With one copy, consistency is automatic. With many replicas, you choose: wait for all to agree (strong, slow) or confirm the write now and let replicas catch up in the background (**eventual = BASE**).

**answer:** الـ **BASE** هي فلسفة الـ NoSQL الموزّعة، عكس ACID، وبتشتغل على فكرة الـ **replicas / multi-server**: **Basically Available** بيرد دايمًا حتى لو الإجابة قديمة شوية، **Soft state** الحالة بتتغيّر لوحدها مع تزامن الـ replicas، **Eventual consistency** الـ replicas بتتفق **في النهاية** مش فورًا. البنك عايز **ACID**، بس عدّاد الـ likes مبسوط بـ **BASE**.

---

### Q4. CAP Theorem ⭐ (comes with BASE)
In a **distributed system**, during a **network partition** you can only keep **2 of 3**:
- **C — Consistency:** every read sees the latest write.
- **A — Availability:** every request gets a response.
- **P — Partition tolerance:** system keeps working despite network splits.
  - 🧠 **الميموري:** *Partition = **جدار** بينزل بين السيرفرات، Tolerance = النظام يتحمّله ويفضل شغّال.* الجدار ده **وارد يحصل دايمًا** في distributed (الشبكة بتقع)، فـ **P إجباري** → الاختيار الحقيقي بين **C و A** بس.

> **What's a network partition?** When servers in a distributed system **can't talk to each other** (network link broken) — the servers themselves still work, but the system splits into isolated groups. A write on one side isn't seen on the other, forcing the C-vs-A choice.

Since **P is mandatory** in any distributed system (networks fail), the real choice is **C vs A**:
- **CP** (pick Consistency) → refuses to answer during a split rather than return stale data. Banks.
- **AP** (pick Availability) → always answers, may be stale. Cassandra, DynamoDB → this is **BASE**.

**answer:** الـ **CAP theorem** بيقول في نظام موزّع وقت الـ **network partition** تقدري تختاري **اتنين بس** من: **Consistency** (كل قراءة تشوف آخر كتابة)، **Availability** (كل request بياخد رد)، **Partition tolerance** (يشتغل رغم انقطاع الشبكة). بما إن الـ P لازمة، الاختيار الحقيقي بين **C و A** — **ACID/CP** بيختار consistency، **BASE/AP** بيختار availability.

---

### Q5. Savepoint + auto-commit
- **Savepoint** = a marker inside a transaction; `ROLLBACK TO savepoint` undoes part without killing the whole transaction.
- **Auto-commit** = each statement is its own transaction (committed immediately) unless you explicitly `BEGIN ... COMMIT`.

**answer:**
- الـ **savepoint** هو marker جوّه الـ transaction، أقدر أعمل rollback ليه من غير ما ألغي الـ transaction كله.
- الـ **auto-commit** يعني كل statement transaction لوحده بيتعمله commit فورًا، إلا لو عملت `BEGIN...COMMIT` بنفسي.

---

# TOPIC 4 — Isolation Levels
📖 Resources: [Transaction Isolation Levels (GFG)](https://www.geeksforgeeks.org/dbms/transaction-isolation-levels-dbms/) · 🎓 Udemy: [Hussein Nasser — Fundamentals of Database Engineering](https://www.udemy.com/course/database-engines-crash-course/) (watch only the **ACID** + **Isolation Levels** sections — don't do the whole 22h course now)

> **The big picture (read this first):** when transactions run **at the same time**, they can mess up each other's data → these bugs are called **anomalies**. **Isolation levels** are the DB's settings for **how many anomalies you allow** — higher level = safer but slower. So: **Q1-Q2 = the problems (anomalies)**, **Q3 = the settings (levels)**, **Q4 = solutions**.

### Q1. Why do isolation levels exist?
- Concurrent transactions interfere with each other and cause **anomalies** (wrong reads/writes).
- An isolation level = **how much interference you tolerate** vs how much speed you want. Stronger = safer, slower.

**answer:**
- الـ transactions بتشتغل في نفس الوقت على **shared data**، فبيحصل **تداخل** بيسبّب **anomalies** (wrong reads/writes).
- الـ **isolation level** هو اللي بيحدّد **كام anomaly أسمح بيها** — كل ما أرفع المستوى، أمنع مشاكل أكتر بس أبطأ (locking أكتر).
- فهو **trade-off بين الأمان (consistency) والسرعة (performance)**.

---

### Q2. The read anomalies (the problems) ⭐⭐
Ordered from weakest to worst-hidden:

**1. Dirty read** — read data another transaction wrote but **didn't commit** yet. Danger: it might `ROLLBACK` → you read something that never existed.

**2. Non-repeatable read** — read the **same row** twice, get **different values**, because another transaction **updated + committed** in between. The value is real (committed), but your transaction saw two states.

**3. Phantom read** — a query returning a **set of rows** (`SUM`, `COUNT`, `WHERE price<5`) is run twice; another transaction **inserted/deleted** a matching row in between → a "ghost" row appears/disappears.

**Examples:**
```
DIRTY READ
balance = 100
TX2: UPDATE balance = 500   (no commit yet)
TX1: read balance → 500     ← read an uncommitted value
TX2: ROLLBACK               → balance back to 100
→ TX1 acted on 500 that never existed ❌

NON-REPEATABLE READ
TX1: read balance → 100
TX2: UPDATE balance = 500 → COMMIT
TX1: read balance → 500     ← same row, different value ❌

PHANTOM READ
TX1: SELECT COUNT(*) FROM sales → 2 rows
TX2: INSERT INTO sales (...) → COMMIT
TX1: SELECT COUNT(*) FROM sales → 3 rows   ← a ghost row appeared 👻
```

> Key difference: **dirty** = value **not committed**. **non-repeatable** = existing row's **value changed** (UPDATE). **phantom** = a **new row appeared/vanished** (INSERT/DELETE).

**answer:**
- **dirty read** = أقرا حاجة **لسه متعملهاش commit**، فممكن يحصل rollback وأبقى قريت قيمة ملهاش وجود.
- **non-repeatable read** = أقرا حاجة، حد يعدّل فيها **ويعمل commit**، وأرجع أقراها تاني ألاقي التعديل → قريت **نفس الصف مرتين بقيم مختلفة** (inconsistent).
- **phantom read** = شغّال على query بيرجّع **set of rows**، أقرا مرة، يحصل **insert/delete + commit**، وأقرا تاني يظهر صف جديد أو يختفي صف — زي non-repeatable بس على **range of rows** مش value معيّنة.

---

### Q3. Lost update (a WRITE anomaly) ⭐
Two transactions **read the same value**, each modifies it, each writes → the **last write wins** and the first update is **lost**.

```
balance = 100
TX1: read 100 → +50 → write 150 → COMMIT
TX2: read 100 → +30 → write 130 → COMMIT   (overwrites 150!)
Result: 130  ❌  (should be 180)
```

This is the real-world **"two users buy the last item"** / double-booking problem.

**answer:** الـ **lost update** = اتنين transactions يقروا **نفس القيمة**، كل واحد يعدّل ويكتب، فآخر كتابة **تمسح** تعديل الأولى. زي مشكلة "اتنين يشتروا آخر قطعة في نفس الوقت". الفرق عن الباقي إنه مشكلة في **الكتابة** مش القراءة.

---

### Q4. The 4 isolation levels (the settings) ⭐⭐
Each level **prevents** more anomalies (✅ = can still happen, ❌ = prevented):

| Level | Dirty | Non-repeatable | Phantom |
|-------|:-----:|:--------------:|:-------:|
| **Read Uncommitted** | ✅ | ✅ | ✅ |
| **Read Committed** (default) | ❌ | ✅ | ✅ |
| **Repeatable Read** | ❌ | ❌ | ✅ |
| **Serializable** | ❌ | ❌ | ❌ |

- **How Repeatable Read prevents non-repeatable:** takes a **snapshot** at transaction start, sees only that snapshot throughout.
- **How Serializable prevents phantom:** makes transactions behave **as if they ran one after another (serial)**; the DB detects conflicts and aborts one (→ retry). Strongest + slowest.
- **Default:** Read Committed (Postgres, Oracle, SQL Server). MySQL InnoDB = Repeatable Read.
- Higher level = fewer anomalies = more locking/retries = slower.

**Serializable example — two people withdraw from the same account:**
```
balance = 100

TX1 (Serializable)              TX2 (Serializable)
BEGIN;                          BEGIN;
read balance → 100              read balance → 100     ← both read 100
UPDATE balance = 100-100 = 0
                                UPDATE balance = 100-100 = 0
COMMIT;  ✅ balance = 0
                                COMMIT;  ❌ serialization failure → RETRY
                                (on retry: reads balance = 0 → rejects the withdrawal)
```
The DB aborts TX2 at COMMIT because "both withdraw 100 from 100" is impossible in any serial order. No double withdrawal.
- **Note:** Serializable detects the conflict → **you must write retry logic**. `SELECT ... FOR UPDATE` (pessimistic lock) is the alternative — TX2 just **waits** instead of retrying.

**answer:** الـ isolation levels بتخليني أختار **كام anomaly تحصل** — كل ما أعلى، مشاكل أقل بس أبطأ.
- **Read Uncommitted** = الأضعف، كل الـ anomalies بتظهر.
- **Read Committed** = بيمنع **dirty read** (الـ default).
- **Repeatable Read** = بيمنع **non-repeatable** كمان، عن طريق **snapshot** بياخده أول الـ transaction ويفضل شايفه لحد ما يخلص.
- **Serializable** = الأقوى، بيمنع **الكل بما فيهم phantom** — بيخلّي الـ transactions تشتغل **كأنها بالتتابع (serial)**، بس الأبطأ.

---

### Q5. Solutions — how to prevent lost update ⭐⭐

**1️⃣ Pessimistic Lock (`SELECT ... FOR UPDATE`)** — lock the row upfront; others wait. Best when conflicts are **frequent**.
```sql
SELECT balance FROM accounts WHERE id=1 FOR UPDATE;
UPDATE accounts SET balance = balance - 50 WHERE id=1;
```

**2️⃣ Atomic Update** — DB does the math in one statement, no read-then-write gap. Best for **counters**.
```sql
UPDATE accounts SET balance = balance - 50 WHERE id=1;
```

**3️⃣ Optimistic Lock (version column)** — no lock; check version at write; retry on mismatch. Best when conflicts are **rare**.
```sql
UPDATE accounts SET balance=50, version=version+1
WHERE id=1 AND version=5;   -- 0 rows affected → conflict → retry
```

| Case | Use |
|------|-----|
| Counter (+/−, likes, stock) | **Atomic** |
| Complex logic + frequent conflicts | **Pessimistic (`FOR UPDATE`)** |
| Rare conflicts + need speed | **Optimistic (version)** |

**answer:** 3 حلول: **(1) Pessimistic** بـ `SELECT ... FOR UPDATE` — قفل من الأول، مناسب لو التعارض متوقّع. **(2) Atomic Update** — الـ DB يعمل الحساب في statement واحد (`balance = balance - 50`)، مناسب للـ counters. **(3) Optimistic** — عمود **version**، أحدّث بشرط `WHERE version = <القيمة القديمة>` وأزوّد الـ version؛ لو حد سبقني الشرط يفشل → **retry**. مناسب لو التعارض نادر.

---

### Q6. Which level to use?
- Default reads → **Read Committed**.
- Reports needing a consistent snapshot → **Repeatable Read**.
- Money/inventory where a concurrency bug = wrong money/stock → **Serializable** (or `SELECT ... FOR UPDATE`).

**answer:**
- القراءة العادية → **Read Committed**.
- تقرير محتاج snapshot ثابت → **Repeatable Read**.
- فلوس/مخزون حسّاسين → **Serializable** أو `SELECT ... FOR UPDATE`.

---

# TOPIC 5 — Normalization
📖 Resource: [Database Normalization](https://www.geeksforgeeks.org/dbms/introduction-of-database-normalization/)

### Q1. What is normalization and why do it?
- **Organizing tables to reduce redundancy** (same data stored in multiple places) and prevent **anomalies**.
- Benefits: less duplication, smaller size, consistent data, easier updates.

### Q2. What anomalies does it fix?
All 3 come from **duplication** — same fact stored in many rows.

**Bad table (students + courses in one):**
```
student_id | name  | email          | course
1          | Sara  | sara@x.com     | Math
1          | Sara  | sara@x.com     | Physics    ← Sara duplicated
1          | Sara  | sara@x.com     | Chemistry
2          | Ansh  | ansh@x.com     | Math
```

**1. Insert anomaly** — can't add data because other data is missing.
```
Add student Karim (no course yet):
INSERT VALUES (3, 'Karim', 'k@x.com', ???)   ← no course → can't insert!
```

**2. Update anomaly** — change email in many rows; miss one → contradiction.
```
Update Sara's email:
row 1: sara.new@x.com  ✅
row 2: sara.new@x.com  ✅
row 3: sara@x.com      ❌ forgot → two truths, DB stays silent
```

**3. Delete anomaly** — delete one thing, lose another by accident.
```
Ansh drops his only course (Math):
DELETE WHERE student_id=2 AND course='Math'
→ Ansh disappears from the DB entirely! 💀
```

**Fix — split into 3 tables** (M-to-M relationship needs a **junction table**):
```
students:                courses:                       enrollments:
id | name  | email       id | name    | credits        student_id | course_id
1  | Sara  | sara@x.com  10 | Math    | 3              1          | 10
2  | Ansh  | ansh@x.com  11 | Physics | 4              1          | 11
                         12 | Chem    | 3              1          | 12
                                                       2          | 10
```
Now:
- Student data in **one** place → no update anomaly for email.
- Course data in **one** place → rename "Math" → "Mathematics" = edit one row.
- Can add Karim to `students` with no enrollment → no insert anomaly.
- Delete an enrollment → student and course still exist → no delete anomaly.

**`enrollments`** is a **junction table** — the standard way to model a many-to-many relationship.

### Q3. Normal Forms — one-liners + tiny examples

- **1NF** — atomic values (no multi-values in one cell).
  ```
  ❌ courses = "Math, Physics"       ✅ one row per course
  ```

- **2NF** — 1NF + no **partial dependency** on a composite PK.
  ```
  PK = (student_id, course)
  ❌ student_name in this table → depends only on student_id (not the whole PK)
  ✅ move student_name to its own students table
  ```

- **3NF** — 2NF + no **transitive dependency** (non-key → non-key).
  ```
  ❌ students(student_id, dept, dept_head)
     dept_head depends on dept (a non-key), not directly on student_id
  ✅ split into students(student_id, dept) + departments(dept, dept_head)
  ```

- **BCNF** — stricter 3NF: every **determinant must be a super key**.
  ```
  ❌ class(student, subject, teacher)  — teacher → subject, but teacher isn't a super key
  ✅ split: teaches(teacher, subject) + enrolled(student, teacher)
  ```

> Memory line: *"the key, the whole key, and nothing but the key."*
> **3NF is the practical target.** BCNF is rare/textbook.

---

### Q4. When would you denormalize? ⭐
- When **reads dominate** and joins are slow — dashboards, feed pages, analytics.
- **Duplicate data on purpose** for speed. Trade-off: faster reads, must keep the duplicate in sync on writes.
- Example: store `likes_count` on `posts` instead of `COUNT(*)` from `likes` every read.
---

# TOPIC 6 — OLTP vs OLAP
📖 Resource: [OLTP vs OLAP](https://www.geeksforgeeks.org/dbms/difference-between-olap-and-oltp-in-dbms/)

### Q1. OLTP vs OLAP? ⭐
| | OLTP | OLAP |
|--|------|------|
| Purpose | Day-to-day **transactions** | **Analysis** / reporting |
| Operations | Short INSERT/UPDATE/DELETE | Complex read-only queries |
| Users | Many concurrent | Few analysts |
| Data | Current, operational | Historical, aggregated |
| Schema | **Normalized** | **Denormalized** (star/snowflake) |
| Example | Banking app, e-commerce checkout | BI dashboard, data warehouse |

**One-liner:** OLTP handles many small real-time transactions (normalized). OLAP handles few complex analytical queries over large historical data (denormalized).

### Q2. What is a data warehouse?
- A large **central store of historical data** collected from many sources, optimized for **analysis** (OLAP), not day-to-day transactions.

### Q3. Star vs Snowflake schema?
- **Star** — a central **fact table** (e.g., sales) surrounded by **dimension tables** (product, time, customer). Dimensions are **denormalized** (flat). Simple + fast.
- **Snowflake** — same, but dimensions are further **normalized** into sub-tables. Less redundancy, more joins.

---

# ✅ Part A Self-Test (say out loud)
1. DBMS vs RDBMS — one line.
2. Name the 5 command categories + one command each.
3. Why is TRUNCATE DDL not DML?
4. Primary key vs Unique key.
5. Can a PK be NULL? Composite?
6. What is referential integrity?
7. Natural vs surrogate key — which and why?
8. Explain ACID with money-transfer.
9. What's a savepoint?
10. Explain dirty / non-repeatable / phantom read.
11. Default isolation level? When Serializable?
12. Walk 1NF → 2NF → 3NF.
13. When would you denormalize?
14. OLTP vs OLAP.
15. Star vs snowflake schema.
