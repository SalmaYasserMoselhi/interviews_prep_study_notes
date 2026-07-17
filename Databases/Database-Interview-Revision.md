# 🗄️ Database Interview Revision — Single Source of Truth

> **How to use:** read the hook 🎣 → scan the notes → check the trap ⚠️.
> If you can say the hook + the trap, you can answer the question.
> **T1** = must nail · **T2** = should know · **T3** = recognize the term

---

## ⚡ 60-Second Cram (read this last, before you walk in)

```
EXEC ORDER    From -> Where -> GroupBy -> Having -> Select -> OrderBy -> Limit
WHERE vs HAVING   where = rows before agg. | having = groups after agg.
JOIN TYPE     = your answer to "what happens to UNMATCHED rows?"
LEFT+WHERE    on right table = becomes INNER (NULL fails compare) -> put it in ON
INDEX         B+tree, O(n)->O(logn). costs WRITES + space. func on col = dead
COMPOSITE     LEFTMOST PREFIX. (a,b) serves a / a+b — NEVER b alone
ACID          Atomic=all-or-nothing | Consistent=rules | Isolated=no peek | Durable=WAL
ISOLATION     dirty=never existed | non-repeat=it CHANGED | phantom=it MULTIPLIED
N+1           1 for list + N for details. fix = eager load
NORMALIZE     the key, the whole key, nothing but the key
CAP           partition happens -> C or A. P is NOT a choice.
BASE          available then correct (ACID = correct then available)
REPLICA       same data copied  -> scales READS + safety
SHARD         diff data split   -> scales WRITES + volume. ZERO safety.
EMBED vs REF  "do I ever want the child WITHOUT the parent?"
```

**The one through-line:** everything hard here came from **one decision — leaving one machine.** 🖥️→🌍
One machine = ACID, joins, hard state, one truth. Leave it and you inherit BASE, broken joins, soft state, elections, split brain, shard keys, CAP.
👉 **Stay on one machine as long as you possibly can.**

---

## 🧭 The 3 Comparison Tables Interviewers Actually Ask

**Copy / Split / Split-on-one-box**

| | Replica 📋 | Partition 🖥️ | Shard ✂️ |
|---|---|---|---|
| data | **copied** | split | split |
| servers | many | **ONE** | many |
| each holds | everything | a piece | a piece |
| solves | reads + failure | table too big | data > 1 machine |
| redundancy | ✅ | ❌ | ❌ **none!** |
| lose one | 🟢 fine | 🔴 dead | 🔴 that slice gone forever |

> 🔑 partition + **NETWORK** = shard. Same op, different machines — **all the pain lives in that one difference.**
> 🔑 shard for **volume**, replicate for **safety**. You need **both**.

**Aggregate / Window / CTE**

```
AGGREGATE 🗜️  a FUNCTION.  5 rows -> 1 row.  rows GONE.    "what's the total?"
WINDOW    🪟  a FUNCTION.  5 rows -> 5 rows. rows STAY.    "how do I compare to it?"
CTE       🧺  a NAME.      rows unchanged.   holds a query. "organize / reuse"
```
> only diff btw agg + window = the word **`OVER`**. `PARTITION BY` = GROUP BY that doesn't collapse.
> CTE isn't an alternative — you put a **window INSIDE a CTE** to filter it (windows have no HAVING).

**Lifespan sort (kills all the "what's the difference" confusion)**

```
this query  ->  subquery 🪆 | CTE 🧺
this session->  temp table 📋 (and you can INDEX it!)
forever     ->  view 👓 | mat.view 💾 | procedure ⚙️ | function 🧮 | trigger ⚡
```
> ⚙️ procedures **PERFORM** (CALL it, can't use in SELECT) · 🧮 functions **PRODUCE** (usable in SELECT)

---
---

# Phase 1 — SQL Foundations

## 1. Relational model & RDBMS basics — T2

🎣 *Rows are the what, columns are the shape.*

**DBMS** -> sw giving client (dev/user) an interface to manage db + data via queries.
Replaced file-based sys due to its issues -> more organized + centralized, **single source of truth**.
Contains: RDBMS & non-relational.

**RDBMS** -> fixed schema, tables/rows/cols, relations btw entities, keys + constraints, **ACID**.
ex: MySQL, Oracle, PostgreSQL

**Database** -> organized collection of data + metadata, on disk, accessed thru a DBMS.

**File-based sys issues** -> incompatible types btw dept., redundancy, inconsistency, no propagation, multiple sources, accessibility, non-scalable, **no ACID**
-> DBMS fixed: single source of truth, managed access, consistency, scalable, indexing
-> google docs case: no AD, each keystroke is a transaction -> IC applied by backend sys
-> new file systems solved old techniques but **headache is still scalability**

**Vocab (they WILL ask):**
```
relation  = table        tuple     = row       attribute = column
DEGREE    = num of COLS  CARDINALITY = num of ROWS   ← most-mixed-up pair
domain    = allowed values for an attribute
```
Properties of a relation -> **atomic** cells (= 1NF!), same domain per col, **unique rows** (= why you need keys), order insignificant.

**Data abstraction levels:**
```
physical -> where data is stored     ex: indexes on balance table
logical  -> schema + relationships   ex: structure of balance table
view     -> user's slice             ex: user sees HIS balance, not all balances
```

**E-R model** -> shows entities + relations. entity = row · entity type = schema/attributes · entity set = all rows of that entity.
**Relationships:** 1-1, 1-M, M-M
**Intension vs extension:** intension = the **schema/metadata** (stable) · extension = the **data currently in it** (changes every insert).
**2-tier vs 3-tier:** 2 = client talks to db directly · 3 = client -> server -> db

🎤 **DBMS vs RDBMS?** DBMS stores/manages data (often as **files**), no formal relationships. RDBMS = DBMS + data as **related tables** + **keys** + **referential integrity** + **ACID**. Every RDBMS is a DBMS, not vice versa.
🎤 **Is Excel an RDBMS?** No — table-shaped, but **no enforced relationships, no constraints, no concurrency control, no ACID**.

========================================================

## 2. Keys — T1

🎣 *Super key = any lock that fits. Candidate = the smallest such lock. Primary = the one you hand out.*

**The funnel:**
```
SUPER KEY      -> ANY col-set that's unique (may carry REDUNDANT extra cols)
   ↓
CANDIDATE KEY  -> a MINIMAL super key (drop any col -> stops being unique)
   ↓
PRIMARY KEY    -> the ONE candidate you chose. NOT NULL + UNIQUE. one per table.
ALTERNATE KEY  -> the candidates you DIDN'T pick
```
-> **PK and AK are both candidates. PK is the one who won.**

**The rest:**
```
FOREIGN KEY -> points at another table's PK. enforces REFERENTIAL INTEGRITY
COMPOSITE   -> any key made of 2+ cols
UNIQUE KEY  -> PK-like BUT allows NULL + MANY per table
```

ex: `students(student_id, national_id, first_name, last_name)` w/ two Saras
```
candidate keys -> {student_id}, {national_id}       ← minimal + unique
super (non-cand)-> {student_id, first_name}         ← unique but REDUNDANT col
NOT a key      -> {first_name}                      ← two Saras
pick student_id as PK -> national_id becomes the ALTERNATE key
```

⚠️ **PK vs UNIQUE (the #1 asked):**
```
PK     -> NO NULL  | EXACTLY ONE per table | the row's identity
UNIQUE -> allows NULL | MANY per table     | a secondary "don't repeat" rule
```
-> *a unique key is a PK that forgives NULLs and doesn't mind siblings*

⚠️ **Can a FK be NULL?** **YES** = "no relationship yet". only must match a real PK if NOT null.
⚠️ **Can a PK be composite?** **YES** ex: `PRIMARY KEY (student_id, course_id)`

```sql
CREATE TABLE orders (
    order_id   INT PRIMARY KEY,
    student_id INT,
    FOREIGN KEY (student_id) REFERENCES students(student_id)
);
INSERT INTO orders VALUES (102, 9, 75);  -- ❌ ERROR: student 9 doesn't exist
```

🎤 **PK vs FK?** PK uniquely identifies a row **in its own table** (NOT NULL, UNIQUE, one per table). FK **references another table's PK**, creating the relationship + enforcing **referential integrity**.
🎤 **Super vs candidate?** Super = unique but **may have redundant cols**. Candidate = **minimal** super key. Every candidate is a super key, not vice versa.

========================================================

## 3. SQL command categories — T2

🎣 *Define / manipulate / control access / control the transaction.*

```
DML (insert, update, delete, select)   -> DATA level.      NOT auto-commit -> ROLLBACKABLE ✅
DDL (create, alter, drop, truncate)    -> STRUCTURE.       AUTO-COMMIT ❌ no rollback
DCL (grant, revoke)                    -> ACCESS level
TCL (commit, rollback, savepoint)      -> TRANSACTIONS
```
**savepoint** -> name a point to roll back to (partial undo, keeps earlier work)

> some texts split SELECT into **DQL** — mention it, treat as DML subset.

🔑 **TCL only ever wraps DML.** You never roll back a CREATE — it committed itself instantly.

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- DML
  SAVEPOINT after_debit;                                     -- TCL
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
ROLLBACK TO after_debit;   -- undo only the credit
COMMIT;
```

⚠️ **WHY does DDL auto-commit?** bec. schema is a **shared contract** — other sessions read it. Letting it sit uncommitted = they'd read a table that might vanish on rollback = chaos. DML only touches YOUR rows in YOUR transaction -> safe to keep undoable.

🎤 **Why can you roll back DELETE but not TRUNCATE?** DELETE is **DML** — runs in a transaction, **logs each row**. TRUNCATE is **DDL** — **auto-commits**, deallocates pages with no per-row logging -> nothing to roll back.

========================================================

## 4. Core querying — T1

🎣 *You write SELECT first, the db runs it almost last — every confusion comes from that gap.*

**Execution order:**
```
From -> Where -> Group By -> Having -> [WINDOW] -> Select -> Order By -> Limit
```
-> **where can't use alias in Select** bec. where runs BEFORE select
-> **where can't use COUNT(*)** bec. where runs BEFORE group by (no groups yet)
-> **order by CAN use the alias** bec. it runs after select
-> **windows can't go in WHERE either** -> wrap in CTE

**Subqueries:**
```
non-correlated -> independent, runs ONCE
correlated     -> depends on outer query -> RE-RUNS PER OUTER ROW => slow (N²)
                  => mostly can be done by joins
```
```sql
-- non-correlated: inner runs once
WHERE salary > (SELECT AVG(salary) FROM employees);

-- correlated: re-runs per employee (references e1)
WHERE salary > (SELECT AVG(salary) FROM employees e2 WHERE e2.dept = e1.dept);
```

🎤 **Correlated vs non-correlated?** Non-correlated is **independent, runs once**, result substituted in. Correlated **references an outer column** -> **re-executes once per outer row** -> expensive, often replaceable by a **JOIN**.
🎤 **Why no alias in WHERE?** **Logical execution order** — WHERE runs **before** SELECT, so the alias doesn't exist yet.

========================================================

## 5. HAVING vs WHERE — T1

🎣 *WHERE talks to people. HAVING talks to crowds.*

```
all rows
   ↓
WHERE    -> filters INDIVIDUALS (raw cols: salary, age, dept)
   ↓
GROUP BY -> builds the clusters
   ↓
HAVING   -> filters CLUSTERS (aggregates: COUNT(*), AVG(salary))
```
where -> filters raw cols **before** grouping
having -> filters clusters outputted **from** group by

**NOTE:** can use HAVING **without** GROUP BY — whole table is treated as **one big group**.

⚠️ **The trap — a row-level condition in HAVING:**
```sql
-- ❌ groups every dept, THEN throws Eng away (wasted work)
GROUP BY department HAVING department <> 'Eng';
-- ✅ discards Eng rows BEFORE grouping
WHERE department <> 'Eng' GROUP BY department;
```
🔑 **RULE: filter as early as you can.** raw col of one row -> **WHERE**. aggregate of a group -> **HAVING**.

**NOTE:** WHERE doesn't just filter — **it changes what the aggregates SEE**. drop a 28k salary and the dept avg moves.

🎤 **Why can't you write `WHERE COUNT(*) > 5`?** WHERE runs **before GROUP BY** — no groups exist yet, nothing to count. That's what HAVING is for. **HAVING is the WHERE clause for groups.**
🎤 **Perf difference?** Yes — a row-level filter in HAVING still works but the db **builds groups it then discards**. WHERE reduces the row set first.

========================================================

## 6. Joins — T1

🎣 *Normalization takes it apart, joins put it back — the join TYPE is just your policy on lonely rows.*

```
INNER -> throw the lonely rows away
LEFT  -> keep the lonely ones from the LEFT  (NULL-pad the right)
RIGHT -> keep the lonely ones from the RIGHT
FULL  -> keep BOTH  (= left + right + inner)
CROSS -> cartesian. 3 rows A × 4 rows B = 12 (most of time WRONG)
SELF  -> join with itself by ALIASES  ex: employee -> manager_id
```
🔑 **the ONLY question a join asks: "when a row has no partner, what do I do with it?"**

**Joins hack:** feel that there's a **master table** -> use LEFT. otherwise INNER.
> ex: "all customers **and** their orders if any" -> LEFT.
> SELF JOIN for org chart -> use **LEFT** (the CEO has no manager, keep them).

**Flow of multiple joins:**
```
1. check column name needed
2. get the table it's in, figure out the FK
3. left join ..... as v  on ..Id = v.Id
4. select ..columnName
```
**NOTE:** better to use **explicit JOIN ... ON**, never comma-joins in the FROM clause.

⚠️ **TRAP 1 — where clause can make LEFT behave like INNER:**
```sql
-- ❌ LEFT JOIN ... WHERE o.amount > 40   -> Milli's NULL fails the compare -> DROPPED
-- ✅ LEFT JOIN ... ON c.id = o.customer_id AND o.amount > 40   -> Milli survives
```
🔑 **ON decides what MATCHES. WHERE decides what SURVIVES — and WHERE runs after.**
-> condition on the **right (optional)** table -> **ON**. condition on the **left (preserved)** table -> **WHERE**.

⚠️ **TRAP 2 — duplicate rows after a JOIN (fan-out):**
Sara has 2 orders -> Sara appears **twice**. **Not a bug — it's the 1-M relationship showing its shape.**
-> ❌ `DISTINCT` **masks** it and can silently break your SUM
-> ✅ **aggregate the M side first** (join to a pre-summed subquery), or join at the right **grain**

**Join execution (how the engine physically does it — the planner picks, you see it in EXPLAIN):**
```
NESTED LOOP -> for each left row, scan right. good: small sets / right is INDEXED
HASH JOIN   -> build hash table on one side, probe w/ other. good: big UNSORTED
MERGE JOIN  -> both sides SORTED on the key, walk in lockstep. good: sort already exists
```

🎤 **When does LEFT behave like INNER?** When a condition on the **right table** sits in **WHERE**. The join NULL-pads unmatched left rows, then WHERE runs **after** and `NULL > x` is **not true** -> filtered out. Fix: move it to **ON**.
🎤 **Duplicates after a join — add DISTINCT?** No. It's **one-to-many fan-out** — correct behavior. DISTINCT **hides** it and can produce **wrong aggregates**. Aggregate before joining, or join at the right grain.
🎤 **CROSS vs FULL?** CROSS has **no condition** — cartesian product. FULL **has** a condition — matched pairs **plus** unmatched from both sides, NULL-padded.

========================================================

## 7. One-to-many query example — T2

🎣 *Join to spread out, group to squash back down.*

```
JOIN      -> bring the tables together
GROUP BY  -> pile the orders per customer
AGGREGATE -> crush each pile into one number
```

```sql
SELECT    c.name, COALESCE(SUM(o.amount), 0) AS total_spend
FROM      customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY  c.id, c.name;
```
```
Sara 125 | Ansh 120 | Milli 0    ← Milli never ordered
```

⚠️ **The 3 decisions that make or break it:**
```
LEFT not INNER    -> keeps Milli. inner would SILENTLY drop her (no error, just a missing row)
COALESCE          -> SUM of nothing = NULL, not 0. she'd show blank
GROUP BY c.id     -> not just name! two customers named Sara would MERGE
```

**COALESCE** -> returns the **first non-NULL**. SQL's "or else".
```sql
COALESCE(NULL, 5) -> 5      COALESCE(10, 20) -> 10 (stops at first non-null)
```
-> also stops NULL **contaminating** arithmetic: `100 * NULL = NULL`, not 100.

========================================================

## 8. UNION vs UNION ALL — T2

🎣 *JOIN widens, UNION stacks — and UNION ALL doesn't stop to check for twins.*

```
joins  -> side by side (more COLUMNS)
unions -> stacked      (more ROWS)
```
```
union     -> no duplication
union all -> with duplicates. FASTER bec. union does operations to remove dupli.
intersect -> rows in BOTH
minus/except -> rows in first but not second
```

⚡ **The perf answer they want:** UNION does an implicit **DISTINCT** -> requires a **sort/hash pass**. UNION ALL just **appends**.
-> **default to UNION ALL** unless you can name a reason to dedup.

**Rules to stack:** same **number of cols**, compatible **types**, same **order**. Names needn't match (first query's win).

**NOTE:** `MINUS` = Oracle · `EXCEPT` = standard/Postgres

🎤 **Which is faster and why?** UNION ALL — UNION must **sort or hash all rows** to find duplicates. UNION ALL simply appends.
🎤 **JOIN vs UNION?** JOIN = **horizontal**, matches rows on a condition -> **wider** rows, needs a **related column**. UNION = **vertical**, stacks -> **more** rows, needs **matching structure**.

========================================================

## 9. Finding / removing duplicates — T2

🎣 *Find by value, delete by id — mix them up and you delete the original too.*

**FIND** -> the pattern you memorize once:
```sql
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;
```

**DELETE (keep one):**
```sql
-- classic: keep the earliest
DELETE FROM users WHERE id NOT IN (SELECT MIN(id) FROM users GROUP BY email);

-- modern: ROW_NUMBER (more flexible — change ORDER BY to keep the NEWEST)
WITH ranked AS (
  SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn FROM users
)
DELETE FROM users WHERE id IN (SELECT id FROM ranked WHERE rn > 1);
```

⚠️ **THE TRAP:**
```sql
-- ❌ deletes BOTH copies — Sara vanishes entirely
DELETE FROM users WHERE email IN (SELECT email FROM users GROUP BY email HAVING COUNT(*)>1);
```
🔑 **detection works on the VALUE. removal must work on the ROW IDENTITY (id).**

**The real fix** -> duplicates are a **symptom**. the disease is a missing constraint:
```sql
ALTER TABLE users ADD CONSTRAINT uq_email UNIQUE (email);
```
> `DISTINCT` fixes the **display**. a constraint fixes the **data**. it's makeup, not surgery.

========================================================

## 10. DELETE vs TRUNCATE vs DROP — T1

🎣 *DELETE picks, TRUNCATE empties, DROP demolishes.*

```
drop     -> delete data + STRUCTURE
truncate -> delete data (structure stays)
delete   -> delete rows per WHERE. without where = same as truncate BUT rollbackable + slow
```
```
 truncate -> DDL, drops all data, leaves structure, AUTO-COMMIT
 delete   -> DML, drops what meets where, ROLLBACKABLE
 DML are rollbackable, DDL mostly not
```

| | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| removes | rows | all rows | table + structure |
| category | DML | DDL | DDL |
| WHERE? | ✅ | ❌ | ❌ |
| rollback? | ✅ | ❌ | ❌ |
| speed | 🐢 | ⚡ | ⚡ |
| fires triggers? | ✅ | ❌ | ❌ |
| resets identity? | ❌ | ✅ | n/a |

⚡ **WHY is truncate faster? (the real question)**
DELETE walks **row by row** -> checks WHERE, writes **each removal to the transaction log** (so it can undo), fires **triggers**, leaves space allocated.
TRUNCATE **deallocates the data pages wholesale** — no per-row logging.
🔑 **the speed and the irreversibility are the SAME FACT seen from two sides.**

⚠️ **TRUNCATE resets the auto-increment counter.** DELETE doesn't. bites hard if other tables still reference old ids.
⚠️ **DROP takes constraints, indexes, FK relationships, and grants with it.** TRUNCATE keeps all of it.
⚠️ **TRUNCATE on a table with FKs pointing at it -> most engines REFUSE.** It can't check rows one-by-one for referential integrity (that's part of why DELETE is slow). The error is the db protecting you.

========================================================

## 11. Views + materialized views — T2

🎣 *A view saves the QUESTION. A materialized view saves the ANSWER.*

```
regular view      -> stores ONLY the query (re-runs EVERY access)
materialized view -> stores query + RESULT (shows saved data instantly)
```
**VIEW** = a saved SELECT wearing a table costume. **stores no data.**

| | View 👓 | Mat. View 💾 |
|---|---|---|
| stores data? | ❌ | ✅ |
| runs query | every access | only on **REFRESH** |
| read speed | query speed 🐢 | instant ⚡ |
| freshness | always current ✅ | can be **STALE** ⚠️ |
| costs | CPU per read | storage + refresh |

**mat.view adv/disadv:** adv = fast · disadv = unupdated data + more storage
`REFRESH MATERIALIZED VIEW customer_totals;`

**Views are good for:**
```
SIMPLIFY -> hide gnarly joins behind a name
SECURITY -> expose name/city, hide salary/ssn. GRANT on the VIEW not the table (least privilege)
CONSISTENCY -> one definition of "active customer", used everywhere
```

⚠️ **When to AVOID a view:**
```
-> heavy query + accessed FREQUENTLY -> you pay full cost EVERY read, hidden from devs
-> it MASKS bad SQL -> perf problems get harder to spot
-> for repeated heavy reads use a MATERIALIZED VIEW or caching instead
```

⚠️ **"Views improve performance"** = ❌ **NO.** The query runs every time. The plan may be cached, but the **work still happens**.
> *the plan is the recipe. a view saves you re-writing the recipe, not re-cooking the meal.* Only a mat.view keeps the cooked meal on the shelf.

**Choose by one question:** *can this data be a few minutes old?*
-> yesterday's revenue dashboard = mat.view ✅ · current account balance = view ✅

---
---

# Phase 2 — Schema Design & Integrity

## 12. Normalization + denormalization — T1

🎣 *The key, the whole key, and nothing but the key.*

**normalization** -> organize data so it **dec. redundancy**, by following rules, but **inc. joins**
**denorm.** -> vice versa

```
1NF -> ATOMIC vals (no composite/repeating records)
2NF -> no PARTIAL depen.  (non-key col dep on PART of the composite PK -> new table)
3NF -> no TRANS. depen.   (non-key col dep on a NON-KEY col -> new table)
BCNF-> every determinant must be a SUPER KEY (see T13)
```
**NOTE:** 2NF only bites with a **COMPOSITE key**. single-col PK -> automatically 2NF.
**NOTE:** 1NF is just the relational model's own **atomic cell** rule (T1).

**What it prevents — name these:**
```
UPDATE anomaly -> Sara's address in 40 rows, miss one -> TWO CONTRADICTORY TRUTHS
INSERT anomaly -> new student with no books yet -> nowhere to put them
DELETE anomaly -> Ansh returns his only book -> you deleted ANSH
```

**NOTE:**
```
read heavy system (dashboards) -> DENORMALIZATION
write heavy system (banking)   -> NORMALIZATION
```

⚠️ **"Normalization improves performance"** = ❌ It improves **INTEGRITY**. It usually makes **reads slower** (more joins) and **writes safer**.
🔑 **normalization trades read speed for correctness.**
🗣️ mature answer: *"normalize by default, denormalize deliberately with a measured reason."*

💡 **why anomalies matter more than they sound:** an update anomaly isn't messy — your db now holds **two contradictory truths with no error and no alert**. Worse than a crash: a crash you notice. Silent inconsistency **rots quietly** until someone ships to the wrong house. Normalization makes contradiction **structurally impossible**, not "something we're careful about."

🎤 **When is a table automatically 2NF?** When its PK is a **single column** — partial dependency needs **part of a composite key**.

========================================================

## 13. BCNF & higher forms — T2

🎣 *3NF protects non-key cols. BCNF makes no exceptions — if it determines, it must be a super key.*

**BCNF (= 3.5NF)** -> for **EVERY** dependency `X -> Y`, **X must be a SUPER KEY**. Full stop.
-> closes 3NF's loophole: 3NF **forgives** a **prime attribute** (part of a candidate key) determining something.

**The classic example — 3NF but NOT BCNF:**
```
class_schedule(student, subject, teacher)
rule: each TEACHER teaches exactly ONE subject
candidate keys: {student,subject} and {student,teacher}
-> every attr is PRIME -> passes 3NF ✅
-> BUT: teacher -> subject, and teacher ALONE is not a super key ❌ BCNF VIOLATION

damage: Ansh drops Math -> you LOSE the fact "Mr. Ali teaches Math" = DELETE ANOMALY
fix: split -> teaches(teacher, subject) + enrolled(student, teacher)
```

**Higher forms (name-level only):**
```
4NF -> removes MULTI-VALUED dependencies (2 independent multi-valued facts crammed together)
5NF -> removes JOIN dependencies (rare, academic)
```

⚠️ **Why not always BCNF?** BCNF decomposition can **break dependency preservation** — you'd need a join to enforce a rule. **3NF is always achievable while preserving all dependencies. BCNF isn't.**
🎯 **3NF is the practical target. BCNF when it's cheap. 4NF+ is textbook.** ← saying this shows judgment.

========================================================

## 14. Constraints vs validations — T2

🎣 *Validation asks nicely. Constraints don't ask.*

```
validation  -> APP level, for UX (good: not hitting db too much. BUT can fall in RACE COND.)
constraint  -> DB level, for DATA INTEGRITY
```
> nightclub: validation = the friendly guy checking IDs 🙏 · constraint = **the wall** 🧱

**The constraints:**
```
NOT NULL    -> value must exist
UNIQUE      -> no duplicates
PRIMARY KEY -> NOT NULL + UNIQUE, one per table
FOREIGN KEY -> must reference an existing row
CHECK       -> must satisfy a condition   ex: CHECK (age >= 18)
DEFAULT     -> auto-fill when omitted
```
> notice PK is just NOT NULL + UNIQUE wearing a badge.

⚠️ **WHY app validation can't replace a constraint — THE RACE:**
```python
if not User.objects.filter(email=email).exists():   # check
    User.objects.create(email=email)                # create
```
```
Req A: check -> "free" ✅
Req B: check -> "free" ✅   (A hasn't inserted yet!)
Req A: insert -> OK
Req B: insert -> OK  💥 DUPLICATE
```
🔑 there's a **GAP between check and act**. Only the db can close it — a **UNIQUE constraint is enforced atomically at write time**, no window.
> same shape as the **race conditions** from multithreading — check-then-act across concurrent actors.

**Also bypassed by:** migration scripts, an admin in psql at 2am, another service, the mobile team's backend.

🗣️ **the line:** *"validation is for **UX**, constraints are for **data integrity**. validation tells the user what's wrong — constraints make wrong data **impossible**."*
-> use **BOTH**. same rule, two layers, two audiences.

========================================================

## 15. Referential integrity & cascading deletes — T2

🎣 *A foreign key is a promise. ON DELETE is what happens when you'd break it.*

**what if del. cust. that has orders?**
**Referential integrity** -> promise that every FK points to a **real row / NULL**, not garbage (a **ghost** 👻 = orphan row).

⚠️ **enforced in BOTH directions:**
```sql
INSERT INTO orders VALUES (3, 99);    -- ❌ can't point at a ghost (customer 99 doesn't exist)
DELETE FROM customers WHERE id = 1;   -- ❌ can't CREATE one (still referenced)
```

**the way to let the db work when delete of parent is needed: `ON DELETE` while creating the table**
🔑 **ON DELETE does NOT break integrity — every policy KEEPS the promise.** It's *how* you delete a parent while still honoring it.

```
RESTRICT / NO ACTION -> can't delete a parent that has children     ← DEFAULT
CASCADE   -> del. parent AND children
SET NULL  -> del. parent, FK for child = NULL   (col must be NULLABLE)
SET DEFAULT -> FK = the column's default value
soft del. -> is_deleted = true. nothing ever truly gone.
```
```sql
CREATE TABLE orders (
    id          INT PRIMARY KEY,
    customer_id INT REFERENCES customers(id)
                ON DELETE CASCADE     -- ← the policy
);
```

**CHOOSING — one question:** *"does the child mean anything without the parent?"*
```
post -> comments      = CASCADE    (a comment on a deleted post is nonsense)
department -> emps    = SET NULL   (THE EMPLOYEE STILL EXISTS! never cascade — you'd fire them)
customer -> orders    = RESTRICT   (financial records. be LOUD.)
```

⚠️ **CASCADE CHAINS 🌊:** `customers --CASCADE--> orders --CASCADE--> order_items`
-> `DELETE FROM customers WHERE id=1` = 1 customer + 40 orders + **380 order_items**. One line. No warning.
🔑 **trace the whole chain first.** The scariest cascades are **two hops away** that nobody remembered.
-> production default: **RESTRICT** (make deletion deliberate) or **soft deletes**.

💡 **Django's `on_delete=` is literally this — and it's MANDATORY because there IS no safe default.** Comments cascade, employees SET NULL, invoices RESTRICT. A framework guessing would silently destroy data half the time. It refuses to guess.

---
---

# Phase 3 — Transactions & Concurrency

## 16. ACID — T1

🎣 *Atomicity says "all steps." Consistency says "valid rules." Isolation says "no peeking." Durability says "no take-backs."*

```
Atomicity   -> ALL OR NOTHING (via ROLLBACK)
Consistency -> state of db before & after the transaction still OBEYS THE RULES
               ex: balance >= 0. (about RULES, not about nodes)
Isolation   -> no other trans. can change data while trans. running,
               depending on the LEVEL of isolation
Durability  -> after commit, data PERSISTS thru sys failure / power cut. preserved on disk.
               HOW? the WRITE-AHEAD LOG (WAL) — log written BEFORE the data files,
               crash -> REPLAY the log
```

⚠️ **DON'T CONFUSE (your own note, keep it):**
```
"C in ACID"  -> the DATA obeys the RULES (constraints)
"C in CAP"   -> all NODES/replicas AGREE with each other (propagation)
```
**Two totally different C's.** Interviewers love catching this.

⚠️ **Atomicity vs Consistency:**
```
Atomicity   -> about the TRANSACTION. "did all steps run?"     = EXECUTION
Consistency -> about the DATA.        "are the rules intact?"  = VALIDITY
```
-> you can be atomic but inconsistent: all steps ran, but balance = -500 (a CHECK should've stopped it).

💡 **Why is Isolation the only one with LEVELS?** Because it's the only one with a **price you can feel**. A, C, D are basically free (you have them or your db is broken). Isolation is a **DIAL** — up for safety, down for speed. That's why it has levels and the others don't.

🎤 **How is Durability implemented?** **Write-Ahead Logging** — changes go to a **sequential log on disk BEFORE** the data files. Crash -> **replay the log**. Sequential log writes are much faster than random data-file writes, which is why it's both safe and practical.

========================================================

## 17. Isolation levels — T1

🎣 *Dirty = never existed. Non-repeatable = it CHANGED. Phantom = it MULTIPLIED.*

```
Read Uncommitted -> blocks nothing   ⚡ fastest, wild west
Read Committed   -> blocks 1         ← PostgreSQL DEFAULT
Repeatable Read  -> blocks 2         ← MySQL/InnoDB DEFAULT
Serializable     -> blocks all 3     🐌 slowest, bulletproof
```

**1. Dirty read**
```
A: UPDATE balance = 500   (not committed yet)
B: SELECT balance -> 500  👀 reads it!
A: ROLLBACK               ← never happened
B acted on data that NEVER EXISTED. 💀
```
**2. Non-repeatable read**
```
A: SELECT balance -> 100
B: UPDATE balance = 200; COMMIT
A: SELECT balance -> 200   😵 same query, different result
```
**3. Phantom read**
```
A: SELECT COUNT(*) WHERE age > 18 -> 5
B: INSERT a new 20-year-old; COMMIT
A: SELECT COUNT(*) WHERE age > 18 -> 6   👻 a row appeared
```
🔑 **non-repeatable = an EXISTING row CHANGED (UPDATE). phantom = a NEW row APPEARED (INSERT).**
> phantom = new rows sneak into your range the second time you look.

| level | dirty | non-repeat | phantom |
|---|---|---|---|
| Read Uncommitted | ❌ | ❌ | ❌ |
| Read Committed | ✅ | ❌ | ❌ |
| Repeatable Read | ✅ | ✅ | ❌ |
| Serializable | ✅ | ✅ | ✅ |

⚠️ **"just use Serializable, it's safest"** = naive. transactions effectively **QUEUE** -> throughput collapse, more **deadlocks**, and **serialization failures your app must RETRY**.
🗣️ **mature:** *"default Read Committed. raise it only for the transactions that need it. and handle retries at Serializable."*
-> you pick **per transaction**, not per app.

💡 **PostgreSQL's Read Committed uses NO read locks at all** — it uses **MVCC** (multi-version concurrency control). Each transaction sees a **snapshot**; the engine keeps **multiple versions** of each row. **Readers never block writers, writers never block readers.** MVCC is why modern isolation is affordable.
💡 InnoDB's Repeatable Read also prevents most **phantoms** via **next-key locking** — going beyond the SQL standard.

========================================================

## 18. Locking & deadlocks — T2

🎣 *Pessimistic locks the door. Optimistic checks if anyone came in. Deadlock = two people each holding the other's key.*

```
-> isolation is the PROMISE
-> locks are the MECHANISM
```
**lock** -> used for **concurrency control**
```
shared lock 📖    -> READ.  MULTIPLE shared locks can coexist
exclusive lock ✍️ -> WRITE. only ONE can exist, blocks everything else
```
🔑 **reads share, writes don't.**

**pessimistic: lock then work** ("conflict WILL happen")
```sql
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;  -- 🔒 locked
UPDATE accounts SET balance = 500 WHERE id = 1;
COMMIT;                                          -- 🔓 released
```
**optimistic: work then check** ("conflict probably won't")
```sql
-- Read row, note version = 3
UPDATE accounts SET balance = 500, version = 4
WHERE id = 1 AND version = 3;    -- ← only if nobody changed it
-- 0 rows updated? Someone beat you. Retry. 🔁
```
```
HIGH contention (everyone hits the same rows) -> PESSIMISTIC  ex: last iPhone in stock
LOW  contention (conflicts rare)              -> OPTIMISTIC   ex: most web apps
```

**deadlock** -> 2 transactions waiting for each other's resource. engine **detects** it, chooses a **victim** and rolls it back.
```
A 🔒 Row1, wants Row2   ←→   B 🔒 Row2, wants Row1   = frozen forever 🪦
```
**HOW detected:** a **WAIT-FOR GRAPH** — who's waiting on whom. Find a **CYCLE** 🔄 = deadlock.

🛡️ **PREVENT — the big one:** **always acquire locks in the SAME ORDER** (e.g. ascending PK).
-> if everyone goes low->high, a cycle is **impossible**.
also: keep transactions **short**, lowest workable **isolation level**, touch **fewer rows**, and build **retry logic** (you can't eliminate deadlocks).

💡 **Why detect instead of prevent?** Prevention needs knowing **every lock upfront** — but transactions are **dynamic** (what you lock depends on what you read). So the engine can't plan ahead. **Detect-and-recover is cheaper than predict-and-prevent** — classic systems tradeoff: optimize the common case, handle the rare one when it bites.

---
---

# Phase 4 — Performance

## 19. Indexing — T1

🎣 *An index is the book's index — don't read 500 pages for one word.*

way to speed up search and retrieval instead of **scanning the whole table** (full table scan).

**-> index uses a B+ tree DS bec:**
```
=> stays SORTED and BALANCED
=> data is in the LEAF NODES, so connected/linked together -> RANGE QUERIES are fast
=> O(n) -> O(log n)     (10M rows -> ~23 hops, each hop HALVES the search space)
```

⚠️ **composite index => LEFTMOST PREFIX RULE** (name it — "must be in order" is too vague)
```
INDEX (last_name, first_name)  -> sorted by last_name FIRST, then first_name within
✅ WHERE last_name = 'Ahmed'
✅ WHERE last_name = 'Ahmed' AND first_name = 'Sara'
❌ WHERE first_name = 'Sara'          ← phone book: can't find all "Sara"
⚠️ order in your WHERE does NOT matter — the INDEX's column order does
```

**COVERING INDEX** -> index has **ALL cols the query needs** -> answers **from the index, never touches the table** = **INDEX-ONLY SCAN** 🎯
-> this is also why **`SELECT *` hurts** — asking for every col means no index can ever cover you.

⚠️ **if a func is used on an indexed col -> index IGNORED**
```sql
ex: WHERE Lower(email) = "mo@mail.com"    -- ❌ email has an index, but it's DEAD
    WHERE email = "mo@mail.com"           -- ✅
```
**WHY:** the index stores **RAW values, not function results** -> the condition is **NON-SARGABLE** -> full scan.
**FIX (Postgres):** an **expression/functional index** -> `CREATE INDEX ON users (LOWER(email));`

**disadv.:** DML becomes slower bec. it's done twice (table + index) + space
-> **10 indexes = one INSERT becomes 11 writes** 💥 (each a B-tree insert that may cause a **node split**)
-> that's why "index everything" is wrong. unused indexes cost every write and give back nothing.

🎤 **What's a covering index?** One containing **all columns the query needs**, so the db answers **entirely from the index** — an **index-only scan** — skipping the table lookup.

========================================================

## 20. Index types — T2

🎣 *Clustered = the data itself, sorted. Everything else is a signpost pointing at it.*

**Where do rows physically live bec. of index?**
```
-> CLUSTERED index (table's rows are SORTED on this index, usually PK, ONCE per table)
   ex: DICTIONARY — the index IS the data
-> NON-CLUSTERED index = SECONDARY INDEX (separate structure, MANY per table)
   ex: BOOK INDEX — a separate list pointing to data
```
> ⚠️ your note said *"secondary search field"* — the real term is **secondary index**.

🔑 **WHY only ONE clustered index?** bec. data can only be **PHYSICALLY SORTED ONE WAY**. That's the whole reason.

**The 2-step lookup (why non-clustered is slower):**
```
CLUSTERED     -> find in index -> 🎯 you're AT the row.        1 step
NON-CLUSTERED -> find in index -> pointer -> 🔍 fetch the row.  2 steps
                                              ↑ BOOKMARK LOOKUP (a.k.a. key lookup)
```
🔑 **a COVERING index ELIMINATES the bookmark lookup** ← ties T19 and T20 together.

**WHY clustered wins at ranges:** rows are **physically adjacent** -> `WHERE id BETWEEN 100 AND 200` = **one sequential block read** 📦. Non-clustered = 100 pointers -> **100 random I/O jumps** 🎲.

💎 **ENGINE DIFF (interview gold):**
```
MySQL/InnoDB -> the PK **IS** the clustered index, always
PostgreSQL   -> NO clustered indexes AT ALL. (CLUSTER cmd = one-time reorder, doesn't persist)
```
> most candidates assume clustered is universal. Knowing PG doesn't have them signals real experience.

**The types:**
```
unique    -> uniqueness (checked at INSERT) + speed. it's the SAME structure doing both:
             to CHECK a dup you must LOOK IT UP — uniqueness comes FREE with the search tree
composite -> mult. columns (LEFTMOST PREFIX)
covering  -> everything the query needs -> index-only scan
hash      -> O(1) for `=` ONLY. hashing DESTROYS order -> NO ranges, NO ORDER BY
partial   -> CREATE INDEX ... WHERE status='active'. 95% inactive? index is 20× smaller 🎯
bitmap    -> low-cardinality cols (gender, status)
full-text / geospatial / sparse-dense -> awareness
```

💡 **Why do random PKs (UUID) wreck InnoDB writes?** The clustered index must stay **physically sorted**. Sequential id -> new row goes at the **end**, cheap ✅. Random UUID -> it belongs **in the middle** -> **PAGE SPLIT**: the engine cracks a full page in two and shuffles rows 💥. **The thing that makes reads fast is what makes random writes expensive.** (fix: sequential ids, or UUIDv7 which is time-ordered)

========================================================

## 21. Query optimization & EXPLAIN — T1

🎣 *EXPLAIN is the db showing you its homework — stop guessing.*

to diagnose why an api/query is slow:
```
EXPLAIN         => shows the PLAN / strategy {ESTIMATED}. doesn't run it.
EXPLAIN ANALYZE => RUNS it, shows real timings {ACTUAL}   ← BEST TO USE
```
**output:**
```
❌ SEQ SCAN   -> scan table row by row              🐢
⚠️ INDEX SCAN -> used idx, then fetches the row
✅ INDEX ONLY -> answered from the index directly   ⚡
```

⚠️ **THE KILLER SIGNAL — estimate vs reality:**
```
rows=10 ... actual rows=48000
   ↑            ↑
planner thought  truth
```
**A big gap = the planner is working from STALE STATISTICS.** It chose a strategy for 10 rows and got 48,000. **Often the real cause — not a missing index.**
**FIX:** `ANALYZE tablename;` -> refreshes stats.

⚠️ **`Rows Removed by Filter`** is the number to hunt. High = reading data just to throw it away.

🛠️ **The optimization checklist — in order:**
```
🔍 EXPLAIN ANALYZE first — never guess
📇 Index the cols in WHERE / JOIN / ORDER BY
🚫 Avoid SELECT * — fetch only what you need (enables covering indexes)
📊 Check estimate vs actual — run ANALYZE if they diverge
✂️ Filter early — push conditions into WHERE
🧼 Keep columns bare — no functions on indexed columns
🔁 Replace correlated subqueries with joins where possible
```

⚠️ **"It says Seq Scan — I need an index!"** = ❌ not always.
```
small table (few hundred rows) -> seq scan is genuinely FASTER (less overhead, sequential I/O)
query returns MOST of the table (>~30%) -> seq scan wins anyway
```
🗣️ **mature:** *"a seq scan is only a problem when it scans **a lot of rows to return a few**."*

🎤 **Planner ignores my index — why?** A **function wraps the column** (non-sargable) · a **leading wildcard** (`LIKE '%x'`) · query returns **too much of the table** · table is **small** · **stale statistics** (run ANALYZE) · the **leftmost prefix** of a composite index isn't in the WHERE.

💡 **Why does a good index still get a bad plan?** The planner doesn't look at your **data** — it looks at a **summary** (row counts, distribution). Bulk-insert a million rows without `ANALYZE` and it still thinks the table has 100 -> confidently picks a **nested loop** that takes an hour. **The planner is only as smart as its statistics.**

========================================================

## 22. N+1 query problem — T1

🎣 *N+1 is asking the db a hundred questions when one would do.*

```
1 -> for the list
N -> for each item's details
```
ex: books list, 10 books, get author name -> **hits the db 11 times**
-> exists due to **normalization** of data (+ the ORM's **LAZY LOADING**)
-> instead of getting all books then querying each for the author -> **get books JOINED with author**

```python
posts = Post.objects.all()      # 1 query
for post in posts:
    print(post.author.name)     # 💥 1 query EACH -> 101 total
```

🔑 **the cost is the NUMBER OF TRIPS, not the work per trip.** Each query is FAST — that's what makes it sneaky. **Nothing shows in the slow-query log.**

**FIX — eager loading:**
```python
Post.objects.select_related('author')     # 1 query, a JOIN     -> FK / OneToOne (many->one)
Post.objects.prefetch_related('tags')     # 2 queries, an IN    -> M2M / reverse FK (one->many)
```
```
Sequelize -> Post.findAll({ include: [User] })
Rails     -> Post.includes(:author)
Laravel   -> Post::with('author')->get()
Mongoose  -> Post.find().populate('author_id')
```
⚠️ **prefetch_related does NOT use a JOIN.** It runs a **2nd query with `WHERE id IN (1,2,3...)`** and joins **in Python**. Deliberate — a 1-M JOIN would **fan out** and duplicate the parent (T6!).

⚠️ **"just always use select_related"** = ❌ it JOINs -> every joined col comes back on **every row**. A 5-table select_related = huge redundant result sets.
🔑 **101 small queries is bad. one monstrous 8-table JOIN is also bad. eager-load exactly what you'll use.**

**How to catch it:** django-debug-toolbar (query count per page — 100+ 🚩) · log queries (same shape repeating with different ids) · **query count scales with row count**.

💡 **Why is it the most common perf bug in existence?** It's **invisible in dev**. 10 posts -> 11 queries -> 15ms -> feels fine ✅. Prod: 10,000 posts -> 10,001 queries -> **30 seconds** 💀. It doesn't fail, it **degrades linearly with your data** — so it gets worse exactly as you get successful. It ships because **the code looks correct**. It *is* correct — it's just asking the wrong question 101 times.

========================================================

## 23. Connection pooling — T2

🎣 *Don't hire a new taxi for every trip — keep a few idling out front.*

**how to open a connection with db?**
```
tcp handshake -> auth -> server memory -> set up session state
= 50-100ms      ...and the req itself costs only 2ms 😱
```
**so not every req will do all of this** -> solution isn't open and close ==> **BORROW AND RETURN**

```
   ┌─────────── POOL ───────────┐
   │  🔌 🔌 🔌 🔌 🔌 (10 open)  │
   └────────────────────────────┘
         ↓ borrow      ↑ return
      Request A      Request A
   Connections stay ALIVE. Requests share them.

Request arrives
   ├── ✅ free conn? -> borrow, run query, return it
   └── ❌ none?      -> wait in queue ⏳ (or timeout)
```
-> the handshake happens **10 times total**, not 10,000/sec.

⚠️ **take care: inc. db conn. does NOT make the sys faster**
```
-> rdbms has LIMITS on num. of connections bec. each costs RAM (PG default max_connections = 100)
pool small => db queue is up ⏳
pool large => hit the conn. limit / fight on cores 🥊 / crash 💥
```
**Sizing:** `pool ≈ (CPU cores × 2) + spindles` -> **~10 for a 4-core box.** That's it.

💥 **THE MULTIPLICATION TRAP (the thing that bites):** pools live in **memory**, memory is **per-process**.
```
threads   -> share memory -> share ONE pool ✅
processes -> separate mem -> EACH gets its OWN pool ⚠️

4 gunicorn workers × pool 10 = 40 connections
+ 4 celery      × pool 10 = 40
                          = 80  ← PG max is 100 😰
× 3 app servers           = 240 -> FATAL: too many connections 💀
```
🔑 **total = processes × servers × pool_size.** Your pool size is **per-process**, not global.
**FIX:** **PgBouncer** — a standalone pooler that pools **across all processes**.

⚠️ **Pool exhaustion:** a connection borrowed and **never returned** = leaked. 20 leaks -> pool empty -> **every request hangs, no errors**. Fix: context managers / `finally`. (The ORM does this for you — which is why hand-rolling connection code is dangerous.)

🧵 **Is a pool multithreading or multiprocessing?** **Neither** — it's a **resource pool** serving whatever concurrency your app server uses. Internally it's a **thread-safe semaphore** guarding a bounded resource (acquire = borrow, release = return, count 0 = block).

💡 **Why does a SMALLER pool often beat a bigger one?** Connections don't create capacity — they create **queues**. 8 cores can truly run ~8 things. 200 connections = 8 cores juggling 200 contexts -> more **context switching**, **lock contention**, **memory pressure**, everything slower. With 10, the extra requests wait **in your app** (cheap) instead of thrashing **in the db** (expensive). **Queue at the door, not on the dance floor.**

========================================================

## 24. Speeding up retrieval — T2

🎣 *The fastest query is the one you never send.*

**How to make a query faster (it takes 30 sec)?**
**first WRONG sol:** check HW perspective (network) / upgrade db instance
```
=> it is ALMOST NEVER a HW issue, it's a CODE problem.
-> upgraded instance 30sec -> 25sec = $400/mo for 5 sec? 🤦
```
**Usual suspects:**
```
1) Missing indexes    30 sec -> 50 ms
2) N+1 problem
3) Bad joins or SELECT * when you need 3 columns
```

**FIX WAY — the ladder 🪜 (climb in order, cheapest first):**
```
1) know the problem FIRST -> EXPLAIN ANALYZE
2) right indexes (target -> where, joins, order by)
3) fix the query (✗ select * / ✗ correlated / filter early / PAGINATE)
4) caching:
     Request -> 💾 Redis? ─┬─ ✅ HIT  -> return (0.5ms) ⚡
                           └─ ❌ MISS -> 🗄️ DB (50ms) -> store -> return
5) read replicas (primary=writes, replicas=reads. watch REPLICATION LAG 🌊)
6) denormalization (if read system {dashboards})
7) partition (one server) -> then shard (many servers)  ← LAST RESORT
8) fix app layer (eager loading not lazy // batch queries not loops)
```
🔑 most problems **die on rung 1 or 2.**

⚠️ **"it's slow -> add caching!"** = ❌ caching a query that's slow from a **missing index** means you now have a **slow query AND a stale-data problem**, the cache **hides the bug**, every miss is still slow, and you own **invalidation forever**.
🔑 **fix the query first. cache the FAST query.**

⚠️ **read replica catch:** **read-your-own-writes** — user writes, gets redirected, reads a lagging replica -> *"MY CHANGE DIDN'T SAVE!"* It saved. They're reading the past. Fix: route that user's reads to the **primary** briefly after a write.

💡 **Why is cache invalidation famously hard?** The cache and the db are now **two sources of truth** with nothing syncing them. Update a price -> the cache doesn't know 🤷. So you invalidate manually on **every write path** — and if you miss **one** (an admin script, a batch job, another service), users see a **wrong price forever with no error**. The cache doesn't fail loudly. **It lies quietly.** That's why TTLs exist: an admission that you *will* miss a path, so at least be wrong for a **bounded time**.

========================================================

## 25. CTEs vs Subquery — T2

🎣 *CTEs hold. Subqueries nest.*

```
subquery -> read INSIDE OUT 🌀. can't reuse. no name.
CTE      -> a TEMP NAME for a query, to reuse it & read TOP TO BOTTOM ⬇️. can RECURSE 🌳
```
```sql
WITH ranked AS (
    SELECT name, ROW_NUMBER() OVER (...) AS rn FROM employees
)
SELECT * FROM ranked WHERE rn = 1;
```
⚠️ **Both are EQUALLY FAST.** A CTE is for **humans**, not speed. (older Postgres even **materialized** them = slower.)
🔑 **the one real power:** a CTE can be **referenced twice**; a subquery must be **written twice**.

**`WITH RECURSIVE`** -> walks **hierarchies** (org charts, category trees, comment threads).

**CTE vs VIEW:** CTE lives for **one statement**, nobody else sees it. A view lives **forever**, everyone can query it. -> **a view is a CTE you saved to disk.**

🔑 **THE PATTERN TO MEMORIZE: window INSIDE a CTE, filter OUTSIDE.** (windows have no HAVING — the CTE is the only place to put the result before filtering it.)

========================================================

## 26. Aggregate vs window functions — T2

🎣 *Aggregate crushes. Window keeps.*

```
aggregate -> 5 rows -> 1 row  ==> SUMMARIZE   "what's the total?"     rows GONE 👻
window    -> 5 rows -> 5 rows ==> COMPARE     "how do I compare?"     rows STAY ✅
```
**Same functions** (SUM/AVG/COUNT/MIN/MAX). **The only difference is the word `OVER`.**
🔑 **`PARTITION BY` = `GROUP BY` that doesn't collapse.**

```
name  | dept  | salary
 Sara | Eng   | 40000
 Ansh | Eng   | 35000
 Karan| Sales | 45000
```
```sql
-- Aggregate:
SELECT dept, AVG(salary) FROM employees GROUP BY dept;
 Eng   | 37500      ← Sara & Ansh gone 👻
 Sales | 45000

-- Window:
SELECT name, salary, AVG(salary) OVER (PARTITION BY dept) AS avg FROM employees;
 Sara  | 40000 | 37500     ← everyone survives ✅
 Ansh  | 35000 | 37500
 Karan | 45000 | 45000
```

**Choose:** *"do I still need the individual rows?"* no -> aggregate (a **report**) · yes -> window (a **comparison**)

**Where each is allowed:**
| | aggregate | window |
|---|---|---|
| SELECT | ✅ | ✅ |
| **WHERE** | ❌ | ❌ |
| **HAVING** | ✅ | ❌ **none!** |
| ORDER BY | ✅ | ✅ |
-> both blocked from WHERE (same **pipeline** reason). but aggregates get **HAVING** — **windows get nothing** -> **CTE**.

**`OVER ()` with EMPTY parens** = one window over **everything** -> grand total on every row (for % of whole).

========================================================

## 27. LIKE vs ILIKE (+ collation, scalar vs agg) — T3

🎣 *Wildcards kill indexes, collation decides what "equal" means.*

```
like  -> case SENSITIVE   ///  ilike -> case INSENSITIVE (PostgreSQL only)
%  -> any number of chars    'S%'   -> Sara, Sam
_  -> exactly ONE char       'S_ra' -> Sara, Sera
```
**MySQL has no ILIKE** — its `LIKE` is already case-insensitive by **collation**. Portable: `LOWER(col) LIKE ...`

⚠️ **LEADING WILDCARD KILLS THE INDEX:**
```
✅ LIKE 'Sara%'   -> uses index ⚡
❌ LIKE '%Sara'   -> FULL SCAN 🐢    (B-tree is sorted by the START of the string)
❌ LIKE '%Sara%'  -> FULL SCAN 🐢
```
-> need "contains"? that's **full-text search** (T30), not LIKE.

**collation:** rules on how text is **sorted and compared** {for ex diff. langs}
```
Case sensitivity -> 'A' vs 'a'   ← the big one
accent sens.     -> 'é' vs 'e'
```
⚠️ same query, different collation, **different answer** — and it affects **ORDER BY** too. Maddening bugs bec. the SQL looks correct.

```
scalar func: LOWER, UPPER, CONCAT, ROUND, COALESCE  => 1 row -> 1 row
agg. func:   sum, count, avg, min, max              => N rows -> 1 row
```
⚠️ **aggregates IGNORE NULLs — except `COUNT(*)`:**
```
COUNT(*)     -> 5   (counts ROWS)
COUNT(email) -> 3   (skips 2 NULLs) 🕳️
```

---
---

# Phase 5 — Advanced SQL / PostgreSQL

## 28. Stored procedures & triggers — T2

🎣 *Procedures wait to be called. Triggers fire themselves.*

**stored procedures** -> stored in db, precompiled callable sql, takes params (**in, out, inout**)
-> **CALL** it. reduces **network traffic** (1 call not 5 round trips), adds **security** (grant CALL without table access), one definition (no drift across services).
**functions** -> **return values**. usable **inside SELECT**.
🔑 **procedures PERFORM. functions PRODUCE.**

**triggers** -> auto. fired on DMLs to make logic
```
BEFORE -> validation, modify the row before it lands ✏️
AFTER  -> log, update related data 📝
```

⚠️ **both are INVISIBLE 👻** — your UPDATE mysteriously writes 3 other tables and nothing in your code says so. Worse: **trigger chains** 🌀.
🔑 **use triggers for AUDITING, not business rules.** Keep logic in the app layer where it's **visible and version-controlled**. (constraints > triggers where possible — simpler.)
> same "invisible side effect" problem as Django **signals**.

💡 **Why did stored procedures fall out of fashion?** They live **outside your git repo** — no code review, no diff, no CI, no rollback. Your app is version-controlled; your procedures are just *in the database*, changed by whoever had psql open. Teams moved logic into the app trading a little performance for a lot of **sanity**.

========================================================

## 29. Migrations / schema versioning — T3

🎣 *Migrations are git for your schema.*

**version control for incremental schema changes.** Each change = a numbered ordered file. DB tracks which ran.
`makemigrations` (generate) -> `migrate` (apply pending). Tools: Flyway, Liquibase, Django, Rails, Laravel.

**RULES:** forward-only + ordered · **NEVER edit an applied migration** (write a new one — history must be immutable) · include a rollback · test on a prod-like copy.

🎯 **THE ONE PATTERN — EXPAND/CONTRACT (zero downtime):**
during deploy, **old and new code run simultaneously**.
```
1. EXPAND   -> add the new col (NULLABLE)     ← old code unaffected ✅
2. MIGRATE  -> backfill + deploy new code     ← both work
3. CONTRACT -> drop the old col               ← only after all old code is gone
```
-> renaming in one step = old pods **crash instantly** 💀. Every migration must be **backward-compatible with the code currently running**.

========================================================

## 30. Full-text search indexing — T3

🎣 *LIKE '%word%' reads every page. FTS flips the book around.*

if I need to search the middle of a str ex: `'%abdelhaq%'`
-> **btree fails bec. the index is done on the FIRST letter** (sort order is meaningless for substrings)
**sol: INVERTED IDX**
```
normal index 📇   row  -> words
inverted idx 🔄   word -> rows      "database" -> [1,5,9]     ← ONE lookup ⚡
```
**Postgres:** `to_tsvector(body) @@ to_tsquery('database')` + `CREATE INDEX ... USING GIN (...)`
**Free extras LIKE can't give:** **stemming** (running/ran/run) · **stop words** · **ranking** (`ts_rank`) · **operators** (`a & b`, `a | b`)
**When to leave PG:** typo-tolerance/fuzzy, facets, huge scale -> **Elasticsearch** (but = another system to run).

💡 **Why can a B-tree NEVER do "contains"?** Its power **IS** its sort order — each hop halves the space **because data is sorted**. But `database`, `basement`, `firebase` all contain "base" and sit in **completely different branches**. The tree can't eliminate anything. **It's not slow — it's answering a question it wasn't built for.**

========================================================

## 31. PostgreSQL specifics — T2 (+ ops T3)

🎣 *Postgres is the relational db that quietly does NoSQL too.*

**JSONB 📦** -> schemaless column INSIDE an ACID table
```
JSON  -> stores TEXT. reparsed every read 🐢. not efficiently indexable. keeps whitespace/key order.
JSONB -> stores BINARY. parsed once ⚡. GIN-INDEXABLE ✅   ← USE THIS
```
```sql
SELECT * FROM products WHERE specs->>'color' = 'red';
CREATE INDEX idx ON products USING GIN (specs);
```
=> **Mongo-style flexibility + ACID + joins + constraints.** Why "just use Postgres" is a real answer.

**ARRAYS 📋** `tags TEXT[]` · `WHERE 'sql' = ANY(tags)` -> ⚠️ often a **1NF violation** in disguise
**tsvector 🔍** -> full-text (T30)
**EXTENSIONS 🧩** -> one line, new capability: **postgis** (geo) · **pgvector** (AI embeddings — why PG became a vector db overnight) · **pg_trgm** (fuzzy)

**PG's index zoo 🦁** (most dbs have 1-2, PG has 5):
```
B-tree 🌲 default: ranges, sorting, equality
Hash   #️⃣ equality only
GIN    🔄 "many values per row" -> arrays, JSONB, full-text
GiST   📐 geometric/spatial, nearest-neighbour
BRIN   📊 huge NATURALLY-ORDERED tables (time-series). 1B rows: B-tree ~20GB vs BRIN ~100KB!
```

### ops cluster — T3
```
MVCC 📸 -> don't lock, keep MULTIPLE VERSIONS of each row.
           writer makes a NEW version, readers see the OLD one
           => readers NEVER block writers, writers NEVER block readers 🎉
           = how isolation levels (T17) work w/o read locks
           ⚠️ dead versions pile up -> VACUUM cleans -> forget = table BLOAT 🎈
WAL 💾  -> changes to a SEQUENTIAL LOG before the data files. crash -> replay 🔁
           = Durability (T16) + powers replication + PITR.   WAL is like oplog in mongo ✅
PARTITIONING ✂️ -> PARTITION BY RANGE 📅 (dates, most common) / LIST 🏷️ (region) / HASH 🎲
           win = PARTITION PRUNING (planner skips partitions that can't match)
pg_dump 🗄️ -> pg_dump -U user -F c mydb > backup.dump
SEQUENCES 🔢 -> nextval('s'). powers SERIAL.
PARALLEL QUERIES ⚡ -> splits one query across CPU cores
```

💡 **"just use Postgres" is a real architecture answer now** — it absorbed its competitors. Docs? **JSONB**. Search? **tsvector**. Geo? **PostGIS**. AI vectors? **pgvector**. Queues? **SKIP LOCKED**. Each was once a reason to add another system to run, monitor, back up, and keep in sync 😰. **The best architecture is usually the one with fewest moving parts.**

========================================================

## 32. Backup & recovery — T3

🎣 *An untested backup is a rumor.*

```
replicas 🖥️ => protect from HW failures  (across SPACE 🗺️)
backup   ⏳ => protect from HUMAN failures (across TIME)
ex: DROP DB -> replicated to all replicas in ms 🌊. only the BACKUP saves you.
```
🔑 **replication is a FIDELITY system — it copies changes faithfully and fast. It has NO concept of "that was a mistake."** Different threats. Need both.

**types:**
```
FULL 📦        -> copy EVERYTHING every time.  take: slow 🐢 + huge. restore: 1 file ⚡
INCREMENTAL ➕ -> changes since the LAST BACKUP (whatever it was)
   Sun FULL | Mon +5 | Tue +5 (since Mon) | Wed +5 (since Tue)
   take: fast+tiny. restore: full + EVERY increment 🔗 (one missing link = 💀)
DIFFERENTIAL 📊 -> changes since the LAST FULL
   Sun FULL | Mon 5 | Tue 10 | Wed 15  ← grows daily
   take: medium/grows. restore: full + LATEST diff = only 2 files ✅
```
🔑 **incremental = cheap to TAKE, expensive to RESTORE. differential = the reverse.**
💡 backups run nightly 😴, restores happen **once, in a panic** 🚨 -> optimize the thing you do **under pressure**.

**i choose my strategy depending on 2 nums:**
```
RPO (Recovery POINT obj) => least piece of data i can lose -> backup FREQUENCY (daily? hourly?)
RTO (Recovery TIME obj)  => least time my db can be down   -> restore MECHANISM
                            (hot standby / can wait for cold restore 📼)
```

**PITR 🎯 (point-in-time recovery)** -> full backup + **replay WAL** until 14:23:07
-> recover to the moment **BEFORE** someone ran `DELETE` without `WHERE` 😅. The real superpower of WAL.
**Rules:** **3-2-1** (3 copies, 2 media, 1 offsite) · **TEST RESTORES** 🧪 · automate 🤖 · encrypt 🔒

---
---

# Phase 6 — NoSQL & MongoDB

## 33. SQL vs NoSQL — T1

🎣 *SQL asks "is this correct?" NoSQL asks "is this fast enough?"*

the issue that made the spark of NoSQL (facebook/google ~2008): **the servers/machines can't fit the data anymore** (SQL -> vertical scaling).
-> relational **ASSUMES ONE SERVER** — that's what makes joins + ACID possible. Split across 1000 machines and the assumption **dies**.
**the trade: give up joins + some consistency, go to scalability.**

| Aspect | SQL 🏛️ | NoSQL 🌊 |
|---|---|---|
| Schema | Fixed | Flexible / schemaless |
| Data | Tables | Documents / KV / Graph / Column |
| Relations | Joins ✅ | Embed / app-side ❌ |
| Consistency | ACID 🔒 | BASE 🌫️ |
| Scaling | Vertical ⬆️ | Horizontal ➡️ |
| Best for | Integrity, complex Q | Scale, speed, flexibility |
| Examples | PostgreSQL, MySQL | MongoDB, Redis, Cassandra |

```
sql        nosql
table   -> collection
row     -> document (max limit 16mb)
column  -> field
```
**note:** sql **can** be sharded but you handle much stuff. in nosql it's **built in, designed for that**.

**CHOOSE SQL:** money/ACID · truly relational · **queries UNKNOWN yet** (SQL answers questions you haven't thought of) · stable structure
**CHOOSE NoSQL:** volume > 1 machine · schema **truly** unpredictable (IoT, logs) · huge throughput on simple lookups · **access patterns KNOWN**

💡 **2026 answer:** *"default to PostgreSQL. reach for NoSQL when you have a reason you can NAME."*

⚠️ **3 TRAPS:**
```
❌ "NoSQL is faster" -> faster AT WHAT?
   1 doc by key -> NoSQL ⚡ | join 4 tables + agg -> SQL 🏆 (NoSQL can't even do it well)
   = faster at a NARROWER set of things

❌ "NoSQL = no schema" -> there's ALWAYS a schema. the question is WHERE:
   SQL   -> schema in the DATABASE -> ENFORCED 🔒
   NoSQL -> schema in APP CODE     -> hoped for 🤞
   "schema-less" = "schema, UNENFORCED" -> 6mo later: docs in 5 shapes, defensive ifs 😖

❌ "it's SQL vs NoSQL" -> not a war. POLYGLOT PERSISTENCE 🧰
   PG 🏛️ orders/users (ACID) | Redis ⚡ sessions (speed) | ES 🔍 search
```

💡 **Why did the industry over-adopt then retreat?** Teams copied Google's **solution** without having Google's **problem** 🤷. Google sharded across 10,000 machines because they genuinely couldn't fit. A startup with 50k users **fits on one machine easily** — but adopted Mongo anyway and inherited **all the costs, none of the benefit** 😩. Then Postgres added **JSONB** = flexibility **without** the sacrifice. **Scale is a problem you should be lucky enough to have before you architect for it.** 🍀

========================================================

## 34. NoSQL document model — T1

🎣 *A row is a fragment. A document is the whole thing.*

```
in sql   -> user details is SPLIT across multi tables
in nosql -> why split? put them in ONE json { one doc, one read, ZERO joins }
```
=> **store data THE SHAPE YOUR APP USES IT.**

**mongo uses BSON (binary json) for documents instead of json:**
```
json -> str only, no dates, no binary
bson -> binary {files/img}, date, objectid, int32, int64, double, decimal128...
        + FASTER to scan (field lengths are PREFIXED -> skip fields without parsing)
⚠️ 16 MB limit per document  ← the real constraint. why you can't infinitely nest.
```
**objectid** => 12 bytes: `(4B timestamp, 5B machine code, 3B counter)`
🔑 **WHY client-side generated?** **no central coordinator needed** — any machine makes a unique id **without asking anyone** 🌍. Critical when sharded across 1000 servers.
💡 **BONUS:** timestamp-prefixed -> **roughly sortable by creation time for free.** `sort({_id:1})` ≈ sort by date.

**how to fix the idea of schema-less** -> mongo solved it by:
```
$jsonSchema -> DB layer  = a CONSTRAINT ⚖️  (enforced on EVERY write, even the shell)
mongoose    -> APP layer = a VALIDATION 🙏  (bypassed by shell / other services / scripts)
```
😬 **the catch:** almost **nobody enables `$jsonSchema`**. 99% of Node+Mongo projects use Mongoose only.
-> *"Mongoose gives the FEELING of a schema without the GUARANTEE."*
-> vs Postgres: you **CAN'T** create a table without types. The guarantee is **default-ON** 🔒.

⚠️ **"documents = denormalized rows"** = nearly right, misses the point.
```
SQL denorm -> DUPLICATING data for speed, accept the sync burden
documents  -> MODELING AROUND ACCESS PATTERNS
              "this data is only ever read together -> store it together"
```
🔑 **SQL models the DATA. Documents model the QUERY.**

💡 **Why is "know your access patterns first" MANDATORY in Mongo but optional in SQL?** In SQL the **schema is neutral** — normalize properly and you can answer **any** question later with a join, even ones you never imagined 🎁 (cost = a join at read time). In Mongo you **bake the query into the storage** — embed addresses and "get user+address" is instant ⚡, but "all users in Cairo grouped by zip" is now **painful** 😖 and **no index fixes it**. You didn't store data — **you committed to a query pattern.** **SQL lets you decide later. Mongo makes you decide now.** The real tradeoff isn't joins — it's **OPTIONALITY**.

========================================================

## 35. ORM vs ODM — T2

🎣 *ORM maps objects to tables. ODM maps objects to documents.*

concepts used to decrease the headache of writing queries, parsing results, manually putting vals in objects.
```
ORM (Obj. Relational Mapping) object <-> TABLES     ex: django orm, eloquent, sqlalchemy, hibernate
ODM (Obj. Document Mapping)   object <-> DOCUMENTS  ex: mongoose
```
**prefetch_related == populate** -> to avoid N+1 ✅

🔑 **THE REAL DIFFERENCE — where the schema lives:**
```
ORM -> the DB ALREADY HAS a schema. the ORM MIRRORS it 🪞  -> enforced by the engine 🔒
ODM -> the DB has NO schema. the ODM INVENTS one 🎨        -> enforced only by the ODM 🤞
```
-> `CharField(max_length=50)` becomes a **real column with a real constraint**. Mongoose's `required: true` doesn't.
-> **= T14 again: Mongoose is VALIDATION, not a CONSTRAINT.**

⚠️ **`.populate()` is NOT a join.** Mongo has none. It's a **2nd query + merge in memory** = exactly `prefetch_related`.
⚠️ **BOTH cause N+1.** Both need eager loading.

⚠️ **"the ORM writes better SQL than me"** = ❌ it writes **safe, generic** SQL. Not **optimal**.
🔑 the ORM's **job** is to hide SQL. Its **failure mode** is that it hides SQL — including the bad SQL it just wrote.
-> **use it, but be able to READ what it generated.** Django `.query` · Mongoose `.explain()`

💡 **Why does Mongoose exist if Mongo's whole pitch is "no schema"?** Because **schema-less was unbearable in practice** 😩. Collections drifted, every read went defensive. So the community built... **a schema layer** 🎭. Then Mongo itself added **$jsonSchema** — the same admission at the db level. **The industry didn't want NO schema. It wanted to CHOOSE WHEN to enforce it.**

========================================================

## 36. BASE vs ACID — T2

🎣 *ACID says "wait until it's right." BASE says "answer now, be right soon."*

since ACID promised me that when i commit the consistency is applied (everyone sees it) —
that was easy due to sql being on **ONE server**. what if more than 1 server?
-> write in Cairo 🇪🇬, read in Tokyo 🇯🇵. To keep the promise, Tokyo must **WAIT for Cairo on EVERY read**.
-> **speed of light says no** ⚡ (~200ms RTT). So: the db answers you with what it has, **but this data might be stale**.

```
ACID -> correct THEN available   ⏳
BASE -> available THEN correct   ⚡
= a swap of WHAT YOU REFUSE TO COMPROMISE ON
```
```
BA -> Basically Available   => always answers, even with stale data 📞
S  -> Soft state            => data changes WITHOUT input (replicas syncing) 🌊
E  -> Eventual Consistency  => propagation occurs btw servers, so eventually
                               all copies CONVERGE (consistency + TIME DELAY) 🎯
```
> **S explained:** hard state 🪨 = your hard drive, changes only when YOU write. Soft state 🌊 = read the same key twice with **no writes between** and get **different answers**, bec. replication landed in between. **S is the SYMPTOM of E** — consistency is eventual, so the state must be allowed to **shift underneath you** while converging.
> 😄 the acronym is a **chemistry joke** (acid/base), reverse-engineered. The **E** is what matters.

```
ex:
t=0    Sara updates her name in Cairo 🇪🇬
t=0    Tokyo replica still says the old name 🇯🇵  ← INCONSISTENT
t=50ms replication arrives -> Tokyo correct ✅    ← CONVERGED
=> for 50ms the system LIED
```
**acceptable in instagram likes count. NOT in banking systems.**
🔑 **THE WHOLE DECISION: "can my domain tolerate a BRIEF LIE?"**

**NOTE: not every ACID is SQL and not every BASE is NoSQL**
```
Mongo    -> multi-doc ACID transactions (since 4.0)
Cassandra-> TUNABLE consistency (per query!)
Postgres -> eventual consistency across READ REPLICAS (a distribution thing)
```
💡 **KILLER POINT:** add **ONE read replica** to Postgres and you've just introduced eventual consistency 🤯.
**SO: BASE vs ACID = a property of DISTRIBUTION, not of SQL vs NoSQL.** ✅

⚠️ **"eventually consistent = unreliable"** = ❌ it's a **DEFINED guarantee**: no new writes => **WILL converge**, bounded window. Not "maybe someday."

🎚️ **A DIAL, NOT A SWITCH:** `{w:1}` ⚡ (photo) vs `{w:"majority"}` 🔒 (payment). Same db, different guarantee per operation.

========================================================

## 37. Embedding vs referencing — T1

🎣 *Embed what you read together. Reference what you read apart.*

**Embedding:** put the child in the parent (comments in the post doc)
```javascript
{ _id: 1, title: "MongoDB Basics",
  comments: [ { author: "Sara", text: "Great post!" },
              { author: "Ansh", text: "Thanks!" } ] }
```
**Reference:** ref. the child inside the parent, but each lives in its own collection

**HOW to know which one:**
```
DO I EVER NEED THIS CHILD WITHOUT HIS PARENT???
no  -> EMBED     (a comment is meaningless without its post)
yes -> REFERENCE (a user exists independently of any order)
```
💡 **same question as CASCADE vs SET NULL (T15)!** Relationship modeling is the same problem in both worlds.

```
ONE-to-FEW        (10s)  -> EMBED     📦  addresses in a user
ONE-to-MANY       (100s) -> REFERENCE 🔗  order items -> products
ONE-to-SQUILLIONS (∞)    -> REFERENCE 🔗  + the CHILD holds the parent id
```
-> **child holds his parent id**, bec. if the parent has too many children, holding the ids **may break the 16mb limit**
**NOTE: take care of the doc limit 16mb.**
⚠️ **it breaks BEFORE it explodes:** every read of the post fetches **ALL 500k comments**. You wanted the title, you got 15MB 😱.
🔑 **NEVER embed anything UNBOUNDED** (viral post comments, activity logs, chat messages).

**hybrid approach:**
```javascript
{ _id: 1,
  user_id: ObjectId("..."),        // 🔗 reference for the full record
  user_snapshot: {                 // 📦 embedded copy for fast display
     name: "Sara",                 // need customer name AT PURCHASE TIME
     email: "sara@mail.com" },
  total: 250 }
```
🔑 **for orders you WANT the old value.** Sara renames next year -> the order should still show the name she had when she bought. **The "stale" duplicate isn't a bug — it's a HISTORICAL RECORD 📜.**
-> **denormalization in documents isn't always a perf hack. sometimes the duplicate IS the point.**

⚠️ **"embed everything, that's the point of Mongo"** = ❌ embedding **DUPLICATES** -> duplicates must be **SYNCED**.
```
name embedded in 4000 orders -> user renames -> update 4000 docs, NOT atomic 😱
=> you just REINVENTED THE UPDATE ANOMALY (T12) 🎭
```
🔑 **embed IMMUTABLE / snapshot data. reference MUTABLE SHARED data.**

**✏️ CAN embedded data be updated? YES — and it's ATOMIC (embedding's real prize):**
```javascript
{ $set: { "address.city": "Giza" } }              // dot notation. ⚠️ forget dots -> replaces whole subdoc
updateOne({_id:1, "comments.author":"Ansh"},
          { $set: {"comments.$.text": "..."} })   // $ = "the element the filter matched"
{ $set: {"comments.$[].flagged": false} }         // $[] = ALL elements
updateOne({_id:1}, { $set:{"comments.$[c].flagged":true} },
          { arrayFilters:[{"c.likes":{$lt:5}}] }) // $[id] = surgical, by CONDITION 🎯
{ $push: {comments:{...}} }  ➕      { $pull: {comments:{author:"Ansh"}} }  ➖
```
🔑 **single-doc writes are ALWAYS atomic — no transaction needed.** Title + new comment + counter = all or nothing.
-> vs references: the same change = 3 collections = you need a **multi-doc transaction** 😰.
-> **T37's "these are ONE thing" proving itself: they update together bec. they ARE one thing.**

⚠️ **the actual hard part isn't UPDATING — it's QUERYING across parents:**
```javascript
find({"comments.author":"Sara"})  // ❌ returns WHOLE POSTS, not comments.
                                  //    scans every post, dig into arrays in app code 🐢
```
⚠️ **auto-update embedded copies?** SQL has `ON UPDATE CASCADE`. **Mongo has NOTHING built in.** Options: **Change Streams** (watch the oplog, react) · Atlas Triggers · Mongoose middleware (bypassable) · or don't duplicate + `$lookup`.
-> **if you're building sync machinery for embedded data, that's a signal you modeled wrong.** 🚩

💡 **Why is this decision scarier in Mongo than SQL?** In SQL, normalization is the **default and reversible** — slow query? add an index, or denorm a column. A schema change, not a rewrite 🔧. In Mongo, embed-vs-reference **IS your query plan**. Embed comments, then need "all comments by Sara across all posts" -> **no index fixes it**, you must **restructure the whole dataset and rewrite every query** 😱. **SQL lets you defer the decision. Mongo makes you bet.** 🎲

========================================================

## 38. MongoDB operations — T1

🎣 *Mongo's query language is JSON all the way down — the query IS an object.*

```javascript
// CREATE
db.users.insertOne({ name: "Sara", age: 25 })
db.users.insertMany([{...}, {...}])
// READ
db.users.find({ age: { $gt: 18 } })    // many (returns a CURSOR 📜)
db.users.findOne({ _id: 1 })           // one document
// UPDATE
db.users.updateOne({ _id: 1 }, { $set: { age: 26 } })
db.users.updateMany({ city: "Cairo" }, { $set: { active: true } })
db.users.replaceOne({ _id: 1 }, { name: "Sara" })   // ⚠️ REPLACES the whole doc
// DELETE
db.users.deleteOne({ _id: 1 })   /   db.users.deleteMany({ active: false })
```

⚠️ **$set vs replaceOne — THE CLASSIC BUG:**
```
doc: {_id:1, name:"Sara", age:25, city:"Cairo"}
updateOne({_id:1},{ $set:{age:26} })  ✅ {_id:1, name, age:26, city}  ← only age
replaceOne({_id:1},{ age:26 })        💥 {_id:1, age:26}  ← name + city ANNIHILATED
```
🔑 **$set PATCHES. replaceOne OVERWRITES.** Forget `$set` -> you **silently delete fields**.

**UPDATE OPERATORS:** `$set` 🔧 `$unset` 🗑️ `$inc` 📈 `$push` ➕ `$pull` ➖ `$addToSet` 🎯(no dups)
> 💡 **`$inc` is ATOMIC** -> `{$inc:{views:1}}` is safe under concurrency. No read-modify-write race.

**QUERY OPERATORS:** `$gt $gte $lt $lte` · `$ne` · `$in $nin` · `$exists` · `$regex` · `$type` · `$and $or $not`

```javascript
db.users.updateOne({_id:1}, {$set:{age:26}})
// -> returns { matchedCount: 1, modifiedCount: 1 }   ← just a REPORT 📋
db.users.findOneAndUpdate({_id:1}, {$set:{age:26}}, {returnDocument: "after"})
// -> returns THE DOCUMENT itself 📄
```
🔑 **updateOne = "do it" 🔨 · findOneAndUpdate = "do it and GIVE IT BACK" 🎁**
-> use findOneAndUpdate to **atomically CLAIM a job from a queue** (update status + get the job in one op, so two workers can't grab it).

```javascript
db.users.updateOne(
  { email: "sara@mail.com" },
  { $set: { lastSeen: new Date() } },
  { upsert: true }     // if not found then create
)
```
🔑 **ONE atomic op -> kills the check-then-insert RACE (T14).**

**Projection:**
```javascript
db.users.find({}, { name: 1, email: 1 })  // only these + _id
db.users.find({}, { password: 0 })        // everything EXCEPT pass
⚠️ CAN'T MIX 1s and 0s (except _id)
```
-> = Mongo's `SELECT` column list. Enables **covered queries**, same as avoiding `SELECT *`.

**cursor** -> a ptr to the ret. of the query. **LAZY**, streams in batches (101 first, then ~4MB) -> a huge `find()` doesn't blow your RAM. `.limit().skip().sort()` are all applied **on the server**.

⚠️ **PAGINATION TRAP:**
```javascript
❌ find().skip(10000).limit(10)   // walks 10,010 docs, THROWS AWAY 10,000 🗑️. skip() is O(n)
✅ find({_id:{$gt:lastSeenId}}).limit(10)   // jumps via index ⚡ CONSTANT time, any page
```
-> same as the sharded `OFFSET` problem. Fine on page 2, catastrophic on page 1000.

**join alt. -> `$lookup`:**
```javascript
db.orders.aggregate([
  { $lookup: { from: "users", localField: "user_id",
               foreignField: "_id", as: "user" }}
])
```
-> a **real server-side LEFT OUTER JOIN**. ⚠️ slow across shards, and often means **you should have embedded**.

**AGGREGATION PIPELINE** -> docs flow through **STAGES**, output of one = input of the next 🚰
```javascript
db.orders.aggregate([
  { $match:  { status: "complete" } },                        // WHERE    🔍
  { $group:  { _id: "$customer_id", total: {$sum:"$amount"} }}, // GROUP BY 🗜️
  { $sort:   { total: -1 } },                                 // ORDER BY ↕️
  { $limit:  10 }                                             // LIMIT    ✂️
])
```
**Common stages map to SQL:**
```
$match = WHERE   |  $group = GROUP BY  |  $project = SELECT
$sort = ORDER BY |  $limit = LIMIT     |  $skip = OFFSET
$lookup = LEFT JOIN  |  $unwind flattens arrays into rows
$match AFTER $group = HAVING
```
🔑 **$match FIRST, ALWAYS** — it's the **only stage that can use an INDEX**, and filtering early means less work for every later stage. (= "filter early", T5)

**`.explain("executionStats")` == EXPLAIN in sql**
```
❌ COLLSCAN -> collection scan, NO INDEX 🐢
✅ IXSCAN   -> index scan ⚡
✅ PROJECTION_COVERED -> covered query 🎯
totalDocsExamined 50000 vs nReturned 1  😱  = the same signal as "Rows Removed by Filter"
```

### Mongo Indexes — same B-tree, same rules as SQL (leftmost prefix applies!)
```javascript
createIndex({email:1})                        // 1=asc, -1=desc
createIndex({email:1},{unique:true})          // 🛡️
createIndex({last:1, first:1})                // compound, LEFTMOST PREFIX
createIndex({body:"text"})                    // 🔍 text
createIndex({loc:"2dsphere"})                 // 🌍 geo
```
**1. MULTIKEY (array index — SQL has NO equivalent)**
```javascript
{ tags: ["sql","mongo","db"] }  ->  createIndex({tags:1})
Mongo SILENTLY indexes EVERY element:
  "sql"->doc1 | "mongo"->doc1 | "db"->doc1   ← 1 doc, 3 entries 🤯
find({tags:"mongo"}) ⚡ instant
```
> **why SQL can't:** a relational col holds **ONE atomic value** (1NF!) — there IS no array. You'd use a **junction table**.
> ⚠️ 100-element array = **100 index entries** for 1 doc 🎈 · ⚠️ **can't compound TWO array fields** (cartesian blowup).

**2. TTL ⏰**
```javascript
createIndex({createdAt:1},{expireAfterSeconds:3600})
-> doc deletes ITSELF after 1hr. no cron, no cleanup job.
```
> PG has nothing like this. Why Mongo does: **caching/sessions** were its early bread and butter. It's a **key-value idea** — which is why **Redis has EXPIRE** too.

**3. NESTED FIELD 🪆**
```javascript
{ user: { address: { city: "Cairo" } } }
createIndex({"user.address.city": 1})   ← dot notation, 3 levels deep
```
> SQL can't — there are no levels. Closest cousin: a **GIN index on JSONB**.

💡 **Why does Mongo have `$lookup` at all if "no joins" is the point?** **Reality won** 🎭. Mongo shipped without joins ("just embed"). Then every real app hit a case where it genuinely needed to relate two collections — and devs were writing **manual joins in app code, badly, over the network, in a loop** 😰. So Mongo added **$lookup** (3.2), then **multi-doc ACID transactions** (4.0), then **$jsonSchema**. Notice: **joins, transactions, schemas — the three things NoSQL discarded ALL QUIETLY RETURNED.** Not because the idea was wrong at Google scale, but because **most apps aren't Google**, and relational features exist for **excellent reasons**.

---
---

# Phase 7 — Distributed Data & Scaling

## 39. Replication — T2

🎣 *One writer, many readers, and a hat that gets passed when someone dies.*

```
REPLICATION 📋 — same data, COPIED
   [ ALL data ] -> [ ALL data ] -> [ ALL data ]
      primary        replica        replica

SHARDING ✂️ — different data, SPLIT
   [ users 1-30M ]  [ users 31-60M ]  [ users 61-90M ]
      server A          server B          server C
```

**FACEBOOK — both, stacked:**
```
            3 BILLION USERS 🌍
        ┌─────────┼─────────┐
    SHARD 1    SHARD 2    SHARD 3        ← horizontal ✂️ (VOLUME)
   users 1-1B  1-2B       2-3B
        │          │          │
   👑 📖 📖   👑 📖 📖   👑 📖 📖         ← vertical 📋 (SAFETY)
   PRI SEC SEC ...
   🇺🇸 🇺🇸 🇮🇪   🇺🇸 🇺🇸 🇸🇬   🇺🇸 🇺🇸 🇩🇪    ← different AZs/regions
```
🔑 **EACH SHARD IS ITS OWN REPLICA SET.** Shard 1's primary knows **NOTHING** about Shard 2.

**WRITE FLOW:**
```
1️⃣ Sara hits "Post" in Cairo
2️⃣ ROUTER: "which shard has user 42?"  hash(42) % 3 = 0 -> SHARD 1 ✂️
           (this is the SHARD KEY doing its job 🔑)
3️⃣ Route to SHARD 1's PRIMARY 👑 (Oregon)
   ⚠️ NOT any secondary — only the primary takes writes
4️⃣ Primary writes it. Logs it to the WAL / binlog / oplog 📜
5️⃣ Primary streams the log to ITS secondaries
   -> Oregon secondary 📖 (~1ms)   -> Ireland secondary 📖 (~80ms 🌊 LAG!)
6️⃣ "Posted!" returns to Sara

NOTE: Shards 2 and 3 NEVER HEARD ABOUT THIS 🤷. Completely independent.
```
🔑 **HOW replication works: NOT by copying data — by copying the LOG.** Secondaries **pull log entries and REPLAY them in order**. The log is a **sequential, deterministic list of ops** -> replay = same state. (= exactly how crash recovery works, T16.)

**Replica set (primary, secondary):**
```
Primary dies 💥 -> secondaries hold an ELECTION 🗳️
-> One secondary is PROMOTED -> it's now the primary 👑
-> Old primary comes back -> it becomes a SECONDARY 📖
```
🎩 **the role isn't attached to the machine — it's a HAT that gets passed.**

⚠️ **Is the primary "a replica"?** It's a **MEMBER of the replica SET**, not "a replica" — different **role**. Mongo's official word is **SECONDARY** (deliberate, names the role). PG says **primary/standby**.
> 🎬 film crew: the **crew** = replica set · the **director** 👑 = primary · the **cameras** 📖 = secondaries. The director is part of the crew but isn't a camera.

**NOTE: nodes have to be split, each on its own server, to survive HW failures.**
```
same machine  -> protects against NOTHING 🤷 (machine dies -> BOTH die)
diff machine  -> server crash, disk failure 🖥️
diff rack     -> rack power/switch 🔌
diff AZ       -> data center fire/flood 🔥
diff region   -> regional outage 🌍
```
also **load**: same server = primary + replica **fight over the same CPU/RAM/disk** = added work, not capacity 📉.

**NOTE: Async vs Sync writing (primary and secondary nodes)**
```
sync  🔒 -> primary WAITS for replicas to confirm, all on the same page
            (if primary dies, data is SAFE. but writes = speed of the SLOWEST replica 🐌)
async 🌊 -> primary writes, secondaries take their time
            (if primary dies, some data MAY BE LOST 💀. but FAST ⚡)
```
-> **most systems: async.** The tiny risk window beats paying replica latency on every write.
**Mongo version:**
```javascript
{ writeConcern: { w: 1 } }           // primary only ⚡ (async)
{ writeConcern: { w: "majority" } }  // wait for a majority 🔒 (safer)
```
**USE CASE:** *it's a DIAL, not a switch.* Photo upload -> `w:1`. Payment -> `w:"majority"`. Same db, different guarantee.

**What if failover occurred?** -> **ELECTION** for making a primary -> **ODD number** to guarantee the winner **because there can't be 2 brains**.
```
majority = >50%.   3 nodes -> majority 2 ✅   |  4 nodes -> majority 3 (no better, costs more)
⚠️ even split + partition -> two halves EACH elect a primary
   👑 A says balance=500 | 👑 B says balance=300 -> SPLIT BRAIN 🧠💥
-> odd makes it ARITHMETICALLY IMPOSSIBLE
```

**Replication lag:** the delay between a write landing on the primary and appearing on a secondary. = **eventual consistency, live.**

💡 **Why can't secondaries accept writes and merge later?** Because there's **no correct way to merge conflicting writes**. A sets 500, B sets 300 — **which is right?** There's no answer *in the data* 🤯. "Last write wins" needs **synchronized clocks** (which don't exist across machines) and **silently destroys** one user's change. Systems that allow it (**multi-master**) get **conflict-resolution hell** — vector clocks, CRDTs, or asking your app to decide. **Single-primary isn't a limitation — it's the cheapest possible way to have a single truth.**

========================================================

## 40. Sharding / horizontal partitioning — T2

🎣 *Sharding is partitioning plus the network — and the network takes your joins.*

```
partitioning -> split the table            => solves "TABLE too big"
   ex: orders_2024, orders_2023...   WHERE year = 2024 => db SKIPS other partitions (PRUNING ✂️)
sharding -> split the table ACROSS MACHINES { horizontal partitioning + NETWORK }
                                            => solves "SERVER too small"
   ex: server A (orders 1-33M), server B (orders 34M-66M)
```
```
PARTITIONING  -> one big table -> many pieces, SAME server       🖥️
SHARDING      -> one big table -> many pieces, DIFFERENT servers 🖥️🖥️🖥️
```
🔑 **partition + NETWORK = shard. All the pain lives in that one difference.**

**HORIZONTAL vs VERTICAL:**
```
horizontal ↔️ -> same COLUMNS, fewer ROWS per split  (partitioning AND sharding both do this)
vertical   ↕️ -> same ROWS, fewer COLUMNS per split  (separating HOT vs COLD data)
```
**NOTE: vertical partitioning is often better solved by NORMALIZATION** ex: 1M rows × 80 cols
**while the use case for horizontal:** ex: 100M rows × 6 cols
> vertical's win: narrow rows -> **more rows fit per page** -> fewer disk reads. But every full read now needs a **JOIN**.
> also used for **security** (employees_public vs employees_private).

⚠️ **THE REAL TRIGGER isn't "one partition got too big"** — it's *"the whole machine hosting ALL the partitions is out of room or overloaded."* A single oversized partition means you should **RE-PARTITION** (split 2026 into monthly instead of yearly) before jumping to sharding.
🎣 **partition HARDER (smaller slices, same server) before you shard (new servers).**

**ARCHITECTURE:**
```
App -> 🧭 ROUTER (mongos) "which shard has user 42?"
        ├─ SHARD1 👑📖📖    ← each shard = its OWN replica set
        ├─ SHARD2 👑📖📖
        └─ SHARD3 👑📖📖
       📚 CONFIG SERVERS = the MAP (which ranges live where). also replicated.
```

🔑 **THE SHARD KEY — the most important decision you'll make:**
```
RANGE 📏 -> users 1-30M -> A, 31-60M -> B
   ✅ range queries efficient (contiguous)
   ❌ HOTSPOTS — sequential ids / created_at -> EVERY new write hits the LAST shard 🔥
      (you sharded and one machine still does all the work 🤦)
HASH 🎲  -> hash(user_id) % 3
   ✅ perfectly even  |  ❌ range queries hit EVERY shard
```
**GOOD SHARD KEY:** high **cardinality** · even **distribution** · **matches your queries** (reads hit ONE shard 🎯)
⚠️ **in Mongo you largely CAN'T change it later** -> wrong = rebuild the cluster 😱

**WHAT BREAKS:**
```
✅ query WITH the shard key -> ONE shard, fast 🎯
🟡 sharding UNION  -> send the query to ALL shards -> each returns -> CONCATENATE (scatter-gather)
                      easy bec. UNION = STACKING (T8), no matching needed.
                      ⚠️ as slow as your SLOWEST shard 🐌
🟢 sharding JOINS  -> on the SAME shard, NO ISSUE (this is CO-LOCATION ✅)
🔴 issue occurs when you need to join across DIFF shards:
   fetch user from S-A -> fetch orders from S-B and S-C -> JOIN OCCURS IN MY APP (py/js) 😰
🔴 ORDER BY + LIMIT -> every shard returns N, you re-sort and throw away 🗑️
   LIMIT 10 OFFSET 10000 -> each shard returns 10,010 = 30,030 rows in app memory to show 10 💥
   -> WHY sharded apps use CURSOR pagination (WHERE id > last_seen), NOT OFFSET
💀 TRANSACTION across shards -> 2-phase commit or SAGAS
```
🔑 **the shard key doesn't just distribute data — IT DECIDES WHICH QUERIES REMAIN POSSIBLE.**

**Cross-shard join strategies:** ① **co-locate** by shard key 🎯 (the main one) · ② **denormalize** 🗜️ (copy user_name into orders) · ③ **broadcast** small tables to every shard 📢 · ④ join in the app 😔

⚠️ **CAN SQL BE SHARDED?** **YES.** Instagram = sharded PG · Facebook = sharded MySQL (1000s of shards) · Shopify · YouTube (built **Vitess**).
```
Mongo 🍃 -> BUILT IN.  sh.shardCollection(...) — it routes, balances, rebalances 🤖
PG 🐘    -> YOU BUILD IT. you pick the key, write routing, handle cross-shard, rebalance 😰
```
🎣 **the difference isn't CAPABILITY — it's WHO SUFFERS.** 😅
**Middle ground:** **Vitess** 🐦 (MySQL) · **Citus** 🐘 (a PG *extension*!) · **CockroachDB** 🪳 (auto-shards)
💡 **why NoSQL FEELS like "the sharding one":** it **dropped joins/txns UPFRONT** -> sharding is easy bec. **there's nothing left to break** 🎭.

⚠️ **SHARD AS LATE AS POSSIBLE 🚨.** Try first: index -> optimize -> cache -> replicas -> partition -> **a bigger machine** (seriously — modern servers do **terabytes and 100k+ QPS**). Most companies **never** shard. **And sharding gives ZERO redundancy — replicate your shards.**

💡 **Why is sharding a ONE-WAY DOOR?** It doesn't just move data — **it deletes assumptions from your codebase**. Every query becomes shard-aware: no cross-shard joins, cursor pagination, single-shard transactions, merged aggregations 🔧. To un-shard, consolidating data is tedious but doable 🚚 — **but your entire app is now built around constraints that no longer exist**, full of workarounds for problems you don't have 😩. **You'd be rewriting the app, not the database. Sharding is an architectural commitment disguised as an infrastructure change.**

========================================================

## 41. CAP theorem — T1

🎣 *When the network breaks, you answer WRONG or you DON'T ANSWER. Pick.*

```
C -> Consistency          {everyone sees the same data}
A -> Availability         {when you call, it answers}
P -> Partition Tolerance  {survives the network breaking}
```
**ONLY CHOOSE 2:** (in a distributed sys -> **C or A. P is MANDATORY.**)

**THE 60-SEC PROOF:** `[A] ✂️---✂️ [B]` — cable CUT. B has **exactly two options**:
```
Option 1 🔒 -> "I can't reach A. I might be wrong. I REFUSE to answer."
               ✅ Consistent. ❌ Not Available.  (ATM / Bank)
Option 2 📞 -> "I'll tell you what I have (maybe stale)."
               ✅ Available. ❌ Not Consistent.  (social media)
```
**NO OPTION 3** 🚪 — B **cannot know** what A knows. The cable is cut. = physics + logic, not a design tradeoff.

🚨 **THE PART EVERYONE GETS WRONG: "I'll build a CA system!"**
**P IS NOT A CHOICE** — cables cut ✂️, switches die 🔌, packets drop 📦, DCs disconnect 🏢, someone trips on a cord 🦶.
-> **you don't choose P. reality gives it to you.**
**CA ==> exists only if there's ONE MACHINE** {no network to be partitioned} — i.e. **not a distributed system at all.**

🔑 **SAY THIS:** *"CAP isn't 'pick 2 of 3.' It's: **when a partition happens, do you choose C or A?**"*
-> separates you from everyone reciting the triangle.

```
CP 🔒 -> Mongo (minority side has no primary -> writes REFUSED) | PG sync repl | etcd, Zookeeper, HBase
         WHEN: money 💰, inventory 📦, seat booking 🎫 (a wrong answer is WORSE than none)
AP 📞 -> Cassandra | DynamoDB | Riak, CouchDB, DNS 🌍
         WHEN: feeds 📱, likes ❤️, catalogs 🛍️, analytics 📊 (a stale answer BEATS none)
```
> 🏧 **ATM partitioned from the core db:** CP = "service unavailable" 😤 (books correct) · AP = "here's $500" 😊 (may overdraw). **Both are right — for different businesses.** ⚖️ **CAP doesn't tell you what to do. It tells you that you MUST DECIDE.**

**PACELC (CAP's blind spot: it only describes DURING a partition — but partitions are RARE)**
```
IF Partition  ->  choose A or C
ELSE (normal) ->  choose L (latency) or C (consistency)
```
-> bec. even on a **healthy network, consistency COSTS LATENCY** (waiting for replica ack takes time).
```
Cassandra = PA/EL   |   MongoDB = PC/EC   |   DynamoDB = PA/EL (tunable)
```
💡 dropping PACELC shows you know CAP is **incomplete** — the "yes, and…" that signals depth.

⚠️ **"Mongo is CP, Cassandra is AP"** = roughly true, but **both are TUNABLE**. The label describes the **default**, not the ceiling. Mongo `w:1` leans AP; Cassandra `ALL` leans CP.
⚠️ **CAP does NOT describe normal operation** — with a healthy network you **can** have C and A. That's why your db works fine most days. **CAP describes the FAILURE MODE.**

💡 **Why does CAP feel like a cheat?** It seems to say *"distributed systems are broken."* They're not — **CAP just names a constraint that was always there** 🎭. Before CAP, engineers **believed** they had all three and shipped systems that **silently violated one under stress** — usually consistency, discovered later as **corrupted data nobody could explain** 🤯. Brewer didn't invent a limitation; he **forced the choice into the open** 🔍. **Systems that "have all three" haven't beaten CAP — they've just hidden which one they drop.**

========================================================

## 42. Redis & key-value stores — T2

🎣 *Redis isn't a db with an index. Redis IS the index.*

**all dbs asked how to increase the speed on disk** (indexes, query planners...)
**Redis asked: what if we didn't use the disk AT ALL?**
```
disk read   ~1,000,000 ns (SSD)  🐢
memory read ~100 ns              ⚡   = 10,000× faster
```
**Redis is a giant js object that lives on the network and mostly never touches disk.**
in-memory **key-value store, a hash table**. `GET user:42` -> **O(1)**, ~0.1ms.
🔑 **the key IS the index. you don't QUERY Redis — you LOOK THINGS UP.**

**rich DS (str, list, hash, ...):**
```
STRING 📝 SET user:42 "Sara"        cache, counters
HASH   🗂️ HSET user:42 name "Sara"  objects (field-level access)
LIST   📋 LPUSH queue "job1"        queues, stacks
SET    🎯 SADD tags "sql"           unique membership
ZSET   🏆 ZADD board 500 "sara"     LEADERBOARDS — ALWAYS sorted ⭐
          ZREVRANGE board 0 9 -> top 10 instantly ⚡
```
> vs SQL: `ORDER BY score DESC LIMIT 10` over millions, re-sorted constantly 😰
> 💡 Redis's **geo** commands are built **on ZSETs** (coords encoded as a sortable score).

**TTL ⏰ is native:** `SETEX session:abc 3600 "user42"` -> the key **deletes itself**. No cron.
-> why Redis owns **sessions / caching / rate limiting** = *"remember this briefly."*

**use case:** caching · **rate limiting** · pub/sub · job queues · sessions · distributed locks (`SET k v NX EX 30`)
```bash
INCR rate:user42        # ATOMIC ✅
EXPIRE rate:user42 60   # window resets itself ⏰   -> >100? block 🚫
```

**Single threaded (network bound, not CPU bound)** — *"that's a bottleneck!"* **NO.**
```
✅ NO locks 🔓  ✅ NO races (every cmd ATOMIC, FREE)  ✅ no context switching  ✅ predictable latency
```
> same insight as the **Node event loop** 🧵 — single-threaded isn't a weakness when the bottleneck is **I/O, not CPU**. **100k+ ops/sec on ONE core.**

**Persistence:** **RDB** 📸 = point-in-time snapshot (fast restart, lose the last N sec) · **AOF** 📜 = append every write command, replay on restart (**this is a WAL!**). Most run both.
⚠️ **it's for RECOVERY, not a DURABILITY GUARANTEE.** Not a system of record. Lose 1 sec unacceptable? -> **PostgreSQL**.

**cost:** no queries · no joins · **expensive RAM** (100GB RAM ≫ 100GB SSD 💸)
🔑 **Redis answers questions you PLANNED FOR. It cannot answer questions you didn't.**

💡 **Why is single-threaded its BEST feature, not a compromise?** Think what concurrency **costs**: locks 🔒, contention 🥊, context switching 🔄, races 🏁, and the whole **mental burden** of thread safety. Redis **deleted all of it with one decision** ✂️ — and lost **nothing**, because its work per command is ~100ns; the real time is the **network**, handled by an **event loop**. Every other db pays enormous complexity for multi-threading because they're **disk-bound**. Redis isn't. **The "limitation" IS the reason `INCR` is atomic with zero lock overhead. Sometimes the simplest architecture wins by refusing the problem entirely.**

========================================================

## 43. Other NoSQL categories — T3

🎣 *Four shapes. Pick the one that matches your data.*
```
KEY-VALUE     🔑 a dict. key -> blob                    Redis, DynamoDB    query: KEY ONLY
DOCUMENT      📄 a dict. key -> QUERYABLE json          MongoDB            query: any field
COLUMN-FAMILY 📊 rows, columns grouped + distributed    Cassandra, HBase   query: partition key
GRAPH         🕸️ nodes + edges. RELATIONSHIPS ARE DATA  Neo4j              query: traversal
```
💡 **document = key-value that lets you LOOK INSIDE the value.** That's the whole difference.

**CASSANDRA 📊** — **NO PRIMARY**, every node accepts writes. Insane writes (millions/sec, append-only). **AP** by default.
⚠️ **query WITHOUT the partition key = an ERROR, not "slow"** 🚫. 🔑 **"your TABLE is your QUERY"** -> need another access pattern? **create another table with the data duplicated.** Denormalization isn't a choice — **it's the model.** Use: time-series 📈, logs 📜, IoT 🌡️.

**NEO4J 🕸️** — SQL needs a **self-join PER HOP** (5 hops = 5 joins 😰; shortest path = 💀). Graph stores the **pointer IN the node** -> **O(1) per hop**. 💡 **the term: "INDEX-FREE ADJACENCY."** Use: social 👥, fraud rings 🚨, recs 🎬.
> 💡 **why least adopted?** Most apps don't have graph-shaped problems — they have "user has orders" = a **1-hop join** SQL does perfectly. Graph only wins at **3+ hops**. Plus graphs **scale badly** (can't shard — edges cross everything 💥) and PG has **recursive CTEs**. **Neo4j is a scalpel.**

**CHOOSING:** fast by key -> KV · general app data -> document · insane writes+known reads -> column · **relationships ARE the question** -> graph · **anything else -> PostgreSQL** 🐘

========================================================

## 44. Data warehousing — T3

🎣 *OLTP runs the business. OLAP understands it.*
```
OLTP 🏃 Online TRANSACTION Processing -> RUN the business
OLAP 🧠 Online ANALYTICAL Processing  -> UNDERSTAND the business
```
| | OLTP 🏃 | OLAP 🧠 |
|---|---|---|
| query | `WHERE id=42` | `SUM() GROUP BY` over millions |
| speed | ms ⚡ | sec-min 🐢 |
| schema | **normalized** | **denormalized** (star) |
| storage | **ROW**-oriented | **COLUMN**-oriented ⭐ |
| ex | PostgreSQL | Snowflake, BigQuery, Redshift |

⭐ **WHY warehouses are fast — ROW vs COLUMN storage:**
```
ROW 📋    [id|name|city|amt][id|name|city|amt]...
   "row 42" -> ONE read ⚡ | "SUM all amounts" -> read EVERY col of EVERY row 😰
COLUMN 📊 [all ids][all names][all cities][all amounts]
   "SUM all amounts" -> ONE contiguous block ⚡, never touches name/city 🎯
```
🔑 **columnar wins TWICE:** skip the cols you don't need **+** similar values sit together -> **compression 10-100×** 💎

```
EXTRACT   📤 -> pull from OLTP dbs, APIs, logs
TRANSFORM 🔧 -> clean, dedupe, reshape, join (to homogeneous formats)
LOAD      📥 -> into the warehouse for analysis + reporting
```
**ELT** = the modern flip — **load raw first, transform INSIDE** the warehouse (it's more powerful than your ETL server).
**WAREHOUSE** 🏭 structured, schema-**on-write** · **LAKE** 🏞️ raw, schema-**on-read** · **LAKEHOUSE** 🏠 both (Databricks, Iceberg)
⚠️ lakes become **data swamps** 🐊 when nobody documents them — schema-on-read = **nobody enforced anything**. (= the NoSQL schema lesson AGAIN 🎭)

💡 **Why can't one db do both?** Their optimal designs are **exact opposites** 🎭. OLTP wants **normalized** + **row storage** + **many indexes**. OLAP wants **denormalized** + **column storage** + **no indexes** (it reads everything anyway). **Every choice that makes one fast makes the other slow.** It's not laziness that we have two systems — **the requirements are genuinely contradictory.**

========================================================

## 45. "Can you run SQL in MongoDB?" trap — T2

🎣 *Not a knowledge question. A "do you think in MODELS or SYNTAX?" question.*
```
❌ "No."                  -> correct, but you said NOTHING
❌ "Yes, there's a tool!" -> missed the point entirely
✅ "Not really, and here's WHY the question is interesting..."
```
**NOT NATIVELY.** Mongo's query language **is JSON**. Bridges exist — **Atlas SQL Interface**, **BI Connector** (Tableau) — but they only **translate SQL -> aggregation pipeline**. **BI adapters for reporting tools, not a SQL engine** 🎭.

🔑 **THE REAL ANSWER: SQL isn't SYNTAX. It's a CONTRACT WITH THE RELATIONAL MODEL.**
| SQL assumes | Mongo has |
|---|---|
| fixed **schema** | dynamic documents 📄 |
| **atomic** columns (1NF) | nested objects + arrays 🪆 |
| **JOINs** across tables | embed, or `$lookup` (limited) 🔗 |
| **set-based** ops | document-based 📦 |
```sql
SELECT * FROM users WHERE address.city = 'Cairo'
                          ↑ SQL has NO CONCEPT of a nested field. columns are FLAT. 🚫
```
🗣️ **SAY THIS:** *"you can translate the **syntax**. you can't translate the **model**. A SQL layer over Mongo is a **BI adapter**, not a query engine."*

🕳️ **THE OTHER HALF OF THE TRAP — the real follow-up is "so you'd pick the wrong database?"**
✅ **"If I need SQL, that's a signal the data is relational — so I'd use a relational database. Wanting SQL in Mongo usually means Mongo was the wrong choice, or the data should have been embedded differently."**
-> shows **judgment**, not trivia 💎

🎭 **THE IRONY:** Postgres does the **reverse, and better** — `WHERE specs->>'color' = 'red'` (JSONB). **Postgres gives you documents inside SQL. Mongo can't give you SQL inside documents.** 🌉 **The bridge is ONE-WAY, and it goes toward relational.**

💡 **Why do interviewers love it?** Because **the wrong answers are so revealing** 🔍. "Yes, BI connector" -> you think dbs are **syntax you configure**. "No" + stop -> you know a **fact but not a reason**. Only the person who explains that SQL is a **contract with the relational model** has shown they understand dbs as **models with tradeoffs**, not products with features. **A one-question filter for depth, costs them 30 seconds.**

---
---

# 📎 Appendix — Loose Q&A Bank

**NULL vs blank vs zero** 🕳️
```
null    -> MISSING / UNKNOWN
blank ""-> already a VALUE (empty string)
zero    -> an actual value, a real answer
ex: num of courses -> 0 = we KNOW there are none | null = we know NOTHING | "" = a value we can't read
```
⚠️ `null = null` is **UNKNOWN**, not true — two unknowns can't be compared. **Don't use it in conditions -> use `IS NULL`.**
⚠️ `COUNT(column)` = rows that are **NOT NULL** · `COUNT(*)` = **all** rows. That's why they differ.
⚠️ **NULL is contagious:** `100 * NULL = NULL`. `COALESCE` stops the spread.

**Top-N per group** 🏆
```sql
WITH ranked AS (
  SELECT name, dept, salary,
         ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) AS rn
  FROM employees )
SELECT * FROM ranked WHERE rn = 1;
```
-> **window functions** don't summarize — they leave every row and **add the calc'd col, duplicated per related row**
   ex: `Eng -> 7000` ==> `Ali|Eng|7000` & `Sara|Eng|7000`
-> ⚠️ **can't filter a window in WHERE** (it runs after) -> **CTE, then filter outside.**
**The 3 rankers** (only differ on ties):
```
values:      100, 100, 90
ROW_NUMBER ->  1,  2,  3    always unique, ties broken arbitrarily
RANK       ->  1,  1,  3    ties tie, then SKIPS ⏭️
DENSE_RANK ->  1,  1,  2    ties tie, no skip
```

**Correlated subquery — when slower than a JOIN?** a subquery that **uses a column from the outer query** -> so it **can't run once** -> **re-executes PER OUTER ROW** (N² behavior). A JOIN does it in **one pass** with a hash table. Rewrite as a JOIN when the subquery is in the SELECT list or a `WHERE ... IN` over a large outer set.

**Duplicate rows after a JOIN — cause + fix?** **1-M fan-out** — the "one" side repeats once per match. **Correct behavior, not a bug.** Fix: **aggregate the M side first** (join to a pre-summed subquery) or join at the right **grain**. ⚠️ **`DISTINCT` masks it** and can silently produce **wrong aggregates**.

**What is a lock?** A **concurrency control** mechanism. **Shared** 📖 = read, many can coexist. **Exclusive** ✍️ = write, only one, blocks all others. **Reads share, writes don't.** (see T18 for optimistic/pessimistic + deadlocks)

**What is data warehousing?** A **central repository** of integrated data from multiple sources, **denormalized** and **column-oriented**, optimized for **analysis and reporting (OLAP)** rather than transactions (OLTP). Data arrives via **ETL/ELT**. (see T44)

**Levels of data abstraction** — physical (how/where stored, ex: indexes) · logical (schema + relationships) · view (the user's slice)
**E-R model** — entity = row · entity type = the schema/attributes · entity set = all rows of that entity · relationships: 1-1, 1-M, M-M
**Intension vs extension** — intension = the **schema/metadata** (stable) · extension = the **data currently stored** (changes on every insert)
**2-tier vs 3-tier** — 2: client -> db directly · 3: client -> server -> db

---

## ✅ Final pre-interview scan

```
[ ] exec order + why WHERE can't see aliases/aggregates
[ ] LEFT+WHERE trap -> put it in ON
[ ] fan-out duplicates -> aggregate first, NOT distinct
[ ] leftmost prefix + func kills the index + covering = index-only scan
[ ] clustered = ONE per table bec. data sorts one way. PG has none.
[ ] EXPLAIN ANALYZE: seq scan on big table / rows removed / estimate vs actual
[ ] N+1: select_related=JOIN, prefetch_related=IN query
[ ] ACID's C = rules | CAP's C = nodes agree
[ ] dirty/non-repeatable/phantom + the 4-level staircase
[ ] deadlock -> wait-for graph, victim, prevent w/ CONSISTENT LOCK ORDER
[ ] normalization = integrity NOT performance
[ ] replica=copy(reads+safety) | shard=split(writes+volume, ZERO safety)
[ ] CAP: P isn't a choice. "when a partition happens, C or A?"
[ ] embed vs ref: "do I want the child without the parent?" + 16MB
[ ] $set patches, replaceOne overwrites
[ ] "if I need SQL, the data is relational — I picked the wrong db"
```

> **Everything hard here came from leaving one machine. Stay on one as long as you can.** 🖥️
