# Part C — Indexing (Interview Q&A)

Covers: what an index is · B-tree · when index NOT used · composite index · types · clustered vs non-clustered · covering index · speed-up-slow-query playbook.
Format: same as Part A — bullets + **answer:** in Arabic + English keywords. We discuss each Q before moving on.
📖 Resources: [GFG — Indexing](https://www.geeksforgeeks.org/dbms/indexing-in-databases-set-1/) · 🎓 Udemy (Hussein Nasser): sections **"Indexing"** + **"B-Tree"** (already have this course).

---

## TOPIC 1 — Why indexes exist

### Q1. What is an index and why do we use it?
- Extra data structure with **only the column's values, sorted** + a **pointer** to the row in the **heap** (the table on disk). **NOT** a copy of the data.
- Without index → **Seq Scan** on the heap. With index → jump to matches in **log N**.
- Default = **B-tree**. When it's actually used → see Q5.

**answer:** الـ **index** هو **data structure** جانبي بيحتوي **قيم العمود مرتّبة** + **pointer** للـ row في الـ **heap** (الجدول على الـ disk) — **مش نسخة من الـ data**، بس قيم + مؤشرات. من غيره → **seq scan**؛ معاه → jump بـ **log N**. الافتراضي **B-tree**. متى الـ DB بتستخدمه فعلًا → Q5.

---

### Q2. Why not index every column?
Indexes aren't free. Cost:
- **Write cost** — every INSERT/UPDATE/DELETE has to update every index too → writes slower.
- **Storage** — extra disk/memory.
- **Planner overhead** — more indexes = more choices to evaluate.

**answer:** الـ index مش ببلاش. كل ما تحطي index أكتر، **الكتابة بتبقى أبطأ** (INSERT/UPDATE/DELETE بيحدّثوا الـ index كمان)، وبيستهلك **مساحة**، وبيدّي الـ query planner شغل زيادة. فبتحطي index بس على الأعمدة اللي فعلًا بتفلتري/بترتّبي بيها.

---

## TOPIC 2 — How B-Tree works (the default index)

### Q3. How does a B-tree index work? ⭐⭐
- **Balanced tree** — all leaves at the same depth → any lookup takes the same steps.
- Traversal: **root → internal nodes → leaf → row pointer** in the heap.
- Depth grows **logarithmically** — million rows ≈ 4-5 levels.
- Handles **equality (`=`)**, **range (`>`, `<`, `BETWEEN`)**, and **sorting (`ORDER BY`)**.

```
Search for 12:
              [10]                  ← root
             /    \
         [4|7]   [14|17]            ← internal
        / | \   /  |  \
    ...     ...  [11,12,13] ...     ← leaves → row pointers
```

**B-tree vs B+Tree (what Postgres/MySQL actually use):**
| | B-tree | **B+Tree** |
|--|--------|------------|
| Row pointers | in **all nodes** | **only in leaves** |
| Internal nodes | keys + row pointers | **keys only** (for navigation) |
| Leaves | independent | **linked-list** between them |

**Why B+Tree wins for ranges:** query like `WHERE id > 35` descends once to the leaf, then just **walks the linked list of leaves sequentially** — no going up and down the tree.

**answer:** الـ **B-tree** = **balanced tree**، كل الـ leaves على نفس الـ depth، فالبحث بـ **log N**. الـ **B+Tree** (اللي بيستخدمه Postgres/MySQL فعليًا) = تعديل عليها: الـ **row pointers في الـ leaves بس** (الـ internal nodes فيها keys للـ navigation فقط)، والـ **leaves مربوطة linked-list**. ده بيخلّي الـ **range queries** (`id > 35`) والـ **ORDER BY** سريعين جدًا — بينزل مرة واحدة، وبعدها يمشي على الـ leaves تتابعيًا. الثمن: الـ **writes أبطأ** لأن كل insert/update بيحدّث الشجرة (splits).

---

### Q4. B-tree vs Hash index?
**How Hash works:** `hash(value) → bucket → (key + row pointer)` stored in a hashmap → lookup is **O(1)** (one step).

**Comparison:**
| | B-tree (B+Tree) | Hash |
|--|-----------------|------|
| `WHERE col = X` | ✅ log N | ✅ **O(1)** (faster) |
| Range (`>`, `<`, `BETWEEN`) | ✅ | ❌ |
| Sorting (`ORDER BY`) | ✅ | ❌ |
| Prefix match (`LIKE 'foo%'`) | ✅ | ❌ |
| Collisions | none | possible → slower |

**Why B+Tree is the default even though Hash is faster for `=`:**
1. Speed gap is tiny in practice — 1M rows in B+Tree ≈ 4-5 steps.
2. B+Tree is **general-purpose** — one index handles equality + range + sort + prefix.
3. Hash suffers from **collisions** — O(1) is best-case, not guaranteed.

**Real-world:** **Redis** uses hashing internally — it's a key-value store, all ops are exact `=`, no ranges needed → Hash is perfect.

```sql
CREATE INDEX idx_email_hash ON users USING HASH(email);   -- rare in Postgres
```

**answer:** الـ **Hash index** بيستخدم **hash function** بتحوّل القيمة لـ bucket فيه (key + row pointer) في hashmap → search **O(1)**. بس **مبيدعمش range ولا sorting ولا prefix match** لأن الـ hash بتشتّت القيم عشوائيًا. الـ **B+Tree** بيدعم كل ده بـ log N. عشان كده الـ **B+Tree هو الافتراضي** في كل الـ DBs — general-purpose، والفرق في السرعة عمليًا صغير (4-5 خطوات لمليون صف). **Redis** بيستخدم hashing لأنه key-value store وكل الـ ops `=` بس.

---

## TOPIC 3 — When indexes are NOT used

### Q5. When does an index NOT help (or not get used)? ⭐⭐
Even if an index exists, the DB might skip it:
- **Small table** — full scan is actually faster.
- **Low selectivity** — column has few distinct values (e.g. `status` with 3 values) → the index barely narrows anything.
- **Function on the column** — `WHERE UPPER(email) = 'X'` skips a normal index on `email` (need a **function index**).
- **Type mismatch** — `WHERE id = '5'` when `id` is INT → implicit cast may skip the index.
- **Leading wildcard** — `LIKE '%foo%'` can't use a B-tree (start position unknown). `LIKE 'foo%'` **can**.
- **`OR` on unindexed columns** — may fall back to scan.

**answer:** الـ DB بيتجاهل الـ index لو: الجدول **صغير** (scan أسرع)، أو **selectivity واطية** (العمود قيمه قليلة)، أو استخدمت **function على العمود** (`UPPER(email)`)، أو **type mismatch**، أو `LIKE '%foo%'` (بادئ بـ wildcard). القاعدة: الـ index بيفيد لما القيمة اللي بتفلتري بيها **نادرة** والعمود **مش متلبّس بأي conversion**.

**Common real-world example — `LIKE '%ZA%'`:**
```
EXPLAIN ANALYZE SELECT id, name FROM employees WHERE name LIKE '%ZA%';
→ Parallel Seq Scan on employees
  Rows Removed by Filter: 6,991,916   ← scanned ~7M rows despite index on `name`
```
The B-tree can't help — it needs a **known starting point**, and `%` at the start means the match can be anywhere in the string.

**Fix — GIN + Trigram** (for `LIKE '%...%'` / substring / fuzzy search):
- **Trigram** = a 3-character piece. Postgres pads the word with spaces on both sides to also index the start/end, then splits into 3-char chunks:
  ```
  "Salma" → "  Salma "
          → [ "  S", " Sa", "Sal", "alm", "lma", "ma " ]
                          └──── real trigrams ────┘
  ```
- **GIN (Generalized Inverted Index)** stores `trigram → list of rows containing it`.
- Query time: split `"ZA"` into trigrams → look up which rows share them → done, no table scan.
```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_employees_name_trgm ON employees USING GIN (name gin_trgm_ops);
-- Now LIKE '%ZA%' uses the index. Also supports fuzzy match: WHERE name % 'Salmaa'
```
Cost: bigger index, slower writes. Worth it for big tables that need substring search.

Alternatives: **full-text search** (`tsvector` + GIN) for word-based search, or **Elasticsearch** for heavy search needs.

---

## TOPIC 4 — Composite (multi-column) index

### Q6. What is a composite index? Does column order matter? ⭐⭐
- Index on multiple columns together, e.g. `INDEX(user_id, created_at)`.
- Sorted by the **first column**, then by the second within the first, etc.
- Works for the **leftmost prefix** only:
  - `WHERE user_id = ?` ✅
  - `WHERE user_id = ? AND created_at > ?` ✅
  - `WHERE created_at > ?` ❌ (skips the first column → index unusable)
- **Column order matters a lot.** Put the column you filter on most often **first**.

```sql
CREATE INDEX idx_orders ON orders(user_id, created_at);

-- ✅ uses index
SELECT * FROM orders WHERE user_id = 5 AND created_at > '2026-01-01';
-- ✅ uses index (user_id alone is the leftmost prefix)
SELECT * FROM orders WHERE user_id = 5;
-- ❌ can't use index (skipped user_id)
SELECT * FROM orders WHERE created_at > '2026-01-01';
```

**answer:** الـ **composite index** هو index على أكتر من عمود مع بعض. مرتّب بالعمود الأول، وبعدين التاني جوّاه، وهكذا. بيشتغل بس على **الـ leftmost prefix** — يعني index على `(user_id, created_at)` بيخدم queries على `user_id` لوحده أو `user_id + created_at`، بس **مش بيخدم query على `created_at` لوحده**. عشان كده **ترتيب الأعمدة مهم جدًا**: العمود اللي بفلتر بيه أكتر يبقى **الأول**.

---

## TOPIC 5 — Index types (brief)

### Q7. What index types are there?
- **B-tree** (default) — equality + range + sort.
- **Hash** — equality only, rare in practice.
- **GIN** (Generalized Inverted Index) — for "does this contain X?" — arrays, JSONB, full-text search, `LIKE '%foo%'` with trigrams.

**answer:** الـ **B-tree** هو الافتراضي (equality + range + sort). **Hash** للـ `=` بس. **GIN** للـ arrays / JSONB / full-text search / substring search (`LIKE '%...%'`).

---

## TOPIC 6 — Clustered vs Non-Clustered + Covering

### Q8. Clustered vs Non-Clustered index? ⭐
- **Clustered** — the **rows themselves** are physically sorted by this index. One per table (usually the primary key). Lookup goes straight to the row.
- **Non-clustered** — a separate structure with a **pointer** to the row's location. You get the key → then jump to the row. Can have many per table.

**answer:** الـ **clustered index** = الصفوف نفسها بتكون مرتّبة فيزيائيًا على القرص حسب العمود ده. واحد بس في كل جدول (عادةً الـ PK). الـ **non-clustered** = data structure منفصل بيحتوي على القيم + **pointer** للصف. تقدري تعملي كذا واحد في نفس الجدول.

---

### Q9. What is a covering index?
- Index that **contains all the columns** the query needs → the DB doesn't have to go back to the table (skip the row lookup).
- Very fast: everything served from the index alone.

Example:
```sql
-- Query: SELECT name FROM users WHERE email = 'x@y.com'
CREATE INDEX idx_users_email_name ON users(email, name);
-- The index has both email AND name → no need to touch the table
```

**answer:** الـ **covering index** هو index بيحتوي على **كل الأعمدة** اللي الـ query محتاجها، فالـ DB مبيرجعش للجدول أصلًا — بياخد كل حاجة من الـ index. أسرع بكتير للـ queries المحدّدة.

---

## TOPIC 7 — Query optimization playbook

### Q10. How do you speed up a slow query? ⭐⭐ (very common interview Q)
The playbook:
1. Run **`EXPLAIN` / `EXPLAIN ANALYZE`** to see the query plan.
2. If **full table scan** on a big table → add an appropriate index.
3. If **stats are stale** → `ANALYZE` the table.
4. Multi-column filter? → **composite index** with correct column order.
5. Change `SELECT *` → only needed columns (enables **covering index**).
6. Convert subquery → JOIN when equivalent (planner sometimes handles JOINs better).
7. Cache the result (Redis) if data doesn't need to be real-time.
8. For expensive aggregates → **materialized view** or **denormalize**.
9. For huge tables → **partitioning**.

**answer:** الـ playbook: أبدأ بـ **`EXPLAIN ANALYZE`** أشوف الخطة. لو full table scan على جدول كبير → أضيف **index مناسب**. لو الأرقام المقدّرة بعيدة عن الفعلية → `ANALYZE`. filter على أكتر من عمود → **composite index** بترتيب صح. أختار الأعمدة اللي محتاجاها بس (مش `SELECT *`) عشان أستفيد من **covering index**. للـ aggregates التقيلة → **materialized view** أو **denormalize**. للجداول العملاقة → **partitioning**.

---

### Q11. EXPLAIN vs EXPLAIN ANALYZE? ⭐
Both show the **query plan**. Difference: one predicts, the other measures.

| | EXPLAIN | EXPLAIN ANALYZE |
|--|---------|-----------------|
| Runs the query? | ❌ No | ✅ Yes |
| Numbers | **Estimated** (from stats) | **Actual** (measured) |
| Shows real time? | ❌ | ✅ |
| Safe on writes? | ✅ | ⚠️ actually deletes/updates |

**Example output:**
```
EXPLAIN → Index Scan ... (cost=0.29..8.30 rows=1)
EXPLAIN ANALYZE → Index Scan ... (cost=0.29..8.30 rows=1)
                                 (actual time=0.045..0.052 rows=2 loops=1)
                  Execution Time: 0.080 ms
```

**Key signal — `rows` estimated vs actual:**
Big gap between planner's `rows=` and `actual rows=` → stats are **stale** → run `ANALYZE table_name` to refresh them.

**⚠️ Safety:** `EXPLAIN ANALYZE` on `DELETE/UPDATE` **executes** the change. Wrap in a transaction:
```sql
BEGIN;
EXPLAIN ANALYZE DELETE FROM users WHERE ...;
ROLLBACK;
```

**answer:** الاتنين بيوروا الـ **query plan**. **`EXPLAIN`** بيوري الخطة المتوقّعة **من غير ما يشغّل** الـ query — الأرقام كلها **تقديرية** (estimated). **`EXPLAIN ANALYZE`** بيـ **يشغّل الـ query فعلًا** ويرجّع الخطة + **actual time** و **actual rows**. الفجوة بين estimated و actual = علامة على **stale stats** فبعمل `ANALYZE`. تحذير: `EXPLAIN ANALYZE` على `DELETE/UPDATE` بيمسح/يعدّل فعلًا — بلفّها في `BEGIN ... ROLLBACK`.

---

### Q12. Seq Scan vs Index Scan vs Bitmap Scan ⭐
The planner picks based on **how much of the table** the query will return:

| Scan | When picked | How it reads |
|------|-------------|--------------|
| **Index Scan** | tiny result (~<1%) | index → jump to each row (random I/O) |
| **Bitmap Scan** | medium result (~1–10%) | index → build page-map → read pages **sequentially** |
| **Seq Scan** | large result (~>10%) | read the whole table sequentially |

**Example — same column, different range:**
```
SELECT name FROM grades WHERE id < 100;      → Index Scan  (98 rows out of 5M)
SELECT name FROM grades WHERE id > 100;      → Seq Scan    (5M rows — most of the table)
SELECT name FROM grades WHERE g > 95;        → Bitmap Scan (~205K rows — in between)
```

**Why not always Index Scan?**
Index Scan = random jumps to disk. If matches are many, random I/O costs more than reading the whole table sequentially → Seq Scan wins.

**Bitmap Scan — 2 steps (بالعربي):**

**المرحلة 1: Bitmap Index Scan**
- بيمشي على الـ **index** بس (مش الجدول).
- لكل match بيشعّل **bit** للـ page اللي فيها الصف.
- النتيجة: **bitmap** = خريطة بالـ pages اللي فيها matches.

**المرحلة 2: Bitmap Heap Scan**
- بياخد الـ bitmap اللي جاهزة من المرحلة 1.
- بيقرا **بس الـ pages المشعّلة** بترتيب من الأصغر للأكبر (**sequential I/O**).
- لكل page بيجيب الصفوف اللي فيها.

**Recheck Cond — يعني إيه:**
- لو الـ bitmap صغير → **row-level** (bit لكل صف) → مفيش recheck.
- لو الـ bitmap كبر عن الـ `work_mem` → Postgres بيضغطه لـ **page-level** (bit لكل page) → يعرف الـ page فيها matches بس **مش عارف أنهي rows** → لازم يعمل **Recheck Cond** على كل صف لما يقرا الـ page.
---

## ✅ Part C Self-Test (say out loud)
1. What is an index and why not put one on every column?
2. How does B-tree work — why is it fast?
3. B-tree vs Hash index?
4. Give 3 cases where an index is NOT used.
5. Composite index on `(a, b)` — which of these use it?
   - `WHERE a = 1`
   - `WHERE b = 2`
   - `WHERE a = 1 AND b = 2`
6. Clustered vs Non-clustered.
7. What is a covering index?
8. Walk through the "speed up slow query" playbook.
