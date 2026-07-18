# SQL Mastery — Interview Q&A

Covers: execution order · JOINs (all + traps) · GROUP BY / HAVING · aggregate vs window · subqueries + CTEs · views · UNION · classic interview problems.
Format: bullets + tables + code + **answer:** in Arabic + English keywords.

---

## 🧭 The One Table We'll Use For Everything

```
customers                orders                        products
─────────────────        ────────────────────────      ─────────────────
id | name  | city        id | customer_id | amount     id | name    | price
1  | Sara  | Cairo       1  | 1           | 100        1  | Mouse   | 30
2  | Ansh  | Giza        2  | 1           | 50         2  | Laptop  | 800
3  | Karim | Cairo       3  | 2           | 200        3  | Keyboard| 60
4  | Milli | Alex        4  | 3           | 30

employees                         departments
──────────────────────────       ──────────────────
id| name | dept_id | manager_id   id | name  | head_id
1 | Sara | 1       | NULL   ← CEO 1  | Eng   | 1  (Sara)
2 | Ali  | 1       | 1            2  | Sales | 3  (Omar)
3 | Omar | 2       | 1
4 | Reem | 2       | 3
```

Notes:
- **Milli has no orders** (used in LEFT JOIN examples).
- `employees.manager_id` → self-reference for org chart (Sara is CEO, no manager).
- `departments.head_id` → FK to employees (who runs the dept).

---

# PART 1 — SQL Execution Order ⭐⭐

### Q1. What's the actual execution order? (surprisingly asked)
```
Written:    SELECT → FROM → JOIN → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT
Executed:   FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT
```

**Why it matters:**
- Can't use a **SELECT alias in WHERE** — WHERE runs before SELECT.
- Can't use **aggregate in WHERE** — WHERE runs before GROUP BY (no groups yet).
- **CAN** use alias in ORDER BY — runs after SELECT.

---

### 🎬 Walkthrough — how a query actually runs

**Tables:**
```
employees:                    departments:
id | name    | dept_id         id | name
1  | Sara    | 1               1  | Eng
2  | Ali     | 1               2  | Sales
3  | Omar    | 2               3  | HR
4  | Reem    | 2
5  | Noor    | 1
6  | Youssef | 3
7  | Malak   | 1
```

**Task:** get departments with more than 2 employees, whose name starts with `S` or `E`, sorted alphabetically.

**Query:**
```sql
SELECT d.name, COUNT(*) AS dept_capacity
FROM departments d
JOIN employees e ON d.id = e.dept_id
WHERE d.name LIKE 'S%' OR d.name LIKE 'E%'
GROUP BY d.name
HAVING COUNT(*) > 2
ORDER BY d.name ASC;
```

**Step-by-step execution:**

**1️⃣ FROM** — start with `departments` (3 rows: Eng, Sales, HR).

**2️⃣ JOIN** — glue `employees` on `d.id = e.dept_id`. No filter yet.
```
d.id | d.name | e.name
1    | Eng    | Sara
1    | Eng    | Ali
1    | Eng    | Noor
1    | Eng    | Malak
2    | Sales  | Omar
2    | Sales  | Reem
3    | HR     | Youssef      ← 7 rows
```

**3️⃣ WHERE** — filter individual rows (`name LIKE 'S%' OR 'E%'`).
HR dropped → 6 rows left.
> ⚡ **Why WHERE now?** cheap filter — reduces work for the expensive GROUP BY next.

**4️⃣ GROUP BY** — bucket rows into groups (nothing thrown away, just organized).
```
Eng   → [Sara, Ali, Noor, Malak]   (4 rows in group)
Sales → [Omar, Reem]               (2 rows in group)
```
> ⚠️ **GROUP BY doesn't filter** — it just organizes rows into buckets. The filtering below (HAVING) is what removes groups.

**5️⃣ HAVING** — filter **whole groups** using aggregates.
```
Eng:   COUNT(*) = 4 > 2 ✅ keep
Sales: COUNT(*) = 2 > 2 ❌ drop
```
> **Why HAVING after GROUP BY?** it needs the aggregates (`COUNT`) that only exist once groups are formed.

**6️⃣ SELECT** — now compute the output columns.
```
Eng | 4
```
> 🚨 **Why SELECT is late:** the alias `dept_capacity` is only born here — that's why you **can't use it in WHERE** (SELECT hasn't run yet).

**7️⃣ ORDER BY** — sort by `d.name ASC`. Alias `dept_capacity` **can** be used here (SELECT already ran).

**8️⃣ LIMIT** — chop the final result. Last step, otherwise you'd cut random rows before sorting.

**Final result:**
```
name | dept_capacity
Eng  | 4
```

**Big-picture reason for this order:**
- **Dependency** — HAVING needs aggregates (from GROUP BY). ORDER BY can use aliases (from SELECT).
- **Performance** — cheap filters (WHERE) run before expensive ops (GROUP BY / SELECT).
- **Logic** — decide *who's in* before computing *what's shown*.

**Analogy:**
- **WHERE** = bouncer at the door (row filter).
- **GROUP BY** = organizer splitting people into city groups (no one leaves, just sorted into rooms).
- **HAVING** = "any room with fewer than 3 people, get out" (group filter).

---

# PART 2 — JOINs ⭐⭐⭐

## 🧪 Practice tables for JOIN exercises

```
customers:                       orders:
──────────────────────           ─────────────────────────────
id | name    | city              id | customer_id | amount | product
1  | Sara    | Cairo             1  | 1           | 100    | Mouse
2  | Ansh    | Giza              2  | 1           | 50     | Keyboard
3  | Karim   | Cairo             3  | 2           | 200    | Laptop
4  | Milli   | Alex              4  | 3           | 30     | Mouse
5  | Youssef | Cairo             5  | 5           | 500    | Laptop

products:                        employees:
──────────────────               ───────────────────────────────
id | name    | price             id | name | salary | manager_id
1  | Mouse   | 30                1  | Sara | 8000   | NULL   ← CEO
2  | Keyboard| 60                2  | Ali  | 5000   | 1
3  | Laptop  | 800               3  | Omar | 6000   | 1
                                 4  | Reem | 4000   | 2
                                 5  | Noor | 7000   | 1
```

**Notes:**
- **Milli** has no orders.
- **Keyboard** appears in orders (product name), but the `products` table has it too.
- **Sara = CEO** (no manager).

**Practice questions:**
1. **INNER** — customer names + their order amounts (skip customers with no orders).
2. **LEFT** — every customer + their total spending (0 if none).
3. **LEFT + trap** — every customer + only orders > 40 (all customers still show).
4. **RIGHT / LEFT reversed** — every product + all its orders (products with no orders still show).
5. **FULL OUTER** — all customers + all orders, matched or not.
6. **CROSS** — every possible (customer, product) pair.
7. **SELF** — each employee + their manager's name (CEO shows with NULL).
8. **SELF advanced** — employees who earn more than their manager.

---

## The Golden Question
> **"When a row has no partner, what do I do with it?"**

That's literally the only difference between join types.

### Q2. All join types + when to use each
```
INNER JOIN   → keep only matched rows (drop lonely from both)
LEFT JOIN    → keep ALL left rows (NULL-pad the right if no match)
RIGHT JOIN   → keep ALL right rows (mirror of LEFT — rare, just swap)
FULL OUTER   → keep everything from both sides
CROSS JOIN   → cartesian product (every A × every B)
SELF JOIN    → table joined to itself (hierarchies)
```

### 🔵 INNER JOIN — only matched rows
```sql
SELECT c.name, o.amount
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id;
```
Result:
```
Sara  | 100
Sara  | 50
Ansh  | 200
Karim | 30
```
❌ **Milli disappears** (no orders → no match → dropped).

### 🔵 LEFT JOIN — keep ALL left rows
```sql
SELECT c.name, o.amount
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;
```
Result:
```
Sara  | 100
Sara  | 50
Ansh  | 200
Karim | 30
Milli | NULL    ← preserved with NULL
```

### 🔵 RIGHT JOIN — mirror of LEFT
Rare — just swap table order and use LEFT for readability.

### 🔵 FULL OUTER JOIN — everything from both
```
Left + Right + matched. Unmatched rows get NULL on the other side.
```
Use when you need "all customers + all orders, whether they match or not."

### 🔵 CROSS JOIN — cartesian product
```sql
SELECT c.name, p.name FROM customers c CROSS JOIN products p;
-- 4 customers × 3 products = 12 rows
```
Every combination. Usually **wrong** if you didn't mean it.

### 🔵 SELF JOIN — same table twice
```sql
-- Employee → their manager (using employees.manager_id)
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
--                       ↑ same table, different alias
-- LEFT because the CEO has no manager
```


---

## 🚨 Trap 1 — LEFT + WHERE بيبقى INNER

```sql
-- ❌ Bug: Milli disappears!
SELECT c.name, o.amount
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.amount > 40;
```
Milli's `o.amount = NULL` → `NULL > 40` = NULL (not TRUE) → row dropped → **LEFT quietly became INNER**.

```sql
-- ✅ Fix: put the condition in ON
LEFT JOIN orders o ON c.id = o.customer_id AND o.amount > 40
```

**Rule:**
> **ON** decides what MATCHES. **WHERE** decides what SURVIVES — and WHERE runs after.
> Condition on the **right (optional)** table → **ON**.
> Condition on the **left (preserved)** table → **WHERE**.


---

## 🔁 Fan-out — 1-to-many duplication

في join 1-to-many، الـ "one" بيتكرّر **مرة لكل match** في الـ "many". **مش bug** — سلوك طبيعي.

**مثال:** Sara عندها 2 orders → Sara بتظهر **مرتين**.
```
Sara  | 100
Sara  | 50       ← Sara اتكرّرت لأن عندها 2 orders
Ansh  | 200
```

القاعدة:
> customer عنده **N orders** → بيظهر **N مرات** في النتيجة.


---

# PART 3 — GROUP BY + HAVING vs WHERE ⭐⭐

### Q3. GROUP BY — what does it do?
Collapses rows sharing a value into **one row per group**, then apply aggregates.

```sql
SELECT customer_id, COUNT(*) AS order_count, SUM(amount) AS total
FROM orders
GROUP BY customer_id;
```
```
customer_id | order_count | total
1           | 2           | 150
2           | 1           | 200
3           | 1           | 30
```

**Rule:** every column in SELECT that isn't in an aggregate **must be in GROUP BY**.

### Q4. WHERE vs HAVING ⭐⭐
```
WHERE    → filters INDIVIDUAL ROWS (before grouping)
HAVING   → filters GROUPS (after grouping)
```

- `WHERE` uses raw columns (`salary > 5000`).
- `HAVING` uses aggregates (`COUNT(*) > 5`, `AVG(salary) > 4000`).

```sql
-- Get departments with more than 3 employees earning > 5000
SELECT dept_id, COUNT(*) AS emp_count
FROM employees
WHERE salary > 5000     -- filter INDIVIDUALS first
GROUP BY dept_id
HAVING COUNT(*) > 3;    -- then filter GROUPS
```

### 🚨 The Classic Trap
```sql
-- ❌ Won't work: WHERE runs before GROUP BY → no COUNT yet
SELECT customer_id FROM orders WHERE COUNT(*) > 5 GROUP BY customer_id;

-- ✅ Correct
SELECT customer_id FROM orders GROUP BY customer_id HAVING COUNT(*) > 5;
```

### 🚨 Perf Trap — put row-level conditions in WHERE
```sql
-- ❌ groups everything, then throws Eng away — wasted work
SELECT dept, AVG(salary) FROM employees GROUP BY dept HAVING dept <> 'Eng';

-- ✅ discards Eng rows BEFORE grouping (cheaper)
SELECT dept, AVG(salary) FROM employees WHERE dept <> 'Eng' GROUP BY dept;
```

**Rule of thumb:** filter **as early as possible**.


---

### Q5. COUNT(*) vs COUNT(column) vs COUNT(DISTINCT column)
```
COUNT(*)              → all rows (including NULLs)
COUNT(column)         → non-NULL values only
COUNT(DISTINCT column)→ unique non-NULL values
```

Example: emails column with 5 rows, 2 NULLs, 3 unique values:
```
COUNT(*)              → 5
COUNT(email)          → 3
COUNT(DISTINCT email) → 3
```

⚠️ **Aggregates ignore NULLs** — except `COUNT(*)`.

---

# PART 4 — Aggregate vs Window Functions ⭐⭐

### Q6. What's the difference?
```
AGGREGATE 🗜️  → 5 rows → 1 row     rows GONE     "what's the total?"
WINDOW    🪟  → 5 rows → 5 rows    rows STAY     "how do I compare?"
```
**Same functions** (SUM/AVG/COUNT/MIN/MAX). **The only difference is the word `OVER`.**

### Example
```sql
-- AGGREGATE: collapses rows
SELECT dept_id, AVG(salary) FROM employees GROUP BY dept_id;
-- dept 1 | 4000
-- dept 2 | 6000

-- WINDOW: keeps every row + adds the avg
SELECT name, dept_id, salary, AVG(salary) OVER (PARTITION BY dept_id) AS dept_avg
FROM employees;
-- Sara | 1 | 3000 | 4000
-- Ali  | 1 | 5000 | 4000
-- Omar | 2 | 6000 | 6000
```

🔑 **`PARTITION BY` = `GROUP BY` that doesn't collapse.**

### Window Functions Cheat Sheet
```
ROW_NUMBER()   → 1, 2, 3           unique sequential
RANK()         → 1, 1, 3 (skip)    ties tie, gaps
DENSE_RANK()   → 1, 1, 2 (no skip) ties tie, no gaps
LAG(col)       → previous row's value
LEAD(col)      → next row's value
SUM(col) OVER (ORDER BY ...) → running total
```

### Common patterns
```sql
-- Top N per group
WITH ranked AS (
  SELECT name, dept_id, salary,
         ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn
  FROM employees
)
SELECT * FROM ranked WHERE rn <= 3;

-- Running total per customer
SELECT customer_id, amount, created_at,
       SUM(amount) OVER (PARTITION BY customer_id ORDER BY created_at) AS running_total
FROM orders;

-- Month-over-month growth
SELECT month, revenue, LAG(revenue) OVER (ORDER BY month) AS last_month,
       revenue - LAG(revenue) OVER (ORDER BY month) AS growth
FROM monthly_sales;
```

### 🚨 Trap — window functions can't go in WHERE
```sql
-- ❌ WHERE runs before windows are computed
SELECT * FROM employees WHERE ROW_NUMBER() OVER (...) = 1;

-- ✅ Wrap in a CTE, then filter
WITH ranked AS (SELECT *, ROW_NUMBER() OVER (...) AS rn FROM employees)
SELECT * FROM ranked WHERE rn = 1;
```


---

# PART 5 — Subqueries + CTEs ⭐⭐

### Q7. Subquery types
```
Non-correlated → independent, runs ONCE
Correlated     → references outer row → runs PER outer row (slow N²)
Scalar         → returns 1 value (usable inline)
```

```sql
-- Non-correlated (fast — runs once)
SELECT name FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);

-- Correlated (slow — runs per row)
SELECT name FROM employees e1
WHERE salary > (SELECT AVG(salary) FROM employees e2 WHERE e2.dept_id = e1.dept_id);
--                                                                      ↑ references outer
```

### 🚨 Trap — correlated subqueries are often replaceable by JOINs (faster)
```sql
-- ❌ Correlated
SELECT c.name, (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.id) AS order_count
FROM customers c;

-- ✅ JOIN + GROUP BY (one pass)
SELECT c.name, COUNT(o.id) AS order_count
FROM customers c LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;
```

### Q8. What is a CTE? (`WITH ... AS`)
A **named**, reusable subquery. **Read top-to-bottom** instead of inside-out.
```sql
WITH top_customers AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders GROUP BY customer_id
    HAVING SUM(amount) > 100
)
SELECT c.name, tc.total
FROM top_customers tc JOIN customers c ON c.id = tc.customer_id;
```

**CTE vs Subquery:**
- Perf: **basically equal**. CTE is for **humans**.
- **CTE can be referenced twice; subquery must be written twice.**
- CTEs support **recursion** (`WITH RECURSIVE`) — org charts, category trees.

### Recursive CTE (hierarchies)
```sql
WITH RECURSIVE emp_tree AS (
    -- Base case: top-level (no manager)
    SELECT id, name, manager_id, 1 AS level
    FROM employees WHERE manager_id IS NULL

    UNION ALL

    -- Recursive step
    SELECT e.id, e.name, e.manager_id, t.level + 1
    FROM employees e JOIN emp_tree t ON e.manager_id = t.id
)
SELECT * FROM emp_tree;
```


---

# PART 6 — Views + Materialized Views ⭐

### Q9. What's a view?
A **saved query** wearing a table costume. **Stores no data** — re-runs on every access.

```sql
CREATE VIEW active_customers AS
SELECT * FROM customers WHERE last_login > NOW() - INTERVAL '30 days';

-- Now query it like a table
SELECT * FROM active_customers WHERE city = 'Cairo';
```

**When to use views:**
- **Simplify** — hide gnarly joins behind a name.
- **Security** — grant read on the view, hide sensitive columns.
- **Consistency** — one definition of "active customer" used everywhere.

### View vs Materialized View
| | View | Materialized View |
|--|------|-------------------|
| Stores data? | ❌ no | ✅ yes (snapshot) |
| Runs query | every access | only on REFRESH |
| Speed | slow (query each time) | instant |
| Freshness | always current | can be stale |
| Refresh | not needed | manual (`REFRESH MATERIALIZED VIEW`) |

**Rule of thumb:** *"can this data be a few minutes old?"* → yes = matview, no = view.

### 🚨 Trap — "views improve performance" = ❌
The query runs every time. Only a **materialized view** saves the result.


---

# PART 7 — UNION / INTERSECT / EXCEPT

### Q10. Stack rows (not join columns)
```
JOIN  → widens (more columns)
UNION → stacks (more rows)
```

```
UNION     → combines + REMOVES duplicates (slower — does DISTINCT)
UNION ALL → combines + KEEPS duplicates (faster — just appends)
INTERSECT → rows in BOTH queries
EXCEPT    → rows in first, NOT in second (MINUS in Oracle)
```

**Rules to stack:** same **number of columns**, compatible **types**, same **order** (names don't have to match).

**Perf answer:** **default to UNION ALL** — UNION does a sort/hash pass to dedup. Only use UNION if you actually need dedup.


---

# PART 8 — Classic Interview Problems ⭐⭐⭐

## Problem 1 — Nth highest salary
```sql
-- 2nd highest salary overall
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC LIMIT 1 OFFSET 1;

-- Nth highest per department
WITH ranked AS (
    SELECT name, dept_id, salary,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM employees
)
SELECT * FROM ranked WHERE rnk = 2;
```

## Problem 2 — Users who never ordered
```sql
-- Method 1: LEFT JOIN + IS NULL (usually fastest)
SELECT c.* FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;

-- Method 2: NOT EXISTS (also fast, safer with NULLs)
SELECT c.* FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);

-- Method 3: NOT IN (⚠️ trap: NULL in subquery → returns nothing!)
SELECT c.* FROM customers c
WHERE c.id NOT IN (SELECT customer_id FROM orders WHERE customer_id IS NOT NULL);
```

## Problem 3 — Top spender per city
```sql
WITH ranked AS (
    SELECT c.name, c.city, SUM(o.amount) AS total,
           ROW_NUMBER() OVER (PARTITION BY c.city ORDER BY SUM(o.amount) DESC) AS rn
    FROM customers c JOIN orders o ON c.id = o.customer_id
    GROUP BY c.id, c.name, c.city
)
SELECT name, city, total FROM ranked WHERE rn = 1;
```

## Problem 4 — Duplicates (find + remove)
```sql
-- Find
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;

-- Remove (keep earliest)
DELETE FROM users
WHERE id NOT IN (SELECT MIN(id) FROM users GROUP BY email);
```

⚠️ **Trap:**
```sql
-- ❌ Deletes BOTH copies — Sara disappears!
DELETE FROM users WHERE email IN (SELECT email FROM users GROUP BY email HAVING COUNT(*)>1);
```
**Detect by VALUE. Delete by ROW ID.**

## Problem 5 — Consecutive dates / gaps
```sql
-- Login streaks — find gaps
SELECT user_id, login_date,
       LAG(login_date) OVER (PARTITION BY user_id ORDER BY login_date) AS prev,
       login_date - LAG(login_date) OVER (PARTITION BY user_id ORDER BY login_date) AS gap_days
FROM logins;
```

## Problem 6 — Month-over-month change
```sql
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS last_month,
       revenue - LAG(revenue) OVER (ORDER BY month) AS diff,
       ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month))
             / LAG(revenue) OVER (ORDER BY month), 2) AS pct_change
FROM monthly_sales;
```

## Problem 7 — Pivot rows to columns
```sql
-- Category counts per month (Postgres)
SELECT month,
       COUNT(*) FILTER (WHERE category = 'A') AS a,
       COUNT(*) FILTER (WHERE category = 'B') AS b,
       COUNT(*) FILTER (WHERE category = 'C') AS c
FROM sales GROUP BY month;

-- Portable: CASE-based
SELECT month,
       SUM(CASE WHEN category = 'A' THEN 1 ELSE 0 END) AS a,
       SUM(CASE WHEN category = 'B' THEN 1 ELSE 0 END) AS b
FROM sales GROUP BY month;
```

## Problem 8 — Self join for pairs (e.g., same city)
```sql
-- Find pairs of customers in the same city
SELECT c1.name AS person1, c2.name AS person2, c1.city
FROM customers c1 JOIN customers c2
     ON c1.city = c2.city AND c1.id < c2.id;
--                          ↑ avoids duplicate pairs and self-pairs
```

---

# PART 9 — NULL Handling

### Q11. Why is `NULL = NULL` not TRUE?
NULL means **UNKNOWN** — two unknowns can't be compared.
- `NULL = NULL` → **NULL** (not TRUE!)
- Always use `IS NULL` / `IS NOT NULL`.
- Aggregates **ignore NULLs** (except `COUNT(*)`).

### COALESCE — first non-NULL
```sql
COALESCE(NULL, NULL, 5, 10) → 5
SELECT name, COALESCE(phone, email, 'no contact') FROM users;
```

### NULLIF — return NULL if two values equal
```sql
NULLIF(a, 0) → NULL if a=0, else a  -- avoid divide-by-zero
SELECT total / NULLIF(count, 0) FROM stats;
```

### 🚨 Trap — NOT IN + NULL
```sql
-- ❌ Returns nothing if subquery has a NULL
SELECT * FROM customers WHERE id NOT IN (SELECT customer_id FROM orders);
-- If any customer_id is NULL, entire result is empty!

-- ✅ Safer
SELECT * FROM customers c WHERE NOT EXISTS (SELECT 1 FROM orders WHERE customer_id = c.id);
```


---

# PART 10 — Quick Syntax Reference

### DDL
```sql
CREATE TABLE t (id SERIAL PRIMARY KEY, name TEXT NOT NULL, email TEXT UNIQUE);
ALTER TABLE t ADD COLUMN age INT;
ALTER TABLE t DROP COLUMN age;
DROP TABLE t;
TRUNCATE TABLE t;
```

### DML
```sql
INSERT INTO t (name, email) VALUES ('Sara', 'sara@x.com');
UPDATE t SET name = 'Sarah' WHERE id = 1;
DELETE FROM t WHERE id = 1;

-- Upsert (Postgres)
INSERT INTO t (id, name) VALUES (1, 'Sara')
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;
```

### SELECT skeleton
```sql
SELECT col1, agg_func(col2) AS alias
FROM t1
JOIN t2 ON t1.id = t2.t1_id
WHERE condition
GROUP BY col1
HAVING agg_condition
ORDER BY col1 DESC
LIMIT 10 OFFSET 20;
```

### Constraints
```sql
PRIMARY KEY | FOREIGN KEY (...) REFERENCES t(id) ON DELETE CASCADE
UNIQUE | NOT NULL | CHECK (age >= 18) | DEFAULT 0
```

---

## ✅ SQL Self-Test (say out loud)
1. Execution order — recite it in order.
2. All join types + when each.
3. LEFT + WHERE trap — explain + fix.
4. Fan-out duplicate trap — why + fix.
5. WHERE vs HAVING.
6. Why `WHERE COUNT(*) > 5` fails.
7. Aggregate vs window function.
8. Why windows can't go in WHERE.
9. Correlated vs non-correlated subquery.
10. CTE vs subquery.
11. View vs materialized view.
12. UNION vs UNION ALL.
13. 2nd highest salary per department — write it.
14. Users who never ordered — 3 methods.
15. Why `NULL = NULL` is not TRUE.
16. NOT IN + NULL trap.

---

## 🎯 The Golden Rules (memorize)
1. **Execution order** — SELECT is written first, executed almost last.
2. **JOIN type** = your policy on unmatched rows.
3. **LEFT+WHERE on right table** = INNER JOIN in disguise.
4. **Fan-out duplicates** are correct behavior — don't slap DISTINCT.
5. **Filter early** — row-level in WHERE, not HAVING.
6. **Window functions** = aggregates that don't collapse rows.
7. **Windows can't be in WHERE** — wrap in a CTE.
8. **UNION ALL by default** — UNION does a costly dedup.
9. **Detect by value, delete by ID.**
10. **NULL is contagious** — `IS NULL`, `COALESCE`, `NOT EXISTS` over `NOT IN`.
