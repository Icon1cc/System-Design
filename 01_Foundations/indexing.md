# Database Indexing Basics

## The Simple Explanation

An index is like a book's table of contents. Instead of reading every page to find what you need, you look up the topic and jump directly to the right page.

```
┌─────────────────────────────────────────────────────────────────┐
│                  WITHOUT INDEX                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Query: SELECT * FROM users WHERE email = 'alice@mail.com'     │
│                                                                 │
│   Database must scan EVERY row:                                 │
│                                                                 │
│   ┌────┬─────────────────────────────┐                         │
│   │ ID │ Email                       │ Check?                  │
│   ├────┼─────────────────────────────┤                         │
│   │ 1  │ bob@mail.com                │ ✗                       │
│   │ 2  │ carol@mail.com              │ ✗                       │
│   │ 3  │ david@mail.com              │ ✗                       │
│   │ .. │ ...                         │ ✗                       │
│   │ 999│ alice@mail.com              │ ✓ Found!                │
│   │ .. │ ...                         │ (must keep checking!)   │
│   └────┴─────────────────────────────┘                         │
│                                                                 │
│   Time: O(n) - must check all n rows                            │
│   With 1 million rows: ~1 million comparisons                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   WITH INDEX                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Index on email column (sorted/structured):                    │
│                                                                 │
│   ┌─────────────────────────────┬──────────────┐               │
│   │ Email (Index)               │ Row Location │               │
│   ├─────────────────────────────┼──────────────┤               │
│   │ alice@mail.com              │ Row 999      │ ◄── Direct!   │
│   │ bob@mail.com                │ Row 1        │               │
│   │ carol@mail.com              │ Row 2        │               │
│   │ david@mail.com              │ Row 3        │               │
│   └─────────────────────────────┴──────────────┘               │
│                                                                 │
│   Query: Look up 'alice@mail.com' in index                      │
│          → Index points to Row 999                              │
│          → Fetch Row 999 directly                               │
│                                                                 │
│   Time: O(log n) - binary search in sorted index                │
│   With 1 million rows: ~20 comparisons!                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## How Indexes Work

### B-Tree Index (Most Common)

B-Trees are self-balancing tree data structures that keep data sorted and allow searches, insertions, and deletions in logarithmic time.

```
┌─────────────────────────────────────────────────────────────────┐
│                    B-TREE INDEX                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Searching for email = 'mike@mail.com'                         │
│                                                                 │
│                         ┌───────────────────┐                   │
│                         │  [dave] [mary]    │    ◄── Root       │
│                         └────┬───────┬──────┘                   │
│                    < dave    │       │  > mary                  │
│              ┌───────────────┘       └───────────────┐          │
│              │                                       │          │
│              ▼                                       ▼          │
│   ┌───────────────────┐                 ┌───────────────────┐  │
│   │ [alice] [carol]   │                 │ [mike] [steve]    │  │
│   └──┬─────────┬──────┘                 └──┬─────────┬──────┘  │
│      │         │                           │         │         │
│      ▼         ▼                           ▼         ▼         │
│   ┌──────┐ ┌──────┐                    ┌──────┐ ┌──────┐       │
│   │alice │ │carol │                    │mike  │ │steve │       │
│   │→row 5│ │→row 2│                    │→row 8│ │→row 4│       │
│   └──────┘ └──────┘                    └──────┘ └──────┘       │
│                                           ▲                     │
│                                           │                     │
│   Search path: root → right → left leaf → Found!                │
│   Only 3 comparisons instead of scanning all rows               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Hash Index

Hash indexes use a hash function for O(1) lookups but don't support range queries.

```
┌─────────────────────────────────────────────────────────────────┐
│                     HASH INDEX                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   hash('alice@mail.com') = 7                                    │
│   hash('bob@mail.com') = 3                                      │
│   hash('carol@mail.com') = 7  (collision!)                      │
│                                                                 │
│   Hash Table:                                                   │
│   ┌────────┬────────────────────────────────┐                  │
│   │ Bucket │ Entries                        │                  │
│   ├────────┼────────────────────────────────┤                  │
│   │   0    │                                │                  │
│   │   1    │                                │                  │
│   │   2    │                                │                  │
│   │   3    │ bob@mail.com → Row 2           │                  │
│   │   4    │                                │                  │
│   │   5    │                                │                  │
│   │   6    │                                │                  │
│   │   7    │ alice@mail.com → Row 5         │                  │
│   │        │ carol@mail.com → Row 8         │ (chained)        │
│   └────────┴────────────────────────────────┘                  │
│                                                                 │
│   ✓ O(1) exact lookups                                          │
│   ✗ No range queries (WHERE age > 25)                           │
│   ✗ No ordering                                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Types of Indexes

### Primary Index (Clustered Index)

Determines the physical order of data in the table. Each table can have only ONE.

```
┌─────────────────────────────────────────────────────────────────┐
│                   CLUSTERED INDEX                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Table with clustered index on ID:                             │
│                                                                 │
│   Data is PHYSICALLY sorted by ID:                              │
│                                                                 │
│   Disk:  [Row 1] [Row 2] [Row 3] [Row 4] [Row 5] ...            │
│             │       │       │       │       │                   │
│             ▼       ▼       ▼       ▼       ▼                   │
│           ID=1    ID=2    ID=3    ID=4    ID=5                  │
│                                                                 │
│   ✓ Range queries on ID are FAST (sequential read)              │
│   ✓ Data IS the index (no extra lookup)                         │
│   ✗ Only ONE per table                                          │
│   ✗ Inserts may require reorganization                          │
│                                                                 │
│   In MySQL InnoDB: Primary key is clustered index               │
│   In PostgreSQL: No clustered index (use CLUSTER command)       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Secondary Index (Non-Clustered Index)

Separate structure that points to the data. A table can have multiple secondary indexes.

```
┌─────────────────────────────────────────────────────────────────┐
│                SECONDARY (NON-CLUSTERED) INDEX                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Secondary index on email:                                     │
│                                                                 │
│   INDEX (separate structure)         TABLE (data)               │
│   ┌─────────────────┬─────┐         ┌────┬─────────┬─────┐     │
│   │ Email           │ Ptr │         │ ID │ Email   │ Name│     │
│   ├─────────────────┼─────┤         ├────┼─────────┼─────┤     │
│   │ alice@mail.com  │  ───┼────────►│ 3  │ alice@..│Alice│     │
│   │ bob@mail.com    │  ───┼────┐    │ 1  │ bob@... │ Bob │     │
│   │ carol@mail.com  │  ───┼──┐ │    │ 2  │ carol@..│Carol│     │
│   └─────────────────┴─────┘  │ └───►│ .. │ ...     │ ... │     │
│                              │      └────┴─────────┴─────┘     │
│                              │        ▲                         │
│                              └────────┘                         │
│                                                                 │
│   Two-step lookup:                                              │
│   1. Find email in index → get pointer                          │
│   2. Use pointer to fetch row from table                        │
│                                                                 │
│   ✓ Multiple per table                                          │
│   ✓ Speeds up queries on indexed column                         │
│   ✗ Extra storage for index                                     │
│   ✗ Slower writes (must update index too)                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Composite Index (Multi-Column Index)

Index on multiple columns. Column order matters!

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMPOSITE INDEX                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   CREATE INDEX idx ON users(last_name, first_name);             │
│                                                                 │
│   Index sorted by last_name, then first_name:                   │
│                                                                 │
│   ┌─────────────┬─────────────┬─────┐                          │
│   │ Last Name   │ First Name  │ Ptr │                          │
│   ├─────────────┼─────────────┼─────┤                          │
│   │ Adams       │ Alice       │ →   │                          │
│   │ Adams       │ Bob         │ →   │                          │
│   │ Baker       │ Carol       │ →   │                          │
│   │ Baker       │ David       │ →   │                          │
│   │ Clark       │ Alice       │ →   │                          │
│   └─────────────┴─────────────┴─────┘                          │
│                                                                 │
│   QUERIES THAT USE THIS INDEX:                                  │
│   ✓ WHERE last_name = 'Adams'                                   │
│   ✓ WHERE last_name = 'Adams' AND first_name = 'Bob'            │
│   ✓ WHERE last_name = 'Adams' AND first_name LIKE 'B%'          │
│                                                                 │
│   QUERIES THAT DON'T (fully) USE THIS INDEX:                    │
│   ✗ WHERE first_name = 'Alice'      (wrong order!)              │
│   ✗ WHERE last_name LIKE '%dams'    (leading wildcard)          │
│                                                                 │
│   RULE: Index used left-to-right, can stop but can't skip       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Covering Index

An index that contains all columns needed for a query, avoiding table lookup.

```
┌─────────────────────────────────────────────────────────────────┐
│                    COVERING INDEX                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Query: SELECT email, name FROM users WHERE email = '...'      │
│                                                                 │
│   Regular Index (on email only):                                │
│   ┌─────────────────┬─────┐     ┌───────────────────────────┐  │
│   │ Email           │ Ptr │────►│ id, email, name, age, ... │  │
│   └─────────────────┴─────┘     └───────────────────────────┘  │
│   Index lookup          +       Table lookup (extra I/O)        │
│                                                                 │
│                                                                 │
│   Covering Index (on email, name):                              │
│   ┌─────────────────┬─────────┐                                │
│   │ Email           │ Name    │ ◄── All needed data in index!  │
│   ├─────────────────┼─────────┤                                │
│   │ alice@mail.com  │ Alice   │     No table lookup needed     │
│   │ bob@mail.com    │ Bob     │                                │
│   └─────────────────┴─────────┘                                │
│                                                                 │
│   CREATE INDEX idx ON users(email) INCLUDE (name);              │
│                                                                 │
│   ✓ Fastest possible query (index-only scan)                    │
│   ✗ Larger index size                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Index Trade-offs

### Write Penalty

```
┌─────────────────────────────────────────────────────────────────┐
│                INDEX WRITE PENALTY                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   INSERT INTO users VALUES (100, 'new@mail.com', 'New');        │
│                                                                 │
│   Without index:                                                │
│   └── Write row to table                                        │
│   └── Done! (fast)                                              │
│                                                                 │
│   With 3 indexes:                                               │
│   └── Write row to table                                        │
│   └── Update index 1 (email)                                    │
│   └── Update index 2 (name)                                     │
│   └── Update index 3 (created_at)                               │
│   └── Done (slower!)                                            │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │           MORE INDEXES = FASTER READS                   │  │
│   │                         BUT                             │  │
│   │                       SLOWER WRITES                     │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
│   Rule of thumb:                                                │
│   • Read-heavy workload: More indexes OK                        │
│   • Write-heavy workload: Fewer indexes                         │
│   • OLTP (mixed): Balance carefully                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Storage Overhead

```
┌─────────────────────────────────────────────────────────────────┐
│                 INDEX STORAGE COST                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Table: users (1 million rows)                                 │
│   ┌──────────────────────────────────────────────┐             │
│   │ Table size: 100 MB                           │             │
│   │                                              │             │
│   │ Index on email (varchar 100): +50 MB         │             │
│   │ Index on name (varchar 50): +30 MB           │             │
│   │ Index on created_at (timestamp): +20 MB      │             │
│   │ Composite index (email, name): +70 MB        │             │
│   │                                              │             │
│   │ Total with indexes: 270 MB (2.7x table size!)│             │
│   └──────────────────────────────────────────────┘             │
│                                                                 │
│   Indexes need RAM too! They're most effective when             │
│   they fit in memory.                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## When to Create Indexes

### Good Candidates for Indexing

```
┌────────────────────────────────────────────────────────────────┐
│                   INDEX CANDIDATES                             │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ✓ PRIMARY KEYS (automatically indexed)                        │
│    └── id, uuid                                                │
│                                                                │
│  ✓ FOREIGN KEYS                                                │
│    └── user_id, order_id (used in JOINs)                       │
│                                                                │
│  ✓ COLUMNS IN WHERE CLAUSES                                    │
│    └── email (WHERE email = ?)                                 │
│    └── status (WHERE status = 'active')                        │
│                                                                │
│  ✓ COLUMNS IN ORDER BY                                         │
│    └── created_at (ORDER BY created_at DESC)                   │
│                                                                │
│  ✓ COLUMNS IN GROUP BY                                         │
│    └── category (GROUP BY category)                            │
│                                                                │
│  ✓ HIGH CARDINALITY COLUMNS                                    │
│    └── email (unique or near-unique values)                    │
│    └── user_id (many distinct values)                          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Poor Candidates for Indexing

```
┌────────────────────────────────────────────────────────────────┐
│                 AVOID INDEXING                                 │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ✗ LOW CARDINALITY COLUMNS                                     │
│    └── gender (only M/F/Other)                                 │
│    └── boolean flags (only true/false)                         │
│    └── status with few values                                  │
│                                                                │
│  ✗ RARELY QUERIED COLUMNS                                      │
│    └── If never in WHERE, no need for index                    │
│                                                                │
│  ✗ FREQUENTLY UPDATED COLUMNS                                  │
│    └── Counter columns                                         │
│    └── Last_accessed timestamps                                │
│                                                                │
│  ✗ VERY WIDE COLUMNS                                           │
│    └── Large TEXT or BLOB fields                               │
│    └── (Consider full-text index instead)                      │
│                                                                │
│  ✗ SMALL TABLES                                                │
│    └── Full table scan may be faster than index lookup         │
│    └── < 1000 rows usually don't need indexes                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## Understanding EXPLAIN

Use EXPLAIN to see how queries use indexes.

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXPLAIN OUTPUT                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   EXPLAIN SELECT * FROM users WHERE email = 'alice@mail.com';   │
│                                                                 │
│   WITHOUT INDEX:                                                │
│   ┌────────────────────────────────────────────────────────┐   │
│   │ type: ALL                    ◄── Full table scan!      │   │
│   │ rows: 1000000                ◄── Scanning all rows     │   │
│   │ filtered: 10%                                          │   │
│   │ Extra: Using where                                     │   │
│   └────────────────────────────────────────────────────────┘   │
│                                                                 │
│   WITH INDEX:                                                   │
│   ┌────────────────────────────────────────────────────────┐   │
│   │ type: ref                    ◄── Using index!          │   │
│   │ possible_keys: idx_email                               │   │
│   │ key: idx_email               ◄── This index used       │   │
│   │ rows: 1                      ◄── Only 1 row examined   │   │
│   │ Extra: Using index                                     │   │
│   └────────────────────────────────────────────────────────┘   │
│                                                                 │
│   KEY METRICS TO WATCH:                                         │
│   • type: ALL (bad) → index/ref/const (good)                    │
│   • rows: Should be small relative to table size                │
│   • key: Should show an index being used                        │
│   • Extra: "Using filesort" or "Using temporary" = slow         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Type Column Values (Best to Worst)

```
┌────────────────────────────────────────────────────────────────┐
│                   EXPLAIN TYPE VALUES                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  BEST                                                          │
│   │                                                            │
│   │  const       - Single row match (primary key = value)      │
│   │  eq_ref      - One row per table (unique index)            │
│   │  ref         - Multiple rows from index                    │
│   │  range       - Index range scan (BETWEEN, <, >)            │
│   │  index       - Full index scan (better than table scan)    │
│   │  ALL         - Full table scan (AVOID!)                    │
│   │                                                            │
│  WORST                                                         │
│                                                                │
│  If you see ALL, you probably need an index!                   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## Common Indexing Mistakes

### Mistake 1: Indexing Low-Cardinality Columns

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   BAD: CREATE INDEX idx ON users(gender);                       │
│                                                                 │
│   Only 3 values: 'M', 'F', 'Other'                              │
│   Index returns 33% of table → full scan may be faster!         │
│                                                                 │
│   BETTER: Use composite index if combined with high-cardinality │
│   CREATE INDEX idx ON users(gender, last_login);                │
│   Now useful for: WHERE gender='F' AND last_login > '...'       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Mistake 2: Wrong Column Order in Composite Index

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Query: WHERE country = 'US' AND email = 'alice@...'           │
│                                                                 │
│   BAD:  INDEX(country, email)                                   │
│         country has ~200 values (low cardinality)               │
│         Must scan many country='US' entries to find email       │
│                                                                 │
│   GOOD: INDEX(email, country)                                   │
│         email is unique (high cardinality)                      │
│         Finds exact email immediately                           │
│                                                                 │
│   RULE: Put highest cardinality columns first                   │
│         (unless you query low-cardinality column alone)         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Mistake 3: Too Many Indexes

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Table with 10 indexes:                                        │
│   • Every INSERT/UPDATE must update 10 indexes                  │
│   • Storage: 10x overhead                                       │
│   • Memory: indexes compete for buffer pool space               │
│                                                                 │
│   SIGNS OF OVER-INDEXING:                                       │
│   • Write performance degraded                                  │
│   • Indexes larger than table                                   │
│   • Many indexes show 0 usage in statistics                     │
│                                                                 │
│   SOLUTION:                                                     │
│   • Monitor index usage (pg_stat_user_indexes)                  │
│   • Drop unused indexes                                         │
│   • Consider composite indexes instead of multiple single       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Mistake 4: Functions on Indexed Columns

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Index on: email                                               │
│                                                                 │
│   BAD: WHERE LOWER(email) = 'alice@mail.com'                    │
│         LOWER() prevents index usage!                           │
│         Database must compute LOWER() for every row             │
│                                                                 │
│   GOOD: WHERE email = 'alice@mail.com'                          │
│         Or store email as lowercase and query directly          │
│                                                                 │
│   ALTERNATIVE: Functional index (PostgreSQL)                    │
│   CREATE INDEX idx ON users(LOWER(email));                      │
│   Now LOWER(email) = '...' will use index                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Question 1: "When would you NOT use an index?"

**Strong Answer:**
```
I would avoid creating an index when:

1. LOW CARDINALITY
   • Column has few distinct values (gender, boolean)
   • Index would return large percentage of table
   • Full scan may be faster

2. SMALL TABLE
   • Less than ~1000 rows
   • Full scan is fast enough
   • Index overhead not worth it

3. WRITE-HEAVY TABLE
   • Logging tables with constant inserts
   • Index maintenance slows writes
   • Consider batch indexing or no index

4. RARELY QUERIED COLUMN
   • Column never appears in WHERE/ORDER BY
   • Index is just wasted space

5. FREQUENTLY UPDATED COLUMN
   • Counter columns, timestamps
   • Constant index updates

Key insight: Indexes are a trade-off. They speed up reads
but slow down writes and consume storage. Always measure!
```

### Question 2: "Explain how you would optimize a slow query"

**Strong Answer:**
```
Step-by-step process:

1. IDENTIFY THE PROBLEM
   • Run EXPLAIN/EXPLAIN ANALYZE
   • Look for: type=ALL, high row estimates, missing keys

2. CHECK EXISTING INDEXES
   • Is there an index that should be used?
   • Is the query written to use it? (no functions, right order)

3. ANALYZE THE QUERY PATTERN
   • What columns in WHERE, JOIN, ORDER BY, GROUP BY?
   • What's the cardinality of filtered columns?

4. DESIGN THE INDEX
   • Composite index for multi-column queries
   • Column order: equality columns first, then ranges
   • Consider covering index if SELECT is limited

5. MEASURE IMPROVEMENT
   • Compare EXPLAIN before/after
   • Test with production-like data volume
   • Monitor write performance too

Example:
Query: SELECT * FROM orders
       WHERE user_id = 123
       AND status = 'shipped'
       ORDER BY created_at DESC;

Index: CREATE INDEX idx ON orders(user_id, status, created_at DESC);
```

---

## Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│                    REMEMBER THIS                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. Indexes make reads faster (O(n) → O(log n))                │
│     But make writes slower (must update index)                 │
│                                                                │
│  2. B-Tree = most common, supports ranges                      │
│     Hash = O(1) exact match only                               │
│                                                                │
│  3. Composite index column ORDER matters                       │
│     Index on (A, B) helps: A=?, A=? AND B=?                    │
│     Does NOT help: B=? alone                                   │
│                                                                │
│  4. Covering index = all needed columns in index               │
│     Avoids table lookup (fastest)                              │
│                                                                │
│  5. Index high-cardinality columns (many distinct values)      │
│     Don't index low-cardinality (gender, boolean)              │
│                                                                │
│  6. Always use EXPLAIN to verify index usage                   │
│     Look for: type, rows, key                                  │
│                                                                │
│  7. Functions on columns prevent index usage                   │
│     WHERE LOWER(email) = ? won't use index on email            │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

Now that you understand indexing, let's learn about consistent hashing—a technique for distributing data across multiple nodes.

**Next:** [Consistent Hashing →](consistent_hashing.md)
