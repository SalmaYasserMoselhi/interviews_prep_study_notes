# Databases Deep Dive: SQL, NoSQL, and Caching — A Complete, Hands-On Curriculum

Sources: [postgresql.org/docs/current](https://www.postgresql.org/docs/current/index.html), [mongodb.com/docs](https://www.mongodb.com/docs/), [redis.io/docs](https://redis.io/docs/latest/).
Pairs with `react-nextjs-roadmap.md` and `python-django-postgres-roadmap.md` to complete the full-stack picture — this file goes *underneath* those, into the database engines themselves: how they actually work, not just how to call them from an ORM.

**How every topic in this file is taught:** concept + intuition → a runnable code snippet → an exercise. Do the exercise before moving on — this is a "follow-through," not a reference doc to skim. PostgreSQL and MongoDB are the two databases used for every concrete snippet; Redis closes the file as the caching layer.

---

# PART 0 — FOUNDATIONS: HOW TO THINK ABOUT DATABASES BEFORE PICKING ONE

## 0.1 What a database actually has to guarantee

**Intuition:** A database isn't just "a place that stores data" — it's a system that makes *promises* about that data under failure. A spreadsheet stores data too, but it doesn't promise anything when two people edit it at once, or when your laptop loses power mid-save. The entire design of SQL vs. NoSQL databases comes down to *which promises each one chooses to make, and which it deliberately gives up* in exchange for something else (usually scale, speed, or flexibility). Everything else in this file — schemas, indexes, replication, sharding, caching — is in service of either keeping a promise or trading it away on purpose.

**Exercise:** Before writing any code, write down (on paper or in a notes file) three real-world systems you've used — a banking app, a social media feed, a multiplayer game leaderboard — and for each, guess: "if two updates happen at the exact same instant, what's the worst thing that could go wrong, and would the user even notice?" Keep these three examples in mind; you'll revisit them once you understand ACID and BASE.

---

## 0.2 ACID — the relational database's promise

**Intuition:** ACID is a checklist of four guarantees a transaction makes. A "transaction" is a group of operations that should succeed or fail *as one unit* — think "transfer $100 from account A to account B," which is really two writes (debit A, credit B) that must never be observed half-done.

- **Atomicity** — the transaction happens entirely or not at all. If the credit to B fails after the debit from A succeeded, the whole thing rolls back; A keeps its money.
- **Consistency** — the transaction can only move the database from one valid state to another valid state, per its constraints (e.g., a `CHECK (balance >= 0)` constraint can never be violated, even mid-transaction-commit).
- **Isolation** — concurrent transactions don't see each other's half-finished work. Two transfers happening at once don't corrupt each other's view of account balances.
- **Durability** — once a transaction commits, it survives a crash. The disk (or WAL — see Section 9) guarantees it.

The word to really sit with is **isolation**, because it's the one with the most nuance (you'll get the isolation levels in Section 8 — `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`). Atomicity, Consistency, and Durability are mostly binary — either the DB has them or it doesn't. Isolation has *degrees*, and that's where most real-world subtlety and bugs live.

**Code snippet (PostgreSQL) — atomicity and durability in action:**
```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Imagine the application crashes right here, before COMMIT.
-- On restart, PostgreSQL guarantees NEITHER update happened.
-- Nothing is half-applied. That's atomicity.

COMMIT;
-- Once this returns successfully, the change survives a server crash,
-- a power outage, anything short of the disk itself being destroyed.
-- That's durability, backed by the Write-Ahead Log (Section 9).
```

**Exercise:** Open `psql` against a scratch database. Create an `accounts` table with `id` and `balance` columns, insert two rows with balances of `500` each. Run a `BEGIN`, do the two `UPDATE`s above, then **don't commit** — open a second `psql` session and `SELECT * FROM accounts;` from there. Confirm you see the *original* balances in the second session (that's isolation — the uncommitted change is invisible to others). Then go back to the first session and run `ROLLBACK;` instead of `COMMIT;`. Confirm the balances are unchanged in both sessions. You've just watched atomicity and isolation happen, not just read about them.

---

## 0.3 BASE — the distributed/NoSQL database's promise

**Intuition:** BASE is ACID's intentional opposite-leaning sibling, and it exists because **the CAP theorem** says you can't have perfect Consistency and Availability during a network Partition — you must choose. Relational databases (running on one strong primary) lean toward consistency. Many NoSQL databases, built to scale horizontally across many machines, lean toward availability — and BASE is the philosophy that results:

- **Basically Available** — the system responds to every request, even during a partial failure, even if the answer might be stale.
- **Soft state** — the state of the system may change over time even without new input, as replicas catch up with each other.
- **Eventual consistency** — if you stop writing, all replicas will *eventually* agree. Not now. Eventually.

The intuition that should stick: ACID asks "is this answer *correct, right now*?" BASE asks "is this answer *fast and available, and will it become correct soon*?" Neither is "better" in the abstract — a bank balance wants ACID; a "likes" counter on a social post is perfectly happy being eventually consistent (nobody cares if it shows 4,021 instead of 4,022 for half a second).

**Code snippet (MongoDB) — observing eventual consistency conceptually:**
```javascript
// In a replica set, a write acknowledged by the primary may not yet
// be visible on a secondary if you read with a relaxed read preference.
// This single line is where you make the ACID/BASE trade-off explicit
// in MongoDB: you choose how strong a guarantee you want, per query.

db.products.find({ sku: "ABC123" }).readPref("secondaryPreferred")
// vs.
db.products.find({ sku: "ABC123" }).readPref("primary")
// The first may return slightly stale data but spreads load and stays
// available even if the primary is briefly unreachable.
// The second is always fresh, but only available if the primary is up.
```

**Exercise:** Write a one-paragraph answer (no code needed) for each of your three systems from Exercise 0.1: which of ACID or BASE would you choose for the *core* write path of each, and why? (Hint: a banking transfer almost certainly wants ACID; a "likes" counter or "currently online" status almost certainly is fine with BASE. The multiplayer leaderboard is the interesting, debatable one — argue it both ways.)

---

## 0.4 What each category of database actually focuses on

**Intuition:** "SQL vs. NoSQL" is a less useful split than thinking about what each database is *optimized to make easy*:

| Category | Optimized for | Example | Trade-off it accepts |
|---|---|---|---|
| **Relational (SQL)** | Structured data with strict relationships, complex joins, strong consistency | PostgreSQL, MySQL | Schema rigidity; harder to scale writes horizontally |
| **Document** | Flexible, nested, evolving schemas; data accessed as a whole "thing" | MongoDB | Joins across documents are awkward; consistency is tunable, not default-strong |
| **Key-Value** | Raw speed, simple lookups by key | Redis, DynamoDB | No querying by value, no relationships |
| **Wide-Column** | Massive write throughput, time-series-like data at huge scale | Cassandra, HBase | Query patterns must be designed upfront; ad-hoc queries are hard |
| **Graph** | Relationships *themselves* are the primary data (friend-of-friend, recommendation paths) | Neo4j | Bad fit for simple tabular data; specialized query language |
| **In-memory cache** | Sub-millisecond access to hot data | Redis, Memcached | Volatile by default; not a system of record |

This file goes deep on **Relational (PostgreSQL)**, **Document (MongoDB)**, and **In-memory cache (Redis)** — the three you'll actually reach for in the overwhelming majority of real projects, including the Django + Next.js stack from the other two roadmaps.

**Exercise:** For each of the six categories above, name one piece of data from a typical e-commerce app (`users`, `orders`, `product catalog`, `shopping cart`, `search-as-you-type suggestions`, `"customers who bought this also bought"`, `session tokens`) that fits it best. You should end up using every category at least once — that's the point: real systems are usually **polyglot persistence** (multiple database types working together), not "pick one database for everything."

---

# PART 1 — SQL & POSTGRESQL: THE RELATIONAL MODEL

> This part follows PostgreSQL's own documentation structure: Data Definition → Data Manipulation → Queries → Data Types → Functions & Operators → Indexes → Full Text Search → Concurrency Control → Performance.

## 1.1 The Relational Model & Why Tables

**Intuition:** A relational database stores data as **relations** (tables), where every row has the same columns, and every value is atomic (no nested structures, classically — though Postgres bends this rule with `JSONB`, more on that in 1.10). The power of this model isn't the table shape itself — it's that you can define **relationships between tables via shared keys**, and the database *enforces* those relationships for you. This is the foundational idea you must internalize before anything else makes sense: a relational database is a network of small, focused tables, each describing one *kind* of thing, wired together by foreign keys.

**Code snippet — creating a small relational schema:**
```sql
CREATE TABLE authors (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    author_id INTEGER NOT NULL REFERENCES authors(id),
    published_year INTEGER
);
```

The `REFERENCES authors(id)` line is the entire relational model in one phrase: it tells Postgres "a book cannot exist pointing to an author that doesn't exist," and Postgres will reject any `INSERT`/`UPDATE` that violates it. That's not your application's job to enforce — it's the database's.

**Exercise:** Create these two tables in a scratch database. Try to insert a book with `author_id = 999` (an author that doesn't exist) and observe the error. Then insert a real author, get their `id`, and insert a book referencing it successfully.

---

## 1.2 Data Definition: Tables, Constraints, and Schema Design

**Intuition:** "Data Definition" (DDL — `CREATE`, `ALTER`, `DROP`) is where you encode your business rules *into the schema itself* rather than trusting application code to enforce them everywhere. The earlier you push a rule into a constraint, the fewer ways your data can ever become invalid — even from a bug, a forgotten check, or a future developer who didn't read your validation code.

**Code snippet — constraints doing real work:**
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_email TEXT NOT NULL CHECK (customer_email LIKE '%@%'),
    total_cents INTEGER NOT NULL CHECK (total_cents >= 0),
    status TEXT NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending', 'paid', 'shipped', 'cancelled')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Four different constraint types are doing four different jobs here: `NOT NULL` (a value must exist), `CHECK` (a value must satisfy a condition — including a crude "enum" via `IN (...)`), `DEFAULT` (a sensible value when none is given), and the implicit `PRIMARY KEY` uniqueness + not-null guarantee. None of this logic lives in your Python/JS code — it's structurally impossible to violate, no matter what inserts the row.

**Exercise:** Add a `CHECK` constraint to the `orders` table that ensures `total_cents` is `0` only if `status = 'cancelled'` (i.e., non-cancelled orders must have a positive total). Hint: you'll need a multi-column `CHECK`. Try inserting a row that violates it and confirm Postgres rejects it.

---

## 1.3 Data Manipulation: INSERT, UPDATE, DELETE, UPSERT

**Intuition:** DML is how data actually moves in and out of tables. The subtlety beginners miss: `UPDATE`/`DELETE` without a `WHERE` clause affect *every row* — there's no "are you sure?" prompt in SQL. The other subtlety: **UPSERT** (`INSERT ... ON CONFLICT`) exists because "insert this, but if it already exists, update it instead" is an extremely common pattern that's *not atomic* if you write it as a separate `SELECT` + `INSERT`-or-`UPDATE` in application code (another request could insert between your check and your action — a race condition).

**Code snippet:**
```sql
INSERT INTO books (title, author_id, published_year)
VALUES ('Dune', 1, 1965);

UPDATE books SET published_year = 1965 WHERE title = 'Dune';

DELETE FROM books WHERE id = 42;

-- UPSERT: insert a book, but if one with this exact title+author
-- already exists (assumes a UNIQUE constraint on (title, author_id)),
-- just bump its published_year instead -- atomically, in one round trip.
INSERT INTO books (title, author_id, published_year)
VALUES ('Dune', 1, 1965)
ON CONFLICT (title, author_id)
DO UPDATE SET published_year = EXCLUDED.published_year;
```

`EXCLUDED` is the special name Postgres gives the row that *would have been inserted*, so you can reference its values inside the `DO UPDATE`.

**Exercise:** Add a `UNIQUE (title, author_id)` constraint to `books`. Run the UPSERT snippet twice in a row and confirm there's still only one `Dune` row after both runs (check with `SELECT * FROM books WHERE title = 'Dune';`). Then explain, in one sentence, why doing this as "check if exists, then decide to insert or update" from application code instead would be unsafe under concurrent requests.

---

## 1.4 Queries: SELECT, WHERE, ORDER BY, JOINs

**Intuition:** `SELECT` is where the relational model pays off — you can ask questions that *span multiple tables* without manually stitching the data together yourself. **JOIN is the single most important SQL concept to deeply understand**, because every join type answers a subtly different question:
- `INNER JOIN`: "rows that have a match on both sides" (most common)
- `LEFT JOIN`: "all rows from the left table, plus matches from the right if they exist, else `NULL`" (use this when you want "all authors, even ones with zero books")
- `RIGHT JOIN`: the mirror of LEFT (rarely used — just swap table order and use LEFT instead, for readability)
- `FULL OUTER JOIN`: "everything from both sides, matched where possible"

**Code snippet:**
```sql
-- INNER JOIN: only authors who have written at least one book
SELECT authors.name, books.title
FROM authors
INNER JOIN books ON books.author_id = authors.id;

-- LEFT JOIN: every author, even ones with zero books (title will be NULL for them)
SELECT authors.name, books.title
FROM authors
LEFT JOIN books ON books.author_id = authors.id;

-- Filtering + sorting
SELECT authors.name, books.title, books.published_year
FROM authors
INNER JOIN books ON books.author_id = authors.id
WHERE books.published_year >= 1960
ORDER BY books.published_year DESC;
```

**Exercise:** Insert an author with zero books. Run both the `INNER JOIN` and `LEFT JOIN` versions of the first query and confirm: the author with no books is *missing* from the `INNER JOIN` result, but *present with a `NULL` title* in the `LEFT JOIN` result. This single observed difference is the most important "aha" in basic SQL — don't move on until you've seen it with your own eyes.

---

## 1.5 Aggregation: GROUP BY, HAVING, and Aggregate Functions

**Intuition:** Aggregation answers "questions about groups of rows" rather than individual rows — "how many books per author," "average order total per month." `GROUP BY` collapses rows sharing a value into one row per group, and aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) compute something across each group. `HAVING` is `WHERE`, but *for groups* — you can't filter on an aggregate result using `WHERE` because `WHERE` runs before grouping happens.

**Code snippet:**
```sql
SELECT authors.name, COUNT(books.id) AS book_count
FROM authors
LEFT JOIN books ON books.author_id = authors.id
GROUP BY authors.id, authors.name
HAVING COUNT(books.id) > 1
ORDER BY book_count DESC;
```

**Exercise:** Add a second and third book for one author, so that author has 3 books while others have 0 or 1. Run the query above and confirm only the 3-book author appears (because of `HAVING COUNT(books.id) > 1`). Then try replacing `HAVING` with `WHERE COUNT(books.id) > 1` and observe the error -- explain in your own words why Postgres rejects it.

---

## 1.6 Subqueries, CTEs, and Window Functions

**Intuition:** Sometimes a single flat query can't express the question. **Subqueries** let you nest a query inside another. **CTEs** (`WITH ... AS`, Common Table Expressions) are named, readable subqueries — they turn a tangled nested query into a sequence of named steps, like naming intermediate variables in code. **Window functions** are the most powerful and most underused tool in this section: they let you compute something *across a set of rows related to the current row* (a "window") **without collapsing rows the way `GROUP BY` does** — you keep every row, but each one gets an extra computed column like "this row's rank within its group" or "running total so far."

**Code snippet:**
```sql
-- CTE: readable multi-step query
WITH prolific_authors AS (
    SELECT author_id, COUNT(*) AS book_count
    FROM books
    GROUP BY author_id
    HAVING COUNT(*) > 1
)
SELECT authors.name, prolific_authors.book_count
FROM authors
JOIN prolific_authors ON prolific_authors.author_id = authors.id;

-- Window function: rank each book by year, within its author, WITHOUT collapsing rows
SELECT
    title,
    author_id,
    published_year,
    RANK() OVER (PARTITION BY author_id ORDER BY published_year) AS rank_within_author
FROM books;
```

`PARTITION BY` is to window functions what `GROUP BY` is to aggregates — except the rows aren't merged away.

**Exercise:** Using the window function snippet, add a `running_total` column to a hypothetical `orders` table that sums `total_cents` per customer, ordered by `created_at`, using `SUM(total_cents) OVER (PARTITION BY customer_email ORDER BY created_at)`. Run it and confirm each customer's rows show a cumulative total that resets per customer. This is the exact pattern behind "show me my account balance history" features.

---

## 1.7 Data Types

**Intuition:** Choosing the right column type isn't pedantry — it determines what constraints the database can enforce for free, how much space rows take, and what operations (and indexes) are even possible. Postgres has an unusually rich type system compared to most relational databases, and using it well is a real skill.

**Code snippet — a tour of the types you'll actually use:**
```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),  -- globally unique IDs, good for distributed systems
    name TEXT NOT NULL,
    price NUMERIC(10, 2) NOT NULL,                  -- exact decimal -- NEVER use FLOAT for money
    in_stock BOOLEAN NOT NULL DEFAULT true,
    tags TEXT[],                                     -- native array type
    metadata JSONB,                                  -- schemaless escape hatch, binary-stored JSON
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()    -- ALWAYS store timestamps with timezone
);

INSERT INTO products (name, price, tags, metadata)
VALUES (
    'Wireless Mouse',
    29.99,
    ARRAY['electronics', 'accessories'],
    '{"color": "black", "wireless": true}'::jsonb
);
```

The single most common real-world bug this prevents: using `FLOAT`/`DOUBLE` for money. Floats can't represent `0.1` exactly in binary, so summed totals drift by fractions of a cent over time. `NUMERIC` stores exact decimal digits — always use it for currency.

**Exercise:** Create the `products` table above. Insert two rows with prices `0.10` and `0.20`. Run `SELECT SUM(price) FROM products;` and confirm you get exactly `0.30`. Then, just to see the failure mode for yourself, create a throwaway column `price_float FLOAT`, insert the same two values, and sum *that* column instead -- in many languages/contexts you'd see a result like `0.30000000000000004`. (Postgres's `FLOAT` may print `0.3` due to display rounding, so if you want to really see the drift, do the equivalent sum in a Python REPL: `0.1 + 0.2`.)

---

## 1.8 Functions and Operators

**Intuition:** Postgres ships hundreds of built-in functions so you can push computation *into the database*, next to the data, instead of pulling raw rows into your application and computing there. This matters for performance (less data over the wire) and correctness (the database can use indexes to speed up function calls in some cases, like text search).

**Code snippet:**
```sql
SELECT
    UPPER(name) AS loud_name,
    LENGTH(name) AS name_length,
    price * 1.08 AS price_with_tax,
    DATE_TRUNC('month', created_at) AS created_month,
    COALESCE(tags, ARRAY[]::TEXT[]) AS safe_tags  -- handle NULL gracefully
FROM products;
```

`COALESCE` is worth special attention: it returns the first non-`NULL` argument, and it's your main tool for "give me a default if this is missing" logic directly in SQL.

**Exercise:** Write a query that computes, for each product, whether its name contains the word "Wireless" (use `ILIKE '%wireless%'` for case-insensitive matching) as a boolean column called `is_wireless`, using a `CASE WHEN ... THEN ... ELSE ... END` expression.

---

## 1.9 Type Conversion

**Intuition:** SQL is statically typed at the column level but Postgres will often implicitly convert types for you in expressions — and when it won't (or when the implicit conversion does something you didn't intend), you need explicit casting with `::type` or `CAST(value AS type)`.

**Code snippet:**
```sql
SELECT '42'::INTEGER + 8;            -- explicit cast, text to int
SELECT CAST(price AS TEXT) FROM products;  -- the verbose, standard-SQL equivalent of ::
SELECT '2024-01-15'::DATE;
SELECT metadata->>'color' FROM products;   -- ->> extracts a JSONB value AS TEXT
```

**Exercise:** Try `SELECT 'abc'::INTEGER;` and read the error message carefully — Postgres tells you exactly why the cast is invalid. Then fix a realistic scenario: you have a text column storing `'29.99'` and need to compare it numerically to `30`. Write the correct cast to make `'29.99'::NUMERIC < 30` evaluate correctly.

---

## 1.10 JSON/JSONB — the schema-flexible escape hatch inside a relational database

**Intuition:** This is the feature that blurs the SQL/NoSQL line. Sometimes a piece of data is genuinely variable-shaped (e.g., per-product custom attributes that differ wildly between a "book" and a "blender"). Rather than forcing every possible attribute into its own column (or reaching for MongoDB only for this one reason), Postgres lets you store a `JSONB` column and **index into it** almost as if it were structured. `JSON` stores the exact text you gave it; `JSONB` stores a parsed, binary form that's slightly slower to insert but much faster to query — **always prefer `JSONB`** unless you have a specific reason to preserve exact formatting/key order.

**Code snippet:**
```sql
-- Querying inside JSONB
SELECT name FROM products WHERE metadata->>'color' = 'black';
SELECT name FROM products WHERE metadata @> '{"wireless": true}'::jsonb;

-- Indexing JSONB for fast containment queries (see Section 2 for indexes generally)
CREATE INDEX idx_products_metadata ON products USING GIN (metadata);
```

`@>` is the "contains" operator — "does this JSONB document contain this key/value pair (possibly among others)?" It's what the GIN index above is built to accelerate.

---

# PART 2 — INDEXES, FULL TEXT SEARCH & PERFORMANCE (POSTGRESQL)

## 2.1 Why Indexes Exist: The Sequential Scan Problem

**Intuition:** Without an index, finding a row matching `WHERE email = 'x@example.com'` means Postgres checks *every single row* in the table — a **sequential scan**. That's fine for 100 rows, catastrophic for 100 million. An index is a separate, sorted data structure (almost always a **B-tree** by default) that lets Postgres jump nearly straight to the matching rows, the same way a phone book's alphabetical order lets you find "Smith" without reading every name from "Aaronson" onward. The cost: every index speeds up some reads but slows down writes slightly (the index must be updated too) and takes extra disk space. Indexing is always a trade-off, never a free win.

**Code snippet:**
```sql
-- Without an index, this is a sequential scan over the whole table
SELECT * FROM authors WHERE email = 'jane@example.com';

CREATE INDEX idx_authors_email ON authors (email);

-- Now the same query can use an index scan instead
SELECT * FROM authors WHERE email = 'jane@example.com';
```

**Exercise:** Create a table with 100,000 rows of dummy data (use `generate_series` to do this fast: `INSERT INTO authors (name, email) SELECT 'Author ' || i, 'author' || i || '@example.com' FROM generate_series(1, 100000) AS i;`). Run `EXPLAIN ANALYZE SELECT * FROM authors WHERE email = 'author54321@example.com';` *before* creating an index on `email`, note the execution time and "Seq Scan" in the output, then create the index and run it again — note the new "Index Scan" and the time difference.

---

## 2.2 Index Types: B-tree, Hash, GIN, GiST, BRIN

**Intuition:** Not all queries benefit from the same kind of index — that's why Postgres ships several index *types*, each suited to a different shape of question:
- **B-tree** (the default): great for equality and range queries (`=`, `<`, `>`, `BETWEEN`), and sorting.
- **Hash**: equality only, slightly faster than B-tree for pure `=` lookups, but can't do ranges or sorting — rarely the right choice over B-tree in practice.
- **GIN** (Generalized Inverted Index): for when a single column value contains *multiple* searchable things — arrays, `JSONB`, full-text search vectors. "Does this array/JSON/document contain X?"
- **GiST** (Generalized Search Tree): geometric data, ranges, nearest-neighbor — used heavily by PostGIS for geographic queries.
- **BRIN** (Block Range Index): tiny, low-overhead indexes for huge tables where data is naturally ordered on disk (e.g., a `created_at` column on an append-only log table) — trades precision for a tiny fraction of the storage cost of a B-tree.

**Code snippet:**
```sql
CREATE INDEX idx_books_year ON books (published_year);              -- B-tree, default
CREATE INDEX idx_products_tags ON products USING GIN (tags);        -- GIN, array containment
CREATE INDEX idx_products_metadata ON products USING GIN (metadata);-- GIN, JSONB containment
CREATE INDEX idx_orders_created_brin ON orders USING BRIN (created_at); -- BRIN, huge append-only table
```

**Exercise:** Using the `products` table from Section 1.7/1.10, write and run a query that finds all products with the tag `'electronics'` (`WHERE tags @> ARRAY['electronics']` or `WHERE 'electronics' = ANY(tags)`). Run `EXPLAIN ANALYZE` before and after adding the GIN index on `tags`, and confirm the plan changes from a sequential scan to a bitmap/index scan once you have enough rows for Postgres to consider it worthwhile (you may need a few thousand rows before the planner prefers the index — small tables are scanned sequentially on purpose, because it's actually faster than the overhead of using an index).

---

## 2.3 Composite Indexes & Index Selectivity

**Intuition:** An index on `(a, b)` is not the same as two separate indexes on `a` and on `b` — a composite index is sorted first by `a`, then by `b` *within* each `a`. It can efficiently serve queries that filter on `a` alone, or on `a` AND `b` together, but **not** efficiently serve a query that filters on `b` alone (you'd need a separate index, or reverse the column order). Column order in a composite index matters enormously, and you should put the column with the most common "always filtered on" usage first. **Selectivity** is the term for "how much does this filter actually narrow down the rows" — an index on a column where 99% of rows share the same value (like a `status` column with only 3 possible values) isn't very useful on its own, because it doesn't eliminate much.

**Code snippet:**
```sql
-- Composite index: efficiently serves "books by this author, sorted/filtered by year"
CREATE INDEX idx_books_author_year ON books (author_id, published_year);

-- This query benefits fully from the index above
SELECT * FROM books WHERE author_id = 1 AND published_year > 2000;

-- This query benefits PARTIALLY -- it can use the index to find author_id = 1,
-- but can't use it to filter published_year efficiently from a different starting point
SELECT * FROM books WHERE published_year > 2000;
```

**Exercise:** Create the composite index above. Run `EXPLAIN ANALYZE` on both queries shown and compare the plans. Then write a query that filters on `published_year` alone (no `author_id`) and confirm via `EXPLAIN` that it does NOT use `idx_books_author_year` efficiently — this is the column-order lesson made concrete.

---

## 2.4 Reading Query Plans: EXPLAIN and EXPLAIN ANALYZE

**Intuition:** `EXPLAIN` shows you what Postgres's query planner *intends* to do, without running the query. `EXPLAIN ANALYZE` actually *runs* the query and shows you what really happened, including real timings — this is the single most important diagnostic tool for performance work, full stop. Reading a plan is reading it **bottom-up and inside-out**: the innermost/lowest steps run first, and their output feeds the steps above them.

**Code snippet:**
```sql
EXPLAIN ANALYZE
SELECT authors.name, books.title
FROM authors
JOIN books ON books.author_id = authors.id
WHERE authors.email = 'jane@example.com';
```
```text
-- Example output to learn to read:
Nested Loop  (cost=0.29..16.34 rows=1 width=64) (actual time=0.045..0.052 rows=2 loops=1)
  ->  Index Scan using idx_authors_email on authors  (cost=0.29..8.30 rows=1 width=40) (actual time=0.025..0.027 rows=1 loops=1)
        Index Cond: (email = 'jane@example.com'::text)
  ->  Seq Scan on books  (cost=0.00..8.00 rows=2 width=36) (actual time=0.010..0.012 rows=2 loops=1)
        Filter: (author_id = 1)
Planning Time: 0.250 ms
Execution Time: 0.080 ms
```
The things to actually look at: **"Seq Scan" on a large table** (a red flag if you expected an index), the gap between **estimated `rows=` and actual `rows=`** (a big gap means Postgres's statistics are stale — see `ANALYZE` in Section 4), and **Execution Time** as your ground truth.

**Exercise:** Run `EXPLAIN ANALYZE` on the slowest query you've written so far in this curriculum (likely something from Section 1.6's window functions or 1.10's JSONB query on a large table). Identify which step in the plan takes the most time, and write one sentence explaining what you'd do to speed it up (add an index? rewrite the query? both?).

---

## 2.5 Full Text Search

**Intuition:** `LIKE '%word%'` is not search — it can't rank results, can't handle word stems ("running" vs. "run"), and can't use a normal index efficiently. Postgres's full text search converts text into a `tsvector` (a sorted list of normalized word "lexemes," with positions), and turns your search term into a `tsquery`, then matches them — with a GIN index, this becomes genuinely fast at scale, and you can rank results by relevance.

**Code snippet:**
```sql
ALTER TABLE books ADD COLUMN search_vector TSVECTOR
    GENERATED ALWAYS AS (to_tsvector('english', title)) STORED;

CREATE INDEX idx_books_search ON books USING GIN (search_vector);

SELECT title, ts_rank(search_vector, query) AS rank
FROM books, to_tsquery('english', 'dune | herbert') AS query
WHERE search_vector @@ query
ORDER BY rank DESC;
```
`@@` is the full-text "matches" operator. `to_tsvector` stems words (so "running" and "run" both reduce to the same lexeme), strips stopwords ("the," "a"), and `ts_rank` scores how relevant a match is based on frequency and proximity.

**Exercise:** Add the `search_vector` generated column and GIN index to your `books` table. Insert a few books with titles like "Dune", "Children of Dune", "Foundation". Search for `'dune'` and confirm both Dune books rank, with "Dune" itself (an exact, shorter match) typically ranking higher than "Children of Dune". Then try the same search using `title ILIKE '%dune%'` instead and discuss in one sentence why the full text search version is more powerful, even on this tiny example (hint: think about ranking and stemming, not just speed).

---

## 2.6 Performance Tips & Parallel Query

**Intuition:** Beyond indexing, Postgres has a real optimizer (the "planner") that chooses join strategies, decides whether to use parallel workers, and relies on up-to-date table statistics to make good choices. **`ANALYZE`** (and the autovacuum daemon, which runs it automatically) keeps those statistics fresh — stale statistics are a common, invisible cause of bad query plans. For genuinely large scans/aggregations, Postgres can split work across multiple CPU cores automatically (**parallel query**) if the planner judges it worthwhile.

**Code snippet:**
```sql
-- Manually refresh statistics after a big bulk load (autovacuum usually handles this,
-- but right after a massive INSERT it's worth doing explicitly)
ANALYZE books;

-- See current parallel query settings
SHOW max_parallel_workers_per_gather;

-- Force a look at whether a big aggregate query would use parallelism
EXPLAIN SELECT author_id, COUNT(*) FROM books GROUP BY author_id;
```

**Exercise:** On your 100,000-row `authors` table from Section 2.1, run `EXPLAIN SELECT COUNT(*) FROM authors;` and check whether the plan mentions "Workers Planned" (parallel execution). If your local Postgres instance has `max_parallel_workers_per_gather` set to 0 or the table is too small to trigger it, that's fine — note what you observe either way, and explain in one sentence why parallelism only pays off above some data-size threshold (hint: think about the fixed overhead cost of spinning up worker processes).

---

# PART 3 — TRANSACTIONS, CONCURRENCY CONTROL & MVCC

## 3.1 MVCC: How Postgres Achieves Isolation Without Locking Everything

**Intuition:** A naive way to give transactions isolation would be to lock the entire table every time anyone reads or writes — but that would make concurrent reads and writes painfully slow. Postgres instead uses **MVCC (Multi-Version Concurrency Control)**: instead of overwriting a row in place, an `UPDATE` creates a *new version* of the row (with its own internal transaction ID markers) and leaves the old version intact until nothing needs it anymore. Each transaction sees a consistent "snapshot" of the data as of when it started, ignoring versions created by transactions that started later or haven't committed. This is *why* readers never block writers and writers never block readers in Postgres — they're literally looking at different row versions.

**Code snippet:**
```sql
-- Session A:
BEGIN;
SELECT balance FROM accounts WHERE id = 1; -- sees 500

-- Session B (concurrently):
BEGIN;
UPDATE accounts SET balance = 600 WHERE id = 1;
COMMIT;

-- Session A, still in its original transaction:
SELECT balance FROM accounts WHERE id = 1; -- STILL sees 500 under REPEATABLE READ,
                                            -- because it's reading its own snapshot.
                                            -- (Under default READ COMMITTED, it WOULD see 600 -- see 3.2.)
```

**Exercise:** Open two `psql` sessions. In session A, run `BEGIN ISOLATION LEVEL REPEATABLE READ;` then `SELECT balance FROM accounts WHERE id = 1;`. In session B, update and commit a change to that row. Back in session A, run the same `SELECT` again *within the same transaction* and confirm you see the old value. Then `COMMIT` session A and run the `SELECT` once more — now confirm you see the new value, because you've started looking at a fresh snapshot.

---

## 3.2 Transaction Isolation Levels

**Intuition:** SQL defines four isolation levels, each permitting fewer "anomalies" (ways concurrent transactions can surprise you) than the last, at the cost of more blocking/retries:
- **Read Uncommitted**: can see other transactions' *uncommitted* changes ("dirty reads") — Postgres doesn't actually implement this distinctly; it behaves like Read Committed.
- **Read Committed** (Postgres's default): each *statement* sees a fresh snapshot of committed data — but two statements in the same transaction can see different data if something committed in between ("non-repeatable read").
- **Repeatable Read**: the entire transaction sees one consistent snapshot from when it began — no non-repeatable reads, but "serialization anomalies" are still theoretically possible.
- **Serializable**: the strongest level — transactions behave *as if* they ran one at a time, in some order, even though they actually ran concurrently. Postgres achieves this by detecting conflicts and forcing one transaction to retry rather than actually serializing execution.

**Code snippet:**
```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- ... do work that depends on consistent reads across multiple statements ...
COMMIT;
-- If Postgres detects a conflict with another concurrent serializable transaction,
-- COMMIT will fail with a serialization_failure error -- your application code
-- must be ready to catch this and retry the whole transaction.
```

**Exercise:** Write a one-paragraph explanation, in your own words, of why a financial reporting query that needs to read multiple tables and have them all agree as of "one consistent moment in time" should use `REPEATABLE READ` (or `SERIALIZABLE`) rather than the default `READ COMMITTED`. Give a concrete example of what could go wrong under `READ COMMITTED` for that use case.

---

## 3.3 Locking: Row-Level Locks and Deadlocks

**Intuition:** MVCC handles *reads* not blocking writes, but concurrent *writes* to the same row still need explicit coordination — that's what row-level locks are for. `SELECT ... FOR UPDATE` locks the selected rows so no other transaction can modify (or also lock) them until you commit/rollback — essential for "read a value, then update it based on what you read" patterns (like decrementing inventory) that would otherwise race. A **deadlock** happens when transaction A waits for a lock held by B, while B waits for a lock held by A — Postgres detects this automatically and aborts one of the two transactions so the other can proceed.

**Code snippet:**
```sql
BEGIN;
SELECT quantity FROM inventory WHERE product_id = 1 FOR UPDATE; -- locks this row
-- Any other transaction trying to UPDATE or SELECT ... FOR UPDATE this same row
-- will now block until this transaction COMMITs or ROLLBACKs.
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 1;
COMMIT;
```

**Exercise:** Open two `psql` sessions. In session A, `BEGIN; SELECT * FROM inventory WHERE product_id = 1 FOR UPDATE;` (don't commit yet). In session B, try `UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 1;` and observe that it hangs, waiting. Go back to session A and `COMMIT;` — watch session B immediately unblock and complete. This is exactly the mechanism that prevents two simultaneous purchases from both decrementing the same last unit of stock incorrectly.

---

## 3.4 Database Roles, Privileges & Multi-Tenancy Basics

**Intuition:** Beyond application-level permission checks (like DRF's permission classes from the other roadmap), Postgres itself has a role/privilege system that can enforce access at the database level — useful for separating a read-only analytics user from your application's full-access user, or implementing genuine **Row-Level Security (RLS)** so a multi-tenant app can't accidentally leak one tenant's data to another even if application code has a bug.

**Code snippet:**
```sql
CREATE ROLE analytics_readonly LOGIN PASSWORD 'secret';
GRANT CONNECT ON DATABASE mydb TO analytics_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analytics_readonly;

-- Row-Level Security: even with SELECT granted, restrict WHICH rows a role can see
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON orders
    USING (tenant_id = current_setting('app.current_tenant')::INTEGER);
```

---

# PART 4 — DURABILITY, REPLICATION & OPERATIONS (POSTGRESQL)

## 4.1 The Write-Ahead Log (WAL): How Durability Is Actually Implemented

**Intuition:** When you `COMMIT`, Postgres doesn't necessarily write the actual data pages to disk immediately (that would be slow) — instead, it writes a compact record of *the change* to the **Write-Ahead Log** first, and guarantees that record is flushed to disk before telling you the commit succeeded. If the server crashes, on restart Postgres replays the WAL to reconstruct any changes that were committed but hadn't yet been written to the main data files. "Write-ahead" means exactly that: the log entry happens *before* the actual data file is touched. This single mechanism is what makes the "D" in ACID real, and it's also the foundation for replication and point-in-time recovery.

**Code snippet:**
```sql
-- See WAL-related settings
SHOW wal_level;          -- 'replica' or 'logical' needed for replication
SHOW max_wal_size;

-- Force a checkpoint (flushes pending changes from WAL into actual data files) -- admin/debugging tool
CHECKPOINT;
```

**Exercise:** Read through `SHOW wal_level;` and `SHOW max_wal_size;` on your local instance and note the values. Then write, in your own words, the sequence of events from "I run `COMMIT`" to "the change is durable" — specifically, what gets written *first*, and why writing it first (rather than writing the data file first) is what makes crash recovery possible.

---

## 4.2 Backup & Restore

**Intuition:** Backups exist for the failure modes WAL/durability *don't* cover: a human running `DROP TABLE` by mistake, a disk physically dying, or needing to spin up a copy of production data for staging. `pg_dump` produces a logical backup (a script of SQL/data that can recreate the database); physical/continuous backup strategies (base backups + WAL archiving) support **point-in-time recovery** — restoring to the exact moment just before a mistake happened.

**Code snippet:**
```bash
# Logical backup of one database
pg_dump -U postgres -d mydb -F custom -f mydb_backup.dump

# Restore it into a fresh database
createdb mydb_restored
pg_restore -U postgres -d mydb_restored mydb_backup.dump
```

**Exercise:** Run `pg_dump` on your scratch database from this curriculum, then create a new empty database and `pg_restore` into it. Confirm (with `\dt` in `psql`, or a `SELECT COUNT(*)` on a couple of tables) that the restored database matches the original. This is the exact muscle memory you need before you ever touch a production database.

---

## 4.3 Replication & High Availability

**Intuition:** A single Postgres instance is a single point of failure and a single source of read load. **Streaming replication** continuously ships WAL records from a **primary** to one or more **standby/replica** servers, which replay them to stay (nearly) in sync — giving you both a hot failover target and extra read capacity (route read-only queries to replicas). This is conceptually identical to MongoDB's replica sets (Section 7) — both databases solve "don't have just one copy of your data" the same way: ship the write log to followers.

**Code snippet (conceptual `postgresql.conf` / `pg_hba.conf` settings, not a single runnable script):**
```ini
# On the primary, postgresql.conf:
wal_level = replica
max_wal_senders = 5

# On the standby, after a base backup, postgresql.conf / standby.signal:
primary_conninfo = 'host=primary-host port=5432 user=replicator'
```
```sql
-- Check replication status from the primary
SELECT client_addr, state, sent_lsn, replay_lsn FROM pg_stat_replication;
```

**Exercise:** (Conceptual, no infra required unless you want to go further with Docker.) Explain in your own words the difference between **synchronous** and **asynchronous** replication, and which one a banking application's primary-to-standby setup should prefer, given everything you now know about ACID and the durability guarantee — specifically, what happens to the "D" in ACID if the primary commits and confirms to the client *before* the standby has the change, and the primary then dies.

---

## 4.4 Routine Maintenance: VACUUM and Autovacuum

**Intuition:** Because of MVCC (Section 3.1), old row versions ("dead tuples") aren't immediately removed when superseded — they need to be cleaned up eventually, or the table bloats and performance degrades. `VACUUM` reclaims that space; `VACUUM ANALYZE` also refreshes planner statistics. Postgres runs **autovacuum** automatically in the background, but understanding what it's doing (and when to intervene manually, e.g. after a massive bulk delete) is a real operational skill.

**Code snippet:**
```sql
VACUUM ANALYZE books;
SELECT relname, n_dead_tup, n_live_tup FROM pg_stat_user_tables WHERE relname = 'books';
```

---

# PART 5 — NOSQL & MONGODB: THE DOCUMENT MODEL

> This part follows MongoDB's own manual structure: CRUD Operations → Data Modeling → Aggregation → Indexes → Transactions → Change Streams → Replication → Sharding.

## 5.1 Documents, Collections, and BSON

**Intuition:** Where Postgres stores rigid rows in tables, MongoDB stores **documents** (JSON-like structures, actually encoded as **BSON** — Binary JSON — which adds extra types like native dates and binary data that plain JSON can't represent) in **collections**. The defining difference: documents in the same collection *don't have to share the same shape*. One product document might have a `dimensions` field; another might not. This isn't sloppiness — it's a deliberate trade: you give up the database enforcing a rigid shape, in exchange for being able to model real-world data that's naturally nested and variable, without needing a join to assemble it.

**Code snippet:**
```javascript
// Inserting a document -- notice the nested object and array, native to the model
db.products.insertOne({
  name: "Wireless Mouse",
  price: 29.99,
  tags: ["electronics", "accessories"],
  dimensions: { length: 10.5, width: 6.2, height: 3.8 },
  reviews: [
    { user: "alice", rating: 5, comment: "Great mouse!" },
    { user: "bob", rating: 4, comment: "Works well." }
  ]
});
```
Compare this to the relational version: in Postgres, `reviews` would almost certainly be a *separate table* with a foreign key back to `products`, requiring a `JOIN` to reassemble. In MongoDB, the review is already embedded — reading the product gives you everything in one document, one read, no join.

**Exercise:** Insert three product documents with deliberately different shapes — one with a `dimensions` object, one without it but with a `weight_kg` field instead, one with neither but an extra `discontinued: true` field. Run `db.products.find()` and observe that MongoDB has no complaints about the differing shapes — then try inserting the equivalent as rows in a Postgres table without a `JSONB` column, and explain in one sentence why you'd be forced to either add nullable columns for every possible field or reach for `JSONB`.

---

## 5.2 CRUD Operations

**Intuition:** MongoDB's CRUD vocabulary mirrors SQL's DML conceptually but uses driver methods and query documents instead of a query *language* string. The biggest mental shift: filters, updates, and even some inserts are themselves expressed as **documents** — MongoDB is, in a sense, "documents all the way down."

**Code snippet:**
```javascript
// Create
db.products.insertOne({ name: "Keyboard", price: 49.99, tags: ["electronics"] });
db.products.insertMany([
  { name: "Monitor", price: 199.99, tags: ["electronics", "displays"] },
  { name: "Desk Mat", price: 15.00, tags: ["accessories"] }
]);

// Read
db.products.find({ price: { $lt: 50 } });               // price less than 50
db.products.find({ tags: "electronics" });               // array contains "electronics"
db.products.findOne({ name: "Keyboard" });

// Update
db.products.updateOne(
  { name: "Keyboard" },
  { $set: { price: 44.99 }, $push: { tags: "sale" } }
);

// Delete
db.products.deleteOne({ name: "Desk Mat" });
```
`$set` and `$push` are **update operators** — MongoDB updates are expressed as a transformation document, not a SQL-style `SET column = value` clause, but the intent is the same.

**Exercise:** Using the `products` collection, write a query that finds every product priced between `$20` and `$100` using `$gte`/`$lte`, and a separate update that adds a `"featured": true` field to every product tagged `"electronics"`, using `$set` combined with a filter on the `tags` array. Confirm both with `find()`.

---

## 5.3 Query Operators in Depth

**Intuition:** MongoDB's query language is operator-rich — `$lt`/`$lte`/`$gt`/`$gte`/`$ne`/`$in`/`$nin` for comparisons, `$and`/`$or`/`$not` for logic, `$exists`/`$type` for shape-checking (genuinely useful given the flexible schema), and `$regex` for pattern matching. The intuition: a query filter document is a small declarative tree describing what should be true about a matching document.

**Code snippet:**
```javascript
// Logical combination
db.products.find({
  $or: [
    { price: { $lt: 20 } },
    { tags: { $in: ["sale"] } }
  ]
});

// Existence + type checking -- genuinely useful with a flexible schema
db.products.find({ dimensions: { $exists: true } });

// Nested field query using dot notation
db.products.find({ "dimensions.length": { $gt: 10 } });

// Array element matching
db.products.find({ reviews: { $elemMatch: { rating: { $gte: 5 } } } });
```
`$elemMatch` deserves special attention: without it, `{ "reviews.rating": { $gte: 5 } }` would match a document if *any* review has `rating >= 5` — but if you need *the same array element* to satisfy multiple conditions at once (e.g., `rating >= 5 AND user == "alice"`), you need `$elemMatch` to keep those conditions scoped to one element.

**Exercise:** Add a product with two reviews: one with `{ user: "carol", rating: 2 }` and one with `{ user: "dave", rating: 5 }`. Write a query using `$elemMatch` to find products with a review that has `rating: 5` AND `user: "dave"` specifically — confirm it correctly excludes a hypothetical product that has a 5-star review from someone *other* than Dave and a low review from Dave (i.e., the high rating and the specific user must be on the *same* review object).

---

## 5.4 Data Modeling: Embedding vs. Referencing

**Intuition:** This is the single most consequential decision in MongoDB schema design, and it has no exact equivalent in relational design because relational tables don't give you the *option* to embed — you always reference (via foreign key). In MongoDB you choose, per relationship:
- **Embed** when the related data is mostly accessed *together*, doesn't grow unboundedly, and doesn't need to be queried independently very often (e.g., a blog post's comments, an order's line items at the time of purchase).
- **Reference** (store an `_id`, look it up separately — analogous to a foreign key) when the related data is large, grows unboundedly, is shared across many parent documents, or needs independent querying (e.g., a product referenced by thousands of orders — you don't want to duplicate the entire product document into every order).

The intuition to really hold onto: **MongoDB schema design starts from "how will this data be read," not "how do I normalize this to avoid duplication."** That's the opposite instinct from relational design, and it's the most common mistake people coming from a SQL background make — they normalize everything into separate collections and end up doing the equivalent of manual application-side joins for every read, losing the actual benefit of the document model.

**Code snippet:**
```javascript
// EMBEDDING -- order line items rarely need independent querying,
// and you WANT the price/name frozen at time of purchase (not affected by future product edits)
db.orders.insertOne({
  customer_email: "jane@example.com",
  items: [
    { product_name: "Wireless Mouse", price_at_purchase: 29.99, quantity: 2 },
    { product_name: "Keyboard", price_at_purchase: 44.99, quantity: 1 }
  ],
  total: 104.97
});

// REFERENCING -- a product is shared across many orders and changes independently over time
db.products.insertOne({ _id: ObjectId("..."), name: "Wireless Mouse", price: 29.99 });
db.orders.insertOne({
  customer_email: "jane@example.com",
  product_ids: [ObjectId("...")],  // reference, not a copy
  total: 29.99
});
```

**Exercise:** Design (on paper, then as actual `insertOne` calls) a `blog_posts` collection where each post has comments. Decide: should comments be embedded or referenced? Write your reasoning in one paragraph (consider: how many comments can a popular post realistically have, and would that ever threaten MongoDB's 16MB-per-document limit — a real, hard constraint that should factor into embedding decisions for high-growth arrays), then implement your chosen design.

---

## 5.5 The $lookup Stage — MongoDB's "Join"

**Intuition:** When you *do* need to combine referenced collections in a single query (rather than two separate round trips from your application), the aggregation framework's `$lookup` stage performs a left-outer-join-like operation. It's less commonly needed than `JOIN` is in SQL — precisely because good MongoDB schema design embeds what's usually read together — but it exists for the cases where you genuinely need it.

**Code snippet:**
```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "product_ids",
      foreignField: "_id",
      as: "product_details"
    }
  }
]);
```

---

# PART 6 — MONGODB: AGGREGATION, INDEXES, TRANSACTIONS & SCALE

## 6.1 The Aggregation Pipeline

**Intuition:** If `find()` is MongoDB's `SELECT ... WHERE`, the **aggregation pipeline** is its `GROUP BY`/window-function/everything-else toolkit — a sequence of **stages**, each transforming the documents flowing through it, conceptually identical to piping commands together in a shell (`cmd1 | cmd2 | cmd3`). Each stage does one job: `$match` filters (like `WHERE`), `$group` aggregates (like `GROUP BY`), `$project` reshapes output (like choosing columns), `$sort` orders, `$limit` caps results. The pipeline mental model is the single biggest unlock for getting comfortable with MongoDB beyond basic CRUD.

**Code snippet:**
```javascript
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: {
      _id: "$customer_email",
      totalSpent: { $sum: "$total" },
      orderCount: { $sum: 1 }
  }},
  { $sort: { totalSpent: -1 } },
  { $limit: 10 }
]);
```
Read this top to bottom: keep only completed orders → group remaining documents by customer email, summing their totals and counting orders → sort by total spent descending → keep the top 10. This is the direct MongoDB analog of the SQL:
```sql
SELECT customer_email, SUM(total) AS totalSpent, COUNT(*) AS orderCount
FROM orders WHERE status = 'completed'
GROUP BY customer_email
ORDER BY totalSpent DESC
LIMIT 10;
```

**Exercise:** Insert 15-20 order documents across 4-5 different `customer_email` values, with varying `total` and `status` fields (`"completed"` and `"pending"`). Run the aggregation above and confirm it correctly identifies your top spenders, excluding pending orders. Then add a `$match` stage at the very end that keeps only customers with `orderCount` greater than 2 — note that you need `$match` again (not the original filter) because you're now filtering on a field that only exists *after* `$group` ran.

---

## 6.2 Indexes in MongoDB

**Intuition:** The exact same sequential-scan problem from Section 2.1 exists in MongoDB — without an index, `find()` checks every document (a "collection scan"). MongoDB's default index type is also a B-tree-like structure, and the same intuitions about selectivity and composite-index column order from Sections 2.1–2.3 transfer directly. The main MongoDB-specific addition: indexes can target fields *inside* embedded documents and arrays (a "multikey index"), which has no direct Postgres equivalent outside of `JSONB`/array GIN indexes.

**Code snippet:**
```javascript
db.products.createIndex({ name: 1 });               // ascending single-field index
db.products.createIndex({ tags: 1 });                // multikey index -- indexes each array element
db.orders.createIndex({ customer_email: 1, status: 1 }); // composite index

// See whether a query used the index
db.orders.find({ customer_email: "jane@example.com", status: "completed" }).explain("executionStats");
```

**Exercise:** Insert a few thousand order documents (a small loop in `mongosh` works fine: `for (let i = 0; i < 5000; i++) { db.orders.insertOne({ customer_email: "user"+i+"@example.com", status: i % 3 === 0 ? "completed" : "pending", total: Math.random()*200 }); }`). Run `.explain("executionStats")` on a query filtering by `customer_email` before and after creating an index on it, and compare `totalDocsExamined` — this is MongoDB's equivalent of comparing "Seq Scan" vs. "Index Scan" row counts in Postgres's `EXPLAIN ANALYZE`.

---

## 6.3 Transactions in MongoDB

**Intuition:** It's a common misconception that "MongoDB doesn't have transactions" — it does, and has since version 4.0, supporting full multi-document ACID transactions across a replica set or sharded cluster. The deeper, more important intuition from MongoDB's own documentation: **multi-document transactions are an escape hatch, not your primary tool for data integrity.** Because all writes to a *single* document are already atomic by default (a write to one document, however nested, either fully succeeds or fully fails — no partial writes), good embedding-based schema design (Section 5.4) often eliminates the *need* for a multi-document transaction in the first place. Reach for one when you genuinely can't avoid touching multiple documents (or collections) atomically — e.g., moving funds between two account documents.

**Code snippet:**
```javascript
const session = db.getMongo().startSession();
session.startTransaction();
try {
  const accounts = session.getDatabase("mydb").accounts;
  accounts.updateOne({ _id: "A" }, { $inc: { balance: -100 } });
  accounts.updateOne({ _id: "B" }, { $inc: { balance: 100 } });
  session.commitTransaction();
} catch (error) {
  session.abortTransaction();
  throw error;
}
```

**Exercise:** Create an `accounts` collection with two documents, `{_id: "A", balance: 500}` and `{_id: "B", balance: 500}`. Run the transaction snippet above (requires a replica set, even a single-node one, to support transactions — `mongod --replSet rs0` plus `rs.initiate()` if you're running locally). Confirm both balances updated together. Then deliberately throw an error inside the `try` block before `commitTransaction()` and confirm via `abortTransaction()` that *neither* balance changed — this is atomicity, the same guarantee from Section 0.2, now demonstrated in a document database.

---

## 6.4 Change Streams

**Intuition:** Change streams let your application **subscribe to real-time changes** happening in a collection — inserts, updates, deletes — without polling. This is MongoDB's building block for reactive features (live dashboards, cache invalidation triggers, syncing to a search index) and has no precise built-in equivalent in vanilla Postgres (Postgres achieves similar things via `LISTEN`/`NOTIFY` or logical replication slots, conceptually parallel but mechanically different).

**Code snippet:**
```javascript
const changeStream = db.orders.watch();
changeStream.forEach(change => {
  print(`Operation: ${change.operationType}, Document: ${JSON.stringify(change.fullDocument)}`);
});
// In another session: db.orders.insertOne({ customer_email: "x@example.com", total: 50 });
// The watching session prints the change in real time.
```

**Exercise:** (Conceptual if you don't have two terminals handy.) Explain, in one paragraph, how you'd use a change stream to keep a Redis cache (Part 7 of this curriculum) automatically invalidated whenever an order document changes — without your application code having to remember to call a cache-invalidation function at every single place that writes to `orders`.

---

## 6.5 Replication: Replica Sets

**Intuition:** A MongoDB **replica set** is a group of `mongod` instances maintaining the same data set: one **primary** accepts all writes, and **secondaries** replicate from it asynchronously by default — this is the BASE/eventual-consistency mechanism from Section 0.3 made concrete. If the primary becomes unavailable, the replica set holds an automatic election among eligible secondaries to choose a new primary. This is the direct conceptual sibling of Postgres streaming replication (Section 4.3) — same problem (don't have one copy of your data, survive a node failure), different implementation details.

**Code snippet:**
```javascript
// Initializing a replica set (run once, on what will become the primary)
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "localhost:27017" },
    { _id: 1, host: "localhost:27018" },
    { _id: 2, host: "localhost:27019" }
  ]
});

rs.status();  // see current primary/secondary roles and health
```

**Exercise:** Revisit your answer from Section 0.2/0.3's exercises about your three example systems. Now that you understand both Postgres streaming replication and MongoDB replica sets, write one sentence on what "write concern" (`{ w: "majority" }` in MongoDB, or synchronous replication in Postgres) trades away, and why a banking system might insist on a write being acknowledged by a majority of replicas before confirming success to the user — even though that makes writes slower.

---

## 6.6 Sharding: Horizontal Scaling

**Intuition:** Replication solves "survive a failure" and "spread out reads," but it doesn't solve "this dataset is too big/too write-heavy for one machine." **Sharding** splits a collection's data across multiple machines (**shards**) based on a **shard key**, so different ranges/hashes of data live on different servers entirely — true horizontal scaling. Choosing a good shard key is one of the highest-stakes decisions in a large MongoDB deployment: a bad shard key (e.g., a monotonically increasing field, causing all new writes to pile onto one shard) creates a "hot shard" that defeats the entire purpose.

**Code snippet:**
```javascript
sh.enableSharding("mydb");
sh.shardCollection("mydb.orders", { customer_email: "hashed" }); // hashed shard key spreads writes evenly
```

---

# PART 7 — CHOOSING BETWEEN SQL AND NOSQL (AND USING BOTH)

## 7.1 A Decision Framework, Not a Holy War

**Intuition:** By now you've built the same kinds of structures (entities, relationships, search, aggregation) in both Postgres and MongoDB, so this section is less "new facts" and more "tying together the trade-offs you've already felt with your hands." A practical decision framework:

- Choose **relational/Postgres** when: your data has clear, stable relationships that you need to query flexibly and in ad-hoc ways; you need strong consistency and multi-row/multi-table transactions as your *default*, not an escape hatch; your schema is unlikely to vary wildly between records of the same "kind."
- Choose **document/MongoDB** when: your data is naturally nested and is usually read/written as a whole unit; your schema legitimately varies between records (different product types with different attributes); you need to scale write throughput horizontally more than you need ad-hoc relational queries; you're iterating on the shape of your data quickly during early development.
- Choose **both** (polyglot persistence) when: different parts of your system have genuinely different access patterns — e.g., user accounts, billing, and orders in Postgres (strong consistency, relational integrity matters a lot), while a product catalog with wildly varying attributes, or an activity feed/event log, lives in MongoDB.

**Exercise:** Take the "Forge" capstone project domain model from `forge-capstone-project.md` (Organization → Project → Sprint → Task → Comment/ActivityLog). Argue, in a short paragraph, whether this domain is better suited to Postgres (as the project actually specifies) or whether parts of it — specifically the `ActivityLog` — might be a legitimate candidate for a document store instead, given how it's append-heavy, schema-variable (different activity types have different relevant fields), and rarely needs complex joins. There's no single "correct" answer — the point is to practice making the trade-off explicitly rather than by default.

---

# PART 8 — CACHING STRATEGIES WITH REDIS

## 8.1 What Redis Is, and Why It's Not "Just a Faster Database"

**Intuition:** Redis is an **in-memory data structure store** — by keeping data in RAM instead of on disk, it serves reads and writes in sub-millisecond time, often 50-100x faster than even a well-indexed disk-backed query. The crucial mental model: Redis is usually **not your system of record** — it's a fast, often-volatile layer sitting *in front of* a durable database (Postgres/MongoDB), trading a small risk of data loss/staleness for enormous speed. Everything in this Part is really about managing that trade-off deliberately, rather than accidentally.

**Code snippet:**
```python
import redis
r = redis.Redis(host="localhost", port=6379, decode_responses=True)

r.set("greeting", "hello")
print(r.get("greeting"))  # "hello"

r.set("session_token_abc", "user_id:42", ex=3600)  # expires in 1 hour
print(r.ttl("session_token_abc"))  # seconds remaining
```

**Exercise:** Install Redis locally (or via Docker: `docker run -d -p 6379:6379 redis`). Run the snippet above. Then run `r.get("session_token_abc")` again after manually expiring it early with `r.expire("session_token_abc", 1)` and waiting 2 seconds — confirm it returns `None`. This is your first direct observation of TTL-based expiration, the mechanism nearly every caching strategy below depends on.

---

## 8.2 Redis Data Structures (Beyond Simple Key-Value)

**Intuition:** Redis's real power isn't "fast key-value storage" alone — it's that the *values* themselves can be rich structures, each with operations tailored to a specific access pattern, all still served in sub-millisecond time. Picking the right structure for the job (instead of always reaching for a plain string holding serialized JSON) is what separates basic Redis usage from using it well.

**Code snippet:**
```python
# Hash -- store an object's fields without serializing/deserializing the whole thing
r.hset("user:42", mapping={"name": "Jane", "email": "jane@example.com", "age": "29"})
r.hget("user:42", "email")          # fetch just one field
r.hgetall("user:42")                # fetch the whole object

# List -- ordered, good for queues/recent-activity feeds
r.lpush("recent_orders", "order:101")
r.lrange("recent_orders", 0, 9)     # latest 10

# Set -- unique unordered membership, fast set operations
r.sadd("user:42:roles", "admin", "editor")
r.sismember("user:42:roles", "admin")  # True

# Sorted Set (ZSET) -- the classic leaderboard structure, ordered by score
r.zadd("leaderboard", {"alice": 1500, "bob": 1800})
r.zrevrange("leaderboard", 0, 2, withscores=True)  # top 3, descending
```

**Exercise:** Build a tiny leaderboard: add 5 players with random scores using `zadd`, then write the query to get the top 3 (`zrevrange`) and separately, a player's *rank* among all players (`zrevrank`). Then build a simple recent-activity feed using a `list`: push 5 events with `lpush`, then trim it to only keep the most recent 3 using `ltrim` — this combination (`lpush` + `ltrim`) is the standard Redis pattern for a bounded recent-activity feed, since it makes the list self-limiting without a separate cleanup job.

---

## 8.3 Cache-Aside (Lazy Loading) — The Default Pattern

**Intuition:** Cache-aside is the pattern you should reach for *by default*, unless you have a specific reason not to. The application owns the logic: on a read, check the cache first; on a miss, read from the database, then populate the cache for next time. On a write, **invalidate** (delete) the cache key rather than try to update it in place — deleting avoids a subtle race condition where a slow-finishing read could overwrite a cache entry with stale data *after* a concurrent write already invalidated it.

**Code snippet:**
```python
import json

def get_user(user_id):
    cache_key = f"user:{user_id}"
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)           # cache hit

    user = db_query_user(user_id)           # cache miss -- fall back to the real DB
    if user:
        r.set(cache_key, json.dumps(user), ex=300)  # repopulate, with a TTL safety net
    return user

def update_user(user_id, data):
    db_update_user(user_id, data)           # write to the source of truth first
    r.delete(f"user:{user_id}")             # then invalidate -- never overwrite in place
```

**Exercise:** Implement both functions against a mock `db_query_user`/`db_update_user` (plain Python dicts standing in for a real DB are fine). Call `get_user` twice in a row and add a `print` inside the "cache miss" branch to prove the second call never executes it (cache hit). Then call `update_user`, and call `get_user` again — confirm it misses the cache (because you deleted the key) and re-populates from the "DB" with the new value.

---

## 8.4 Read-Through, Write-Through, and Write-Behind

**Intuition:** Cache-aside puts the cache-management logic in your application. The alternative patterns push that logic *into a caching layer* the application talks to instead of the database directly:
- **Read-through**: the cache itself knows how to fetch from the DB on a miss — your application just always asks the cache, never touches the DB directly. Simpler call sites, but couples you to a cache-aware data-access layer.
- **Write-through**: every write goes to the cache *and* the database, synchronously, before the write is considered complete. Strong consistency between cache and DB, at the cost of every write now paying the latency of both systems.
- **Write-behind (write-back)**: writes go to the cache immediately (fast), and the cache asynchronously flushes to the database later, often batched. Excellent write latency and throughput; the real risk is data loss if the cache crashes before flushing — this is the trade-off to interrogate hardest before choosing it.

**Code snippet (write-through, the easiest to reason about explicitly):**
```python
def update_user_write_through(user_id, data):
    db_update_user(user_id, data)                              # write to DB
    r.set(f"user:{user_id}", json.dumps(data), ex=300)         # AND write to cache, same request
    # Contrast with cache-aside's update_user(), which deletes instead of setting --
    # write-through accepts the small race-condition risk in exchange for the cache
    # never going cold on writes.
```

**Exercise:** Write a one-paragraph comparison: given a high-write-volume IoT sensor-data ingestion system where losing a few seconds of data on a crash is acceptable but write latency must be minimized, which of the three patterns (read-through, write-through, write-behind) fits best, and why would the other two be actively wrong choices for this specific workload?

---

## 8.5 TTL Strategy and Cache Invalidation

**Intuition:** A cache entry without a TTL is a ticking time bomb — if you ever forget to invalidate it on every single code path that changes the underlying data, it serves stale data forever. **Always set a TTL**, even on data you actively invalidate elsewhere, as a safety net against the invalidation logic having a bug or a code path you forgot. Different data deserves different TTLs based on how often it changes and how costly staleness is.

**Code snippet:**
```python
# Short TTL for fast-changing, low-cost-if-stale data
r.set("trending:posts", json.dumps(trending), ex=60)        # 1 minute

# Medium TTL for semi-static data
r.set(f"user:{user_id}:profile", json.dumps(profile), ex=3600)  # 1 hour

# Jitter to prevent many keys expiring at the exact same instant (a mini stampede -- see 8.6)
import random
jitter = random.randint(0, 30)
r.set(f"product:{product_id}", json.dumps(product), ex=600 + jitter)
```

**Exercise:** Cache 20 "product" keys all with exactly `ex=10` (10 seconds, no jitter). Wait 10 seconds and observe (via `r.keys("product:*")` or similar) that they all vanish at once — and explain in one sentence why, if all 20 of those products are frequently read, this exact-same-expiry pattern could cause a sudden spike of simultaneous database queries the moment they all expire together. Then redo it with jitter (`ex=10 + random.randint(0, 5)`) and observe the expirations spread out instead.

---

## 8.6 Eviction Policies & Memory Management

**Intuition:** TTL handles *planned* expiration, but Redis also needs a policy for when it's simply out of memory and a new write needs room. **Eviction policies** decide which keys to sacrifice. `noeviction` refuses new writes once full (treats Redis more like a strict store than a cache). `allkeys-lru` evicts the Least Recently Used key, regardless of TTL — the standard choice for a pure cache. `allkeys-lfu` evicts the Least Frequently Used key — better when you have a skewed "hot set" of frequently-reused keys you want to protect even if they weren't accessed in the last few seconds. `volatile-*` variants only consider keys that have a TTL set, leaving keys without a TTL untouched (useful if you're mixing cache data with some non-expiring keys in the same instance, though running separate instances for separate purposes is usually cleaner).

**Code snippet:**
```bash
# redis.conf, or via CONFIG SET at runtime
CONFIG SET maxmemory 256mb
CONFIG SET maxmemory-policy allkeys-lru
```
```python
info = r.info("stats")
print(info["evicted_keys"], info["keyspace_hits"], info["keyspace_misses"])
```

**Exercise:** Set `maxmemory` to a deliberately tiny value (e.g., `10mb`) on a local/test Redis instance, set `maxmemory-policy` to `allkeys-lru`, then write a loop inserting a few thousand sizable string values until you start seeing `evicted_keys` increase in `INFO stats`. This is your direct, hands-on confirmation that eviction is actually happening, not just a setting you trust blindly.

---

## 8.7 Cache Stampede (Thundering Herd) & Mitigation

**Intuition:** A **cache stampede** happens when a popular cache key expires (or is invalidated) and a flood of concurrent requests *all* miss the cache at the same instant and *all* hammer the database simultaneously trying to repopulate it — potentially overloading the database far worse than if the cache didn't exist at all for that moment. Two common mitigations: a **lock** (the first request to miss acquires a short-lived lock and repopulates the cache while other requests either wait briefly or serve stale data; once the lock holder finishes, everyone benefits), and **probabilistic early expiration** (recompute slightly *before* the actual TTL expires, with randomized timing, so requests don't all converge on the exact same expiry instant).

**Code snippet (mutex-based stampede protection):**
```python
import time

def get_user_safe(user_id):
    cache_key = f"user:{user_id}"
    lock_key = f"lock:{cache_key}"

    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)

    # Try to acquire a short-lived lock; only one concurrent requester wins it
    if r.set(lock_key, "1", nx=True, ex=5):
        try:
            user = db_query_user(user_id)
            r.set(cache_key, json.dumps(user), ex=300)
            return user
        finally:
            r.delete(lock_key)
    else:
        # Someone else is already repopulating -- brief wait and retry,
        # rather than also hitting the database
        time.sleep(0.05)
        return get_user_safe(user_id)
```
`nx=True` ("set if Not eXists") combined with `ex=5` is what makes lock acquisition atomic and self-healing — even if the lock holder crashes before releasing it, the lock expires on its own after 5 seconds.

**Exercise:** Simulate a stampede without real concurrency: write a loop that calls the *unsafe* `get_user` from Section 8.3 ten times in a row right after invalidating the cache, with a `print` inside `db_query_user` to count real DB hits — you'll see 10 (well, conceptually — true concurrency needs threads/async to actually race; reason through why 10 *simultaneous* requests would all hit this branch in the unsafe version). Then trace through the `get_user_safe` logic by hand and explain which single request would actually reach `db_query_user` if 10 requests arrived at the exact same microsecond.

---

## 8.8 Caching Beyond Key-Value: Query Result Caching & Session Storage

**Intuition:** Two of the most common production uses of Redis deserve calling out explicitly because they generalize the patterns above into concrete, everyday infrastructure:
- **Session storage**: instead of server-side session data living in your application server's memory (which breaks the moment you run more than one server instance, since a user's session might hit a different server on their next request), store sessions in Redis — any server instance can read/write the same session, with a TTL that naturally expires inactive sessions.
- **Query result caching**: for an expensive aggregate query (the kind from Section 1.6's window functions, or Section 6.1's aggregation pipelines) that doesn't need to be perfectly real-time, cache the *result* of the query itself, keyed by its parameters, rather than re-running it on every request.

**Code snippet:**
```python
# Session storage
def create_session(user_id):
    session_id = generate_random_token()
    r.hset(f"session:{session_id}", mapping={"user_id": user_id, "created_at": time.time()})
    r.expire(f"session:{session_id}", 1800)  # 30-minute sliding-ish session
    return session_id

# Query result caching, keyed by the query's actual parameters
def get_top_spenders_cached(limit=10):
    cache_key = f"top_spenders:{limit}"
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)
    result = run_expensive_aggregation_query(limit)  # the Postgres/Mongo query from Parts 1-6
    r.set(cache_key, json.dumps(result), ex=120)
    return result
```

**Exercise:** Tie this back to the very first roadmap in this series: in Next.js (`react-nextjs-roadmap.md`, Section 17), `revalidateTag`/`cacheTag` manage *framework-level* caching of rendered output. Write a short paragraph explaining where Redis-based caching like this would sit *relative to* that Next.js caching layer in a full request — i.e., draw the path a request takes from the browser through Next.js, to Django/DRF, to Redis, to Postgres, and identify which layer would actually get hit first on a warm cache, and which layers are skipped entirely when it is.

---

## 8.9 Redis as More Than a Cache: Pub/Sub, Persistence, and Cluster Mode

**Intuition:** Three closing concepts so you understand the full shape of Redis rather than just its caching use, in case future projects need them:
- **Pub/Sub**: Redis can act as a lightweight real-time message broker — publishers send messages to a channel, subscribers receive them instantly. Useful for things like broadcasting a "cache invalidated" event across multiple application server instances.
- **Persistence (RDB & AOF)**: even though Redis is "in-memory," it can persist to disk — RDB takes periodic point-in-time snapshots (fast restarts, but can lose the last few minutes of data on a crash); AOF logs every write operation (closer to a WAL, more durable, slightly slower). You choose this trade-off explicitly based on whether Redis in your system is purely disposable cache (skip persistence entirely) or holds anything you'd be upset to lose (enable AOF).
- **Cluster mode**: like MongoDB sharding (Section 6.6), Redis Cluster partitions data (via hash slots) across multiple nodes for horizontal scale, once a single instance's memory or throughput isn't enough.

**Code snippet:**
```python
# Pub/Sub
pubsub = r.pubsub()
pubsub.subscribe("cache_invalidations")
for message in pubsub.listen():
    if message["type"] == "message":
        print(f"Invalidate: {message['data']}")

# In another process:
r.publish("cache_invalidations", "user:42")
```

**Exercise:** Write a one-paragraph design note (no code required) for the following scenario: you have 3 instances of a Django app, each with its own in-process memory cache *in addition to* shared Redis. A `PUT /users/42` request hits instance A and updates Redis. Explain how Pub/Sub could be used so that instances B and C also invalidate their *local* in-process caches for user 42, instead of serving stale local-cache data until their own TTL expires.

---

# CLOSING: HOW THIS FILE FITS THE OTHER TWO ROADMAPS

You now have three documents that cover one continuous, full-stack system end to end:

1. **`react-nextjs-roadmap.md`** — the client, including how it fetches and caches data (`fetch`, Cache Components, `revalidateTag`)
2. **`python-django-postgres-roadmap.md`** — the application/API layer, including the Django ORM as a translation layer over the relational concepts in this file's Part 1-4
3. **`databases-sql-nosql-caching.md`** (this file) — the actual storage and caching engines underneath everything: what ACID/BASE really guarantee, how Postgres and MongoDB structurally differ and why you'd choose each, and how Redis fits in front of both as the speed layer

The **`forge-capstone-project.md`** project is still the right target to apply all three against — and Section 7.1's exercise above is a direct invitation to reconsider one piece of its schema (the `ActivityLog`) through a polyglot-persistence lens now that you've built real intuition for both models with your own hands, not just read about them.





