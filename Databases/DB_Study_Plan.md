# Databases — 2-Day Study Plan

**How to use:** go 1 → 2 → 3. Each step = **one concept + one specific link (10–20 min max) + the interview question to answer out loud**. "details" = my full notes lower in this file.
**Rule:** read the link → answer the Q out loud → tick → next. Don't open anything not listed here.

---

## 📅 DAY 1 — DBMS Theory + SQL Theory

### Part A — DBMS Fundamentals
1. **DBMS vs RDBMS + DDL/DML/DCL/TCL** → [read](https://www.geeksforgeeks.org/dbms/difference-between-rdbms-and-dbms/) · Q: DBMS vs RDBMS? one command per category? *(details: Topic 1)*
2. **Keys** (Primary/Foreign/Unique/Composite/Candidate/Surrogate) → [read](https://www.geeksforgeeks.org/dbms/types-of-keys-in-relational-model-candidate-super-primary-alternate-and-foreign/) · Q: PK vs Unique? why surrogate over natural? *(Topic 2)*
3. **ACID + Transactions** → [read](https://www.geeksforgeeks.org/dbms/acid-properties-in-dbms/) · Q: explain ACID with money-transfer. *(Topic 3)*
4. **Isolation Levels + 3 anomalies** → [read](https://www.geeksforgeeks.org/dbms/transaction-isolation-levels-dbms/) · Q: dirty vs non-repeatable vs phantom? default level? *(Topic 4)*
5. **Normalization 1NF→3NF** → [read](https://www.geeksforgeeks.org/dbms/introduction-of-database-normalization/) · Q: walk 1NF→3NF; when denormalize? *(Topic 5)*
6. **OLTP vs OLAP** → [read](https://www.geeksforgeeks.org/dbms/difference-between-olap-and-oltp-in-dbms/) · Q: OLTP vs OLAP? star schema? *(Topic 6)*

### Part B — SQL Theory
7. **SQL Execution Order** → [read](https://www.geeksforgeeks.org/sql/order-of-execution-of-sql-queries/) · Q: why can't you use a SELECT alias in WHERE? *(Topic 7)*
8. **JOINs** (all types) → [read](https://www.geeksforgeeks.org/sql/sql-join-set-1-inner-left-right-and-full-joins/) · Q: INNER vs LEFT? users who never ordered? *(Topic 8)*
9. **GROUP BY + HAVING** → [read](https://www.geeksforgeeks.org/sql/sql-group-by/) · Q: WHERE vs HAVING? COUNT(*) vs COUNT(col)? *(Topic 9)*
10. **Subqueries + CTE + Window Functions** → [read](https://www.geeksforgeeks.org/sql/window-functions-in-sql/) · Q: RANK vs DENSE_RANK? CTE vs subquery? *(Topic 10)*
11. **Views + Triggers + Stored Procedures** → [read](https://www.geeksforgeeks.org/sql/sql-views/) · Q: view vs materialized view? when trigger? *(Topic 11)*
12. **UNION vs UNION ALL** → [read](https://www.geeksforgeeks.org/sql/sql-union-clause/) · Q: difference? which is faster? *(Topic 12)*
13. **EXISTS vs IN** → [read](https://www.geeksforgeeks.org/sql/in-vs-exists-in-sql/) · Q: when each? NOT IN + NULL trap? *(Topic 13)*
14. **NULL + 3-valued logic** → [read](https://www.geeksforgeeks.org/sql/sql-null-values/) · Q: why `NULL = NULL` not TRUE? COALESCE? *(Topic 14)*

### Part C — Indexes + Concurrency
15. **Indexes** ⭐ → [read](https://www.geeksforgeeks.org/dbms/indexing-in-databases-set-1/) · Q: B-tree? composite column order? when NOT used? *(Topic 15)*
16. **Query Optimization playbook** → *(details: Topic 16 — memorize the 8 steps)* · Q: speed up a slow query?
17. **MVCC + Locks** → [read](https://www.geeksforgeeks.org/dbms/what-is-multi-version-concurrency-control-mvcc-in-dbms/) · Q: what's MVCC? shared vs exclusive lock? *(Topic 17)*
18. **Optimistic vs Pessimistic + Deadlocks** → [read](https://www.geeksforgeeks.org/dbms/deadlock-in-dbms/) · Q: prevent double-selling last item? prevent deadlock? *(Topic 18)*
19. **N+1 Problem** → *(details: Topic 19)* · Q: what's N+1? example + fix?
20. **DELETE vs TRUNCATE vs DROP** → [read](https://www.geeksforgeeks.org/sql/difference-between-delete-drop-and-truncate/) · Q: differences? *(Topic 20)*
21. **SQL Injection + Prepared Statements** → [read](https://www.geeksforgeeks.org/sql/sql-injection/) · Q: how prevent injection? *(Topic 21)*

**✅ End of Day 1 — answer the 25-question self-test at the bottom of this file.**

---

## 📅 DAY 2 — NoSQL + Caching + Practice

### Part A — NoSQL + Scale
22. **SQL vs NoSQL** → [read](https://www.mongodb.com/nosql-explained/nosql-vs-sql) · Q: when each? name the 4 NoSQL types. *(Topic 22)*
23. **CAP Theorem + BASE** → [read](https://www.geeksforgeeks.org/dbms/the-cap-theorem-in-dbms/) · Q: explain CAP; CP vs AP example. *(Topic 23)*
24. **Mongo: document/collection + embed vs reference** ⭐ → [read](https://www.mongodb.com/docs/manual/core/data-model-design/) · Q: embed vs reference — 2 examples each. *(Topics 25–26)*
25. **Mongo Aggregation Pipeline** → [read](https://www.geeksforgeeks.org/mongodb/aggregation-in-mongodb/) · Q: what does `$lookup` do? *(Topic 27)*
26. **Replication vs Sharding** → [read](https://www.mongodb.com/docs/manual/sharding/) · Q: difference? good shard key? sync vs async? *(Topics 29–30)*

### Part B — Caching
27. **Why cache + Cache patterns** → [read](https://www.geeksforgeeks.org/system-design/cache-aside-pattern/) · Q: cache-aside vs write-through vs write-behind. *(Topics 32–33)*
28. **TTL + Eviction (LRU/LFU)** → [read](https://redis.io/docs/latest/develop/reference/eviction/) · Q: LRU vs LFU? why always set TTL? *(Topic 35)*
29. **Cache Stampede** → [read](https://www.en.wikipedia.org/wiki/Cache_stampede) · Q: what is it? how prevent (mutex)? *(Topic 36)*
30. **Redis data structures** → [read](https://redis.io/docs/latest/develop/data-types/) · Q: leaderboard? rate limit? distributed lock? *(Topic 37)*

### Part C — Practice (only after theory)
31. **SQL queries** — 8 problems on [SQLZoo JOIN + SELECT](https://sqlzoo.net/) + 3 window-function problems on [LeetCode Top SQL 50](https://leetcode.com/studyplan/top-sql-50/). Must write **2nd highest salary** from memory.
32. **Drill Q&A** — skim [Devinterview SQL Qs](https://github.com/Devinterview-io/sql-interview-questions), answer 15 out loud.

### Part D — Mock (last hour)
Answer out loud, timed: the **Top-40 questions** at the bottom of this file.

---

# 📚 COMPLETE INTERVIEW TOPIC CHECKLIST (reference — open only if a step above is unclear)

## 1. DBMS vs RDBMS + Command Categories

**DBMS** = software to store/manage data (may not use relations). **RDBMS** = DBMS that stores in tables with relationships (MySQL, Postgres, Oracle).

**Command categories:**
| Category | Purpose | Examples |
|----------|---------|----------|
| **DDL** — Data Definition | Define schema | `CREATE, ALTER, DROP, TRUNCATE, RENAME` |
| **DML** — Data Manipulation | Change data | `INSERT, UPDATE, DELETE, MERGE` |
| **DQL** — Data Query | Read data | `SELECT` |
| **DCL** — Data Control | Permissions | `GRANT, REVOKE` |
| **TCL** — Transaction Control | Transactions | `COMMIT, ROLLBACK, SAVEPOINT` |

**Interview Qs:** DBMS vs RDBMS · Give one command per category · Why TRUNCATE is DDL not DML? (structural reset, not row-by-row).

**Resource:** [GeeksforGeeks — DBMS vs RDBMS](https://www.geeksforgeeks.org/dbms/difference-between-rdbms-and-dbms/) · [GeeksforGeeks — SQL Command Types](https://www.geeksforgeeks.org/sql/sql-ddl-dql-dml-dcl-tcl-commands/).

---

## 2. Keys

- **Primary Key** — unique + not null. One per table. Identifies each row.
- **Composite Key** — PK made of multiple columns together.
- **Foreign Key** — points to another table's PK. Relational integrity.
- **Unique Key** — enforces uniqueness (nullable). Can have many per table.
- **Candidate Key** — any column(set) that COULD be PK.
- **Super Key** — any set that uniquely identifies a row (superset of candidate).
- **Surrogate Key** — artificial ID (auto-increment / UUID). Prefer over natural (which is business data — email, phone — can change).
- **Natural Key** — meaningful business value used as PK.

**ON DELETE actions:**
- `CASCADE` — delete children too.
- `SET NULL` — set FK to NULL.
- `RESTRICT` / `NO ACTION` — prevent delete.

**Interview Qs:** PK vs Unique · Can PK be null? Composite? · Why surrogate over natural? · ON DELETE CASCADE — when?

**Resource:** [GFG — Keys in DBMS](https://www.geeksforgeeks.org/dbms/types-of-keys-in-relational-model-candidate-super-primary-alternate-and-foreign/).

---

## 3. ACID + Transactions + Savepoints

- **A**tomicity — all or nothing.
- **C**onsistency — moves from valid state → valid state (constraints held).
- **I**solation — concurrent txns don't see each other's half-work.
- **D**urability — committed = survives crash.

**Savepoint** = a marker inside a transaction. Can rollback to it without rolling back the entire transaction.

```sql
BEGIN;
INSERT INTO orders (...) VALUES (...);
SAVEPOINT sp1;
UPDATE inventory SET qty = qty - 1 WHERE id = 5;
-- oops, wrong item
ROLLBACK TO sp1;   -- undo just the update, keep the insert
COMMIT;
```

**Use-cases:**
- **Money transfer** → debit A + credit B in one txn (Atomicity).
- **Order placement** → insert order + reduce stock + insert payment.

**Interview Qs:** Explain ACID · What's a transaction? · Auto-commit — what is it? · What's a savepoint?

**Resource:** [GFG — ACID Properties](https://www.geeksforgeeks.org/dbms/acid-properties-in-dbms/) · [Video — Hussein Nasser ACID](https://www.youtube.com/watch?v=pomxJOFVcQs).

---

## 4. Isolation Levels + Anomalies

**3 anomalies:**
- **Dirty read** — read another txn's uncommitted data.
- **Non-repeatable read** — same row read twice → different values (someone committed between).
- **Phantom read** — same query twice → different set of rows (someone inserted/deleted).

**4 levels:**
| Level | Prevents | Cost |
|-------|----------|------|
| **Read Uncommitted** | nothing | fastest, unsafe |
| **Read Committed** ⭐ (default) | dirty reads | may see non-repeatable |
| **Repeatable Read** | non-repeatable | may see phantoms |
| **Serializable** | all 3 — behaves as if serial | slowest, retries |

**Use-cases:**
- Default read → **Read Committed**.
- Report needing consistent snapshot → **Repeatable Read**.
- Financial reconciliation / inventory → **Serializable**.

**Interview Qs:** Explain 3 anomalies · When Serializable? · Trade-offs.

**Resource:** [GFG — Isolation Levels](https://www.geeksforgeeks.org/dbms/transaction-isolation-levels-dbms/) · [Video — Hussein Nasser Isolation Levels](https://www.youtube.com/watch?v=-gxyut1VLcs).

---

## 5. Normalization + Denormalization

- **1NF** — atomic values (no `"a,b,c"` in a cell). No repeating groups.
- **2NF** — 1NF + no partial dependency on a composite key (every non-key col depends on the FULL PK).
- **3NF** — 2NF + no transitive dependency (non-key → non-key). Every non-key col depends on **only** the PK.
- **BCNF** — stricter 3NF; every determinant is a candidate key.
- **Denormalization** — deliberate duplication for read speed.

**When normalize:** OLTP (writes matter, integrity critical) — banking, orders.
**When denormalize:** OLAP / read-heavy (dashboards, analytics, feed pages).

**Instagram example:** `posts.likes_count` denormalized because read on every scroll.

**Interview Qs:** Walk 1NF → 3NF · When denormalize? · Design schema for X (normalize first).

**Resource:** [GFG — Normalization](https://www.geeksforgeeks.org/dbms/introduction-of-database-normalization/) · [Video — Gate Smashers Normalization](https://www.youtube.com/watch?v=xoTyrdT9SZI).

---

## 6. OLTP vs OLAP + Data Warehouse

| | OLTP | OLAP |
|--|------|------|
| Purpose | Day-to-day transactions | Analysis / reports |
| Users | Many concurrent | Few analysts |
| Ops | Short reads + writes | Complex reads, long queries |
| Schema | Normalized | **Star / Snowflake** (denormalized) |
| Examples | Banking, e-commerce, CRM | Data warehouse, BI dashboards |

**Star schema** = fact table + surrounding dimension tables (denormalized dims).
**Snowflake schema** = star + dimensions further normalized.

**Interview Qs:** OLTP vs OLAP · What's a data warehouse? · Star vs snowflake.

**Resource:** [GFG — OLTP vs OLAP](https://www.geeksforgeeks.org/dbms/difference-between-olap-and-oltp-in-dbms/) · [ByteByteGo Video — OLTP vs OLAP](https://www.youtube.com/watch?v=iw-5kFzIdgY).

---

## 7. SQL Execution Order ⭐ (surprisingly asked)

Written order ≠ execution order:
```
Written:   SELECT → FROM → JOIN → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT
Executed:  FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT
```

**Why it matters:**
- You **can't** use a column alias defined in `SELECT` inside `WHERE` (WHERE runs before SELECT).
- **You CAN** use alias in `ORDER BY` (ORDER BY runs after SELECT).
- HAVING can filter on aggregates (runs after GROUP BY).

**Interview Qs:** Why can't alias in WHERE? · Order of execution.

**Resource:** [SQL Execution Order — SQLBolt](https://sqlbolt.com/) · [Article](https://blog.jooq.org/a-beginners-guide-to-the-true-order-of-sql-operations/).

---

## 8. JOINs

- **INNER JOIN** — rows with match on both.
- **LEFT JOIN** — all left + matching right (NULL if none).
- **RIGHT JOIN** — mirror of LEFT.
- **FULL OUTER JOIN** — all from both.
- **CROSS JOIN** — cartesian product.
- **SELF JOIN** — table joined to itself.
- **Correlated subquery** — subquery references outer row (runs per row, slow).

**Use-cases:**
- Users who never ordered → `users LEFT JOIN orders WHERE order.id IS NULL`.
- Employees + managers (same table) → SELF JOIN on `emp.manager_id = mgr.id`.
- Compare each user to average → correlated subquery.

**Interview Qs:** All join types + differences · Users without orders · Self join example.

**Resource:** [GFG — SQL JOINs](https://www.geeksforgeeks.org/sql/sql-join-set-1-inner-left-right-and-full-joins/) · [SQL Joins Visualizer](https://sql-joins.leopard.in.ua/).

---

## 9. GROUP BY + HAVING + Aggregate Functions

- `GROUP BY col` → one row per unique col value.
- Aggregates: `COUNT, SUM, AVG, MIN, MAX`.
- `COUNT(*)` = all rows; `COUNT(col)` = non-nulls; `COUNT(DISTINCT col)` = uniques.
- `WHERE` filters rows (before grouping). `HAVING` filters groups (after grouping).

**Interview Qs:** WHERE vs HAVING · COUNT(*) vs COUNT(col) · Top 3 customers by spending · Group with more than N members.

**Resource:** [GFG — GROUP BY / HAVING](https://www.geeksforgeeks.org/sql/sql-group-by/).

---

## 10. Subqueries + CTEs + Window Functions

- **Subquery** — nested query.
- **CTE** (`WITH name AS (...)`) — named, readable, supports recursion.
- **Window functions** — compute over a window WITHOUT collapsing rows.
  - `ROW_NUMBER()` — unique seq number per partition.
  - `RANK()` — same value = same rank, gaps in numbering.
  - `DENSE_RANK()` — no gaps.
  - `LAG() / LEAD()` — previous / next row's value.
  - `SUM() OVER (...)` — running total.

**Use-cases:**
- **2nd highest salary per dept** → `ROW_NUMBER()`.
- **Cumulative balance** → `SUM() OVER (PARTITION BY user ORDER BY date)`.
- **Category tree** → recursive CTE.
- **Month-over-month growth** → `LAG()`.

**Interview Qs:** CTE vs subquery · RANK vs DENSE_RANK · 2nd highest salary · Top N per group · Running total.

**Resource:** [GFG — CTE](https://www.geeksforgeeks.org/sql/cte-in-sql/) · [GFG — Window Functions](https://www.geeksforgeeks.org/sql/window-functions-in-sql/) · [Mode — Window Functions Tutorial](https://mode.com/sql-tutorial/sql-window-functions).

---

## 11. Views + Stored Procedures + Triggers + Functions

- **View** = named saved query. Runs live on read. Simplifies + hides complexity + security (grant on view not on table).
- **Materialized View** = stored result. Fast reads, need refresh. Use for expensive aggregates.
- **Stored Procedure** = precompiled block of SQL. Reusable server-side logic. Called with `CALL proc_name()`.
- **Function** = returns a value (unlike procedure).
- **Trigger** = auto-run on event (INSERT/UPDATE/DELETE). BEFORE / AFTER. Row-level or statement-level.

**Use-cases:**
- **View** — hide sensitive columns from a role.
- **Materialized view** — nightly analytics rollup.
- **Trigger** — audit log every UPDATE on `users`.
- **Stored procedure** — batch job logic.

**Interview Qs:** View vs Materialized view · Procedure vs Function · When trigger? · Downsides of triggers? (hidden logic, hard to debug).

**Resource:** [GFG — Views](https://www.geeksforgeeks.org/sql/sql-views/) · [GFG — Triggers](https://www.geeksforgeeks.org/sql/sql-trigger-student-database/).

---

## 12. UNION / INTERSECT / EXCEPT

- **UNION** — combines rows, removes duplicates.
- **UNION ALL** — combines rows, keeps duplicates (faster).
- **INTERSECT** — rows in both.
- **EXCEPT** (MINUS in Oracle) — rows in first, not in second.

Both queries must have same columns / compatible types.

**Interview Q:** UNION vs UNION ALL — always asked. Prefer UNION ALL unless you actually need deduplication (it's cheaper).

**Resource:** [GFG — SQL Set Operations](https://www.geeksforgeeks.org/sql/sql-union-clause/).

---

## 13. EXISTS vs IN vs JOIN

- **`IN (subquery)`** — returns list, matches value.
- **`EXISTS (subquery)`** — returns TRUE/FALSE, short-circuits at first match.
- **JOIN** — often equivalent, sometimes faster.

**Rules of thumb:**
- Large subquery result → `EXISTS` (short-circuits).
- Small list → `IN`.
- Need columns from the other table → **JOIN**.

**NOT IN + NULL trap:** `NOT IN (subquery with NULL)` returns nothing! Use `NOT EXISTS` instead.

**Interview Q:** EXISTS vs IN — when each · NOT IN gotcha.

**Resource:** [GFG — EXISTS vs IN](https://www.geeksforgeeks.org/sql/in-vs-exists-in-sql/).

---

## 14. NULL + 3-Valued Logic

- NULL means "unknown" (NOT zero, NOT empty string).
- **`NULL = NULL` → NULL** (not TRUE!). Use `IS NULL`.
- Any op with NULL → NULL: `NULL + 5 = NULL`.
- Aggregates skip NULLs (except `COUNT(*)`).
- **`COALESCE(a, b, c)`** — first non-NULL.
- **`NULLIF(a, b)`** — NULL if a=b, else a.

**Interview Qs:** Why `NULL = NULL` not TRUE? · COALESCE example · `WHERE col != 5` and col is NULL — does row match? (no! NULL != 5 is NULL, not TRUE).

**Resource:** [Modern SQL — NULL](https://modern-sql.com/concept/three-valued-logic).

---

## 15. Indexes ⭐ (THE #1 performance topic)

**Concepts:**
- No index → **sequential (full table) scan**. With index → **index scan** (log N).
- **B-tree** (default) — equality + range + sort.
- **Hash index** — equality only, faster for pure `=` (rare in practice).
- **Composite index `(a, b)`** — serves `WHERE a=?` and `WHERE a=? AND b=?`, **NOT** `WHERE b=?`. Column order **matters**.
- **Selectivity** — low-selectivity column (like status with 3 values) → index rarely helpful alone.
- **Covering index** — index contains all needed columns → skip table lookup.
- **Clustered index** — data physically sorted by it. One per table. PK usually clustered.
- **Non-clustered index** — separate structure pointing to rows.

**When index NOT used:**
- Small table.
- Function on column: `WHERE UPPER(email) = ?` (unless function index).
- Low selectivity.
- Data type mismatch: `WHERE id = '5'` on INT column.
- `LIKE '%foo%'` — leading wildcard.

**Interview Qs:** Index — what and why? · B-tree · Why not index everything? · When not used? · Composite order? · Clustered vs non-clustered.

**Resource:** [GFG — Indexing](https://www.geeksforgeeks.org/dbms/indexing-in-databases-set-1/) · [Use The Index Luke — free book](https://use-the-index-luke.com/) ⭐⭐ · [Video — Hussein Nasser B-tree](https://www.youtube.com/watch?v=UpycpnwvOOU).

---

## 16. Query Optimization — "Speed up slow query" playbook

1. Run **EXPLAIN** / query plan.
2. **Full table scan** on big table → add index.
3. Row estimate vs actual mismatch → refresh stats (`ANALYZE`).
4. Composite index for multi-column filter.
5. `SELECT *` → only needed columns (covering index helps).
6. Subquery → JOIN (often equivalent + faster).
7. Cache result (Redis) if not real-time.
8. Denormalize or materialized view for expensive aggregates.
9. Partition huge tables.

**Interview Q:** classic — "how would you speed this query?" — say the playbook.

**Resource:** [Video — Hussein Nasser Query Plan](https://www.youtube.com/watch?v=U69dJTe9uJo).

---

## 17. Concurrency + MVCC + Locks

- **Shared lock (S)** — for reads. Many can hold together.
- **Exclusive lock (X)** — for writes. One only.
- **Row-level** — better concurrency. **Table-level** — coarser, more blocking.
- **MVCC (Multi-Version Concurrency Control)** — writers create new row versions; readers see snapshot. Readers don't block writers, writers don't block readers.

**Interview Qs:** Shared vs exclusive · What's MVCC?

**Resource:** [GFG — MVCC](https://www.geeksforgeeks.org/dbms/what-is-multi-version-concurrency-control-mvcc-in-dbms/) · [Video — Hussein Nasser MVCC](https://www.youtube.com/watch?v=aA34mIbIH2E).

---

## 18. Optimistic vs Pessimistic Locking + Deadlocks

- **Pessimistic** — lock row upfront (`SELECT ... FOR UPDATE`). Use when conflicts likely.
- **Optimistic** — no lock; check a `version` column on update; retry on conflict. Use when conflicts rare.
- **Deadlock** — A waits for lock B holds; B waits for lock A holds. DBMS detects → aborts one.
- **Prevent deadlocks** — always acquire locks in the same order across the app.

**Use-cases:**
- **Ticket booking / stock decrement** → pessimistic.
- **Blog post edit** (rare conflicts) → optimistic.
- **Two users buy last item** → `SELECT quantity FOR UPDATE` on inventory row.

**Interview Qs:** Optimistic vs pessimistic · What causes a deadlock? Prevent · Prevent double-selling scenario.

**Resource:** [Article — Optimistic vs Pessimistic](https://www.baeldung.com/jpa-optimistic-locking) · [Video — Hussein Nasser Deadlocks](https://www.youtube.com/watch?v=OeEnKrO5jNU).

---

## 19. N+1 Query Problem

ORM loads parent, then queries child once **per** parent → 1 + N queries.

**Example (blog page — bad):**
```
posts = db.query("SELECT * FROM posts LIMIT 10")     # 1 query
for post in posts:
    author = db.query(f"SELECT * FROM users WHERE id = {post.author_id}")  # N queries
```

**Fixes:**
- `JOIN` in one query.
- `WHERE id IN (...)` batch.
- ORM eager load: `.select_related()` (Django), `.include()` (Sequelize), `.joinedload()` (SQLAlchemy).
- GraphQL: DataLoader.

**Interview Q:** What's N+1? Example + fix.

**Resource:** [GFG — N+1 Problem](https://www.baeldung.com/cs/orm-n-plus-one-select-problem).

---

## 20. DELETE vs TRUNCATE vs DROP

| | DELETE | TRUNCATE | DROP |
|--|--------|----------|------|
| Type | DML | DDL | DDL |
| Removes | rows (WHERE optional) | all rows | table itself |
| Rollback | yes | usually no | no |
| Triggers | fires | doesn't | doesn't |
| Reset auto-inc | no | yes | table gone |
| Speed | slow | fast | fastest |

**Resource:** [GFG — DELETE vs TRUNCATE vs DROP](https://www.geeksforgeeks.org/sql/difference-between-delete-drop-and-truncate/).

---

## 21. SQL Injection + Prepared Statements + Connection Pooling

- **SQL Injection** — user input concatenated into SQL: `"SELECT * FROM users WHERE id = " + input` → `input = "1 OR 1=1"` returns all.
- **Prepared statement** = query with `?` placeholders. DB parses once; params bound separately → injection impossible.
```java
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
ps.setInt(1, userId);   // safe from injection
```
- **Connection pooling** — reuse DB connections. New connection is expensive (TCP + auth). Pool holds N open, hands out on demand.

**Interview Qs:** How prevent SQL Injection? · What's connection pooling · Why prepared statements?

**Resource:** [OWASP — SQL Injection Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) · [Video — Connection Pool by Hussein Nasser](https://www.youtube.com/watch?v=zBnMHhBSuws).

---

## 22. SQL vs NoSQL — the choice

| | SQL | NoSQL |
|--|-----|-------|
| Structure | Tables, rows, fixed schema | Flexible (docs / key-value / etc) |
| Consistency | Strong (ACID) | Usually eventual (BASE) |
| Scale | Vertical mostly | Horizontal easily |
| Joins | Powerful | Limited (Mongo has `$lookup`) |
| When | Structured, related data | Flexible/nested, huge scale |

**NoSQL flavors:**
- **Document** — MongoDB, CouchDB.
- **Key-Value** — Redis, DynamoDB.
- **Wide-column** — Cassandra, HBase.
- **Graph** — Neo4j.

**Use-cases (memorize):**
| Scenario | Pick |
|----------|------|
| Banking, orders, invoices | SQL |
| E-commerce product catalog (varying attrs) | Mongo |
| Session tokens, rate limit, cache | Redis |
| Time-series / IoT | Cassandra |
| Social graph | Neo4j |
| Likes / view counter | Redis |

**Resource:** [MongoDB — SQL vs NoSQL](https://www.mongodb.com/nosql-explained/nosql-vs-sql) · [Video — ByteByteGo SQL vs NoSQL](https://www.youtube.com/watch?v=ruz-vK8IesE).

---

## 23. CAP Theorem + BASE

- **CAP** — during a network **P**artition you can only pick **C** (Consistency) OR **A** (Availability).
  - CP systems — refuse writes during split (banks): MongoDB (single-primary), HBase.
  - AP systems — accept writes, reconcile later: Cassandra, DynamoDB.
- **BASE** — Basically Available, Soft state, Eventual consistency (NoSQL contrast to ACID).

**Interview Qs:** Explain CAP · CP vs AP example · Why can't you have all 3.

**Resource:** [Video — ByteByteGo CAP Theorem](https://www.youtube.com/watch?v=BHqjEjzAicA).

---

## 24. Consistency Models

- **Strong** — reads see latest committed write. Slowest.
- **Eventual** — replicas eventually agree. Fastest.
- **Causal** — related writes seen in order.
- **Read-your-writes** — you see your own writes immediately.
- **Linearizable** — as if operations happened in real-time sequence.

**Use-cases:** bank balance → strong. Social feed likes → eventual. User's own profile after edit → read-your-writes.

**Resource:** [Jepsen — Consistency Models](https://jepsen.io/consistency).

---

## 25. MongoDB Fundamentals

- **Document** — JSON-like (stored as **BSON**). Can nest / have arrays.
- **Collection** — bunch of documents (like a table, but flexible schema).
- **`_id`** — auto ObjectId primary key.
- CRUD: `insertOne`, `find`, `updateOne`, `deleteOne` + batch versions.
- Query operators: `$lt, $gt, $in, $or, $and, $exists, $elemMatch, $regex`.

**Interview Qs:** SQL row vs Mongo doc · What's BSON · `$elemMatch` — when.

**Resource:** [MongoDB University — Free Courses](https://learn.mongodb.com/) ⭐.

---

## 26. MongoDB Embed vs Reference ⭐

**Embed** when:
- Read together always.
- Doesn't grow unboundedly.
- Not queried independently.

**Reference** when:
- Shared across parents.
- Grows unboundedly.
- Queried independently.
- Would blow 16MB doc limit.

**Use-cases:**
- Order line items → **embed** (frozen at purchase).
- Product referenced by many orders → **reference**.
- Blog post + limited comments → embed.
- Reddit-style (10K+ comments) → reference.

**Resource:** [MongoDB Docs — Data Model Design](https://www.mongodb.com/docs/manual/core/data-model-design/).

---

## 27. Mongo Aggregation Pipeline

Sequence of stages, each transforming docs:
```
$match  → filter (like WHERE)
$group  → group by (like GROUP BY)
$sort
$limit
$project → shape output
$lookup  → join to another collection
$unwind  → split array into multiple docs
```

**Use-case — top spenders:**
```
$match: status=completed → $group by customer, sum total → $sort desc → $limit 10
```

**Resource:** [MongoDB Docs — Aggregation](https://www.mongodb.com/docs/manual/aggregation/).

---

## 28. MongoDB ACID Support

**Yes since 4.0** — multi-document transactions on a replica set. **BUT** — good schema (embedding) usually removes the need. Transactions = escape hatch, not default.

Single-doc writes always atomic.

---

## 29. Replication + HA

- **Master-replica** — writes on master, reads spread on replicas.
- **Sync replication** — master waits for replica ack before COMMIT. Safe, slow.
- **Async** — master commits, replicas catch up later. Fast, may lose recent writes on crash.
- **Semi-sync** — waits for at least one replica.
- **Failover** — replica auto-promoted on master crash.

**Use-cases:**
- Banking → sync (can't lose txns).
- Analytics dashboard → async fine.

**Resource:** [Video — Hussein Nasser Replication](https://www.youtube.com/watch?v=bxmDnn7lrnk).

---

## 30. Sharding + Horizontal Scale

- **Vertical scale** — bigger machine. Has a ceiling.
- **Horizontal (sharding)** — split data across nodes.
- **Shard key** — decides which shard a row lives on.
- **Hot shard** — bad shard key → uneven load. Monotonic ID → all new writes hit one shard.

**Good shard keys:** high cardinality + evenly distributed (hashed `user_id`, `tenant_id`).

**Interview Qs:** Replication vs sharding · Shard key criteria · Instagram sharding.

**Resource:** [Video — ByteByteGo Sharding](https://www.youtube.com/watch?v=5faMjKuB9bc).

---

## 31. Backups + PITR

- **Full backup** — everything.
- **Incremental** — changes since last incremental.
- **Differential** — changes since last full.
- **PITR (Point-in-Time Recovery)** — full backup + write-ahead log replay → restore to exact moment.

**Interview Q:** How would you recover from `DROP TABLE users;` mistake?
→ base backup + replay WAL up to just before the DROP.

---

## 32. Why Cache?

RAM is 50–100× faster than disk. Cache = fast layer in front of a slower store (DB, API, computed result).

**Use-cases:**
- User profile shown on every request.
- Trending posts / top products.
- API rate limit counters.
- Session storage (multi-server safe).
- Expensive aggregation result.

---

## 33. Cache Patterns

| Pattern | How | When |
|---------|-----|------|
| **Cache-aside (lazy)** ⭐ default | App: check cache → miss → hit DB → set cache. Write: **invalidate key**. | Reads > writes |
| **Read-through** | Cache fetches from DB on miss | Simpler call sites |
| **Write-through** | Write to cache + DB synchronously | Strong consistency |
| **Write-behind (write-back)** | Write to cache immediately, async to DB | High write throughput, OK with loss risk |

**Cache-aside pseudocode:**
```
get(key):
  data = cache.get(key)
  if data: return data
  data = db.query(key)
  cache.set(key, data, ttl=300)   # always TTL
  return data

update(key, value):
  db.update(key, value)
  cache.delete(key)   # invalidate, don't overwrite
```

**Interview scenario:** IoT sensor ingestion, OK with a few seconds loss → **write-behind**. Bank balance → **write-through**.

**Resource:** [AWS — Caching Best Practices](https://aws.amazon.com/caching/best-practices/).

---

## 34. Cache Invalidation

**"There are only two hard things: cache invalidation and naming things."**

- **TTL** — always set one as safety net.
- **Explicit delete** on write.
- **Event-driven** — DB change publishes event → invalidate everywhere.

---

## 35. TTL + Eviction

**Eviction (when cache full):**
- **LRU** — Least Recently Used. Default choice for cache.
- **LFU** — Least Frequently Used. Better when there's a stable hot set.
- **noeviction** — refuse writes when full (strict store).
- **volatile-*** — only evict keys with TTL.

**Why always TTL?** Safety net if invalidation logic has a bug.

---

## 36. Cache Stampede (Thundering Herd)

Hot key expires → thousands of concurrent misses → all hit DB together → DB melts.

**Fixes:**
- **Mutex** — only first requester repopulates; others wait briefly.
- **Probabilistic early expiration** — recompute *before* TTL, randomized timing.
- **Jitter TTLs** — spread out expiration times.

**Resource:** [Video — Cache Stampede](https://www.youtube.com/watch?v=6QuFtCZFqho).

---

## 37. Redis Data Structures + Use-cases

| Structure | Use-case |
|-----------|----------|
| **String** | Simple cache, counter |
| **Hash** | User profile fields |
| **List** | Recent activity feed (`lpush` + `ltrim`) |
| **Set** | Unique members (roles, tags) |
| **Sorted Set (ZSET)** | Leaderboard (score-sorted) |
| **Bitmap** | Daily active users (bit per user) |
| **HyperLogLog** | Approximate unique count |
| **Streams** | Event log (Kafka-lite) |

**Common tasks:**
- Leaderboard → ZSET (`ZADD`, `ZREVRANGE`).
- Rate limit → counter with TTL.
- Distributed lock → `SET key val NX EX 10`.
- Real-time chat → Pub/Sub.

**Resource:** [Redis Docs — Data Types](https://redis.io/docs/data-types/).

---

## 38. Redis Pub/Sub + Persistence

- **Pub/Sub** — publisher publishes to channel; all subscribers get it in real-time. Ephemeral (no history).
- **RDB (snapshot)** — periodic dump to disk. Fast restart, may lose last few minutes.
- **AOF (append-only file)** — logs every write. More durable, slightly slower.
- Choose: pure cache → RDB / skip persistence. Anything you'd hate to lose → AOF.

---

# 🎯 TOP 40 INTERVIEW QUESTIONS

### DBMS Basics
1. DBMS vs RDBMS.
2. Give one command each — DDL, DML, DCL, TCL.
3. Primary vs Foreign vs Unique key.
4. Composite key vs Composite index.
5. Why surrogate over natural keys?

### Transactions + Concurrency
6. Explain ACID.
7. What are the 4 isolation levels?
8. Explain dirty read / non-repeatable / phantom.
9. What's MVCC?
10. Optimistic vs Pessimistic locking + use-case each.
11. What's a deadlock? Prevent it.
12. Two users buy last item — prevent double-selling.

### SQL
13. All join types + difference.
14. WHERE vs HAVING.
15. COUNT(*) vs COUNT(col) vs COUNT(DISTINCT col).
16. CTE vs subquery.
17. RANK vs DENSE_RANK vs ROW_NUMBER.
18. **Write 2nd highest salary per department.**
19. What's a window function? 2 use-cases.
20. UNION vs UNION ALL.
21. EXISTS vs IN.
22. Why `NULL = NULL` is not TRUE?
23. SQL execution order.
24. DELETE vs TRUNCATE vs DROP.
25. View vs Materialized view.
26. When trigger?

### Normalization
27. Walk 1NF → 2NF → 3NF.
28. When to denormalize?
29. OLTP vs OLAP.

### Indexes + Perf
30. What's an index? B-tree?
31. When index NOT used?
32. Composite index — column order matters?
33. "Speed up this slow query" — the 8-step playbook.
34. N+1 problem — example + fix.

### NoSQL + Scale
35. SQL vs NoSQL — when each?
36. CAP theorem.
37. Replication vs sharding.
38. Embed vs reference in Mongo — 2 examples each.

### Caching
39. Cache-aside vs write-through vs write-behind.
40. Cache stampede — prevent.

---

# 🧠 ONE-LINER CHEAT SHEET

- **ACID** — Atomicity, Consistency, Isolation, Durability.
- **BASE** — Basically Available, Soft state, Eventual consistency.
- **CAP** — Consistency, Availability, Partition-tolerance; pick 2 during split.
- **MVCC** — snapshot per txn; readers don't block writers.
- **Index** — reads fast, writes slower.
- **B-tree** — equality + range + sort.
- **Composite `(a,b)`** — serves `a` and `a+b`, NOT `b` alone.
- **JOIN** — INNER=both, LEFT=all left + matches, FULL=all both.
- **GROUP BY** collapses; **HAVING** filters groups.
- **Window function** — computes across window, keeps rows.
- **N+1** — 1 parent + N child queries → JOIN or eager load.
- **Normalize** — remove redundancy. **Denormalize** for read speed.
- **DELETE** = rows, DML, rollback. **TRUNCATE** = all rows, DDL. **DROP** = table gone.
- **Replication** — copies of same data. **Sharding** — split data.
- **Cache-aside** — try cache → miss → DB → set. Write → invalidate.
- **TTL + jitter** — always TTL, jitter avoids stampede.
- **Cache stampede** — hot key expires → flood → mutex on repopulate.
- **Optimistic lock** — version column, retry.
- **Pessimistic lock** — `SELECT ... FOR UPDATE`.
- **Deadlock** — cycle of waits. Prevent by consistent lock order.
- **View** — live query. **Materialized view** — stored result.
- **UNION** dedupes; **UNION ALL** doesn't (faster).
- **EXISTS** short-circuits; **IN** matches list.
- **NULL** = unknown; use `IS NULL`.
