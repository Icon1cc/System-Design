# Data Modeling Strategy

## What is Data Modeling?

Data modeling is **designing how your data is structured, stored, and related**. It's the foundation that determines how efficiently your system can read, write, and query data.

```
┌─────────────────────────────────────────────────────────────┐
│              DATA MODELING = FOUNDATION                     │
│                                                             │
│         Application                                         │
│             │                                               │
│             ▼                                               │
│         API Layer                                           │
│             │                                               │
│             ▼                                               │
│      ┌─────────────┐                                       │
│      │ Data Model  │  ← This determines everything:        │
│      │   Schema    │    - Query performance                │
│      │   Indexes   │    - Scale limits                     │
│      │   Relations │    - Data integrity                   │
│      └─────────────┘    - Maintenance complexity           │
│             │                                               │
│             ▼                                               │
│         Database                                            │
└─────────────────────────────────────────────────────────────┘
```

## The Data Modeling Process

### Step 1: Identify Entities

Entities are the "things" in your system.

```
Twitter Example:
┌─────────────────────────────────────────────────────────────┐
│                    ENTITIES                                 │
│                                                             │
│  PRIMARY ENTITIES:          SECONDARY ENTITIES:             │
│  ├── User                   ├── Media                       │
│  ├── Tweet                  ├── Notification                │
│  ├── Follow                 ├── Session                     │
│  └── Like                   └── AuditLog                    │
│                                                             │
│  Ask yourself:                                              │
│  - What are the nouns in requirements?                     │
│  - What do users create/interact with?                     │
│  - What do we need to track?                               │
└─────────────────────────────────────────────────────────────┘
```

### Step 2: Define Relationships

```
RELATIONSHIP TYPES:

1. ONE-TO-ONE (1:1)
   User ──────── Profile
   "Each user has exactly one profile"

2. ONE-TO-MANY (1:N)
   User ──────<< Tweet
   "One user has many tweets"

3. MANY-TO-MANY (M:N)
   User >>────<< User (follows)
   "Users follow many users, are followed by many"

Twitter Relationships:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│    ┌──────┐                                                 │
│    │ User │                                                 │
│    └──┬───┘                                                 │
│       │                                                     │
│       ├─────1:N─────▶ Tweet                                │
│       │                  │                                  │
│       ├─────M:N─────▶ Follow (self-referential)            │
│       │                                                     │
│       └─────1:N─────▶ Like ◀─────N:1───── Tweet            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Step 3: Define Attributes

For each entity, define its fields:

```
USER ENTITY:
┌──────────────────────────────────────────────────────────┐
│  FIELD           TYPE          CONSTRAINTS               │
│  ──────────────────────────────────────────────         │
│  user_id         UUID/BIGINT   PRIMARY KEY              │
│  username        VARCHAR(30)   UNIQUE, NOT NULL         │
│  email           VARCHAR(255)  UNIQUE, NOT NULL         │
│  password_hash   VARCHAR(255)  NOT NULL                 │
│  display_name    VARCHAR(50)   NOT NULL                 │
│  bio             TEXT          NULLABLE                 │
│  avatar_url      VARCHAR(500)  NULLABLE                 │
│  created_at      TIMESTAMP     NOT NULL, DEFAULT NOW()  │
│  updated_at      TIMESTAMP     NOT NULL                 │
│  is_verified     BOOLEAN       DEFAULT FALSE            │
│  follower_count  INT           DEFAULT 0                │
│  following_count INT           DEFAULT 0                │
└──────────────────────────────────────────────────────────┘
```

### Step 4: Identify Access Patterns

**This is the MOST IMPORTANT step!**

Your schema must support your queries efficiently.

```
┌─────────────────────────────────────────────────────────────┐
│            ACCESS PATTERN ANALYSIS                          │
│                                                             │
│  QUERY                           FREQUENCY    LATENCY REQ   │
│  ─────────────────────────────────────────────────────     │
│  Get user by ID                  Very High    < 10ms       │
│  Get user by username            Very High    < 10ms       │
│  Get home timeline               Very High    < 200ms      │
│  Get user's tweets               High         < 100ms      │
│  Get tweet's likes               Medium       < 100ms      │
│  Search tweets by keyword        Medium       < 500ms      │
│  Get followers of user           Medium       < 100ms      │
│  Get trending topics             Low          < 1s         │
│                                                             │
│  HIGH FREQUENCY + LOW LATENCY = Needs optimization         │
└─────────────────────────────────────────────────────────────┘
```

## Schema Design Patterns

### Pattern 1: Normalization vs Denormalization

```
NORMALIZED (3NF):
┌─────────────────────────────────────────────────────────────┐
│  Separate tables, no duplicate data                         │
│                                                             │
│  tweets                    users                            │
│  ┌─────────────────┐      ┌────────────────┐              │
│  │ tweet_id        │      │ user_id        │              │
│  │ user_id (FK) ───┼─────▶│ username       │              │
│  │ content         │      │ display_name   │              │
│  │ created_at      │      └────────────────┘              │
│  └─────────────────┘                                       │
│                                                             │
│  To display tweet with author: JOIN required               │
│                                                             │
│  PROS: No data duplication, easy updates                   │
│  CONS: JOINs are expensive at scale                        │
└─────────────────────────────────────────────────────────────┘

DENORMALIZED:
┌─────────────────────────────────────────────────────────────┐
│  Embed related data, accept duplication                     │
│                                                             │
│  tweets                                                     │
│  ┌─────────────────────────────────────┐                   │
│  │ tweet_id                            │                   │
│  │ user_id                             │                   │
│  │ author_username      ← duplicated   │                   │
│  │ author_display_name  ← duplicated   │                   │
│  │ author_avatar        ← duplicated   │                   │
│  │ content                             │                   │
│  │ created_at                          │                   │
│  └─────────────────────────────────────┘                   │
│                                                             │
│  To display tweet with author: No JOIN needed!             │
│                                                             │
│  PROS: Fast reads, no JOINs                                │
│  CONS: Data duplication, complex updates                   │
└─────────────────────────────────────────────────────────────┘
```

**When to Denormalize:**
- Read-heavy workloads (100:1 read:write)
- Data rarely changes
- Query performance is critical
- Eventual consistency is acceptable

### Pattern 2: Embedding vs Referencing

```
EMBEDDING (Document DBs):
┌─────────────────────────────────────────────────────────────┐
│  Store related data together in one document               │
│                                                             │
│  {                                                          │
│    "tweet_id": "123",                                       │
│    "content": "Hello world",                               │
│    "author": {                        ← Embedded           │
│      "user_id": "456",                                     │
│      "username": "john",                                   │
│      "avatar": "https://..."                               │
│    },                                                       │
│    "likes": [                         ← Embedded array     │
│      {"user_id": "789", "timestamp": "..."},              │
│      {"user_id": "012", "timestamp": "..."}               │
│    ]                                                        │
│  }                                                          │
│                                                             │
│  GOOD FOR: Data accessed together, bounded arrays          │
│  BAD FOR: Unbounded arrays, frequently updated data        │
└─────────────────────────────────────────────────────────────┘

REFERENCING:
┌─────────────────────────────────────────────────────────────┐
│  Store IDs, look up separately                             │
│                                                             │
│  Tweet:                        User:                        │
│  {                             {                            │
│    "tweet_id": "123",            "user_id": "456",         │
│    "author_id": "456", ─────────▶"username": "john"        │
│    "content": "Hello"          }                            │
│  }                                                          │
│                                                             │
│  GOOD FOR: Large documents, unbounded lists                │
│  BAD FOR: When data is always accessed together            │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 3: Pre-computing for Read Performance

```
┌─────────────────────────────────────────────────────────────┐
│           PRE-COMPUTED/MATERIALIZED DATA                    │
│                                                             │
│  PROBLEM: Counting followers requires scanning             │
│  SELECT COUNT(*) FROM follows WHERE followee_id = ?        │
│  → Slow for users with millions of followers               │
│                                                             │
│  SOLUTION: Pre-compute and store count                     │
│                                                             │
│  users                                                      │
│  ┌─────────────────────────────┐                           │
│  │ user_id                     │                           │
│  │ username                    │                           │
│  │ follower_count     ← Pre-computed count                │
│  │ following_count    ← Pre-computed count                │
│  │ tweet_count        ← Pre-computed count                │
│  └─────────────────────────────┘                           │
│                                                             │
│  Update counts on follow/unfollow (async is OK)            │
└─────────────────────────────────────────────────────────────┘
```

## Complete Schema Example: Twitter

### SQL Schema

```sql
-- Users table
CREATE TABLE users (
    user_id         BIGINT PRIMARY KEY,
    username        VARCHAR(30) UNIQUE NOT NULL,
    email           VARCHAR(255) UNIQUE NOT NULL,
    password_hash   VARCHAR(255) NOT NULL,
    display_name    VARCHAR(50) NOT NULL,
    bio             TEXT,
    avatar_url      VARCHAR(500),
    is_verified     BOOLEAN DEFAULT FALSE,
    follower_count  INT DEFAULT 0,
    following_count INT DEFAULT 0,
    tweet_count     INT DEFAULT 0,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_username (username),
    INDEX idx_created_at (created_at)
);

-- Tweets table
CREATE TABLE tweets (
    tweet_id        BIGINT PRIMARY KEY,
    user_id         BIGINT NOT NULL,
    content         VARCHAR(280) NOT NULL,
    reply_to_id     BIGINT,           -- NULL if not a reply
    retweet_of_id   BIGINT,           -- NULL if not a retweet
    like_count      INT DEFAULT 0,
    retweet_count   INT DEFAULT 0,
    reply_count     INT DEFAULT 0,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (reply_to_id) REFERENCES tweets(tweet_id),

    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at),
    INDEX idx_user_created (user_id, created_at DESC)  -- For user timeline
);

-- Follows table (M:N relationship)
CREATE TABLE follows (
    follower_id     BIGINT NOT NULL,
    followee_id     BIGINT NOT NULL,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (follower_id, followee_id),
    FOREIGN KEY (follower_id) REFERENCES users(user_id),
    FOREIGN KEY (followee_id) REFERENCES users(user_id),

    INDEX idx_followee (followee_id)  -- For "who follows me" queries
);

-- Likes table
CREATE TABLE likes (
    user_id         BIGINT NOT NULL,
    tweet_id        BIGINT NOT NULL,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (user_id, tweet_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (tweet_id) REFERENCES tweets(tweet_id),

    INDEX idx_tweet (tweet_id)  -- For "who liked this" queries
);

-- Media table
CREATE TABLE media (
    media_id        BIGINT PRIMARY KEY,
    tweet_id        BIGINT NOT NULL,
    media_type      ENUM('image', 'video', 'gif') NOT NULL,
    url             VARCHAR(500) NOT NULL,
    thumbnail_url   VARCHAR(500),
    width           INT,
    height          INT,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (tweet_id) REFERENCES tweets(tweet_id),
    INDEX idx_tweet (tweet_id)
);
```

### Query Examples

```sql
-- Get user profile
SELECT * FROM users WHERE username = 'johndoe';

-- Get user's tweets (paginated)
SELECT * FROM tweets
WHERE user_id = 123
ORDER BY created_at DESC
LIMIT 20;

-- Get home timeline (simplified - real implementation is complex)
SELECT t.* FROM tweets t
JOIN follows f ON t.user_id = f.followee_id
WHERE f.follower_id = 123
ORDER BY t.created_at DESC
LIMIT 20;

-- Check if user follows another
SELECT 1 FROM follows
WHERE follower_id = 123 AND followee_id = 456;

-- Get followers of a user
SELECT u.* FROM users u
JOIN follows f ON u.user_id = f.follower_id
WHERE f.followee_id = 123
ORDER BY f.created_at DESC
LIMIT 20;
```

### NoSQL Schema (MongoDB/DynamoDB)

```
┌─────────────────────────────────────────────────────────────┐
│                 NOSQL TWITTER SCHEMA                        │
└─────────────────────────────────────────────────────────────┘

User Document:
{
  "_id": "user_123",
  "username": "johndoe",
  "email": "john@example.com",
  "display_name": "John Doe",
  "bio": "Software engineer",
  "avatar_url": "https://...",
  "is_verified": false,
  "stats": {
    "follower_count": 1500,
    "following_count": 200,
    "tweet_count": 543
  },
  "created_at": "2024-01-15T10:00:00Z"
}

Tweet Document:
{
  "_id": "tweet_456",
  "user_id": "user_123",
  "author": {                          // Denormalized
    "username": "johndoe",
    "display_name": "John Doe",
    "avatar_url": "https://...",
    "is_verified": false
  },
  "content": "Hello, world!",
  "media": [
    {
      "type": "image",
      "url": "https://..."
    }
  ],
  "stats": {
    "like_count": 42,
    "retweet_count": 5,
    "reply_count": 3
  },
  "created_at": "2024-01-15T12:30:00Z"
}

Timeline Document (Fan-out on write):
{
  "_id": "timeline_user_789",
  "user_id": "user_789",
  "tweets": [                          // Pre-computed timeline
    {
      "tweet_id": "tweet_456",
      "author_id": "user_123",
      "author_username": "johndoe",
      "content": "Hello, world!",
      "created_at": "2024-01-15T12:30:00Z"
    },
    // ... more tweets
  ],
  "last_updated": "2024-01-15T12:30:00Z"
}
```

## Indexing Strategy

### Index Types

```
┌─────────────────────────────────────────────────────────────┐
│                   INDEX TYPES                               │
│                                                             │
│  1. PRIMARY KEY INDEX                                       │
│     - Automatically created                                 │
│     - Unique identifier lookups                            │
│                                                             │
│  2. SECONDARY INDEX                                         │
│     - For non-primary key queries                          │
│     - Example: INDEX(username)                             │
│                                                             │
│  3. COMPOSITE INDEX                                         │
│     - Multiple columns                                      │
│     - Order matters! (user_id, created_at) ≠ (created_at, user_id) │
│                                                             │
│  4. COVERING INDEX                                          │
│     - Includes all query columns                           │
│     - Avoids table lookup                                  │
│                                                             │
│  5. FULL-TEXT INDEX                                         │
│     - For text search                                       │
│     - Example: Searching tweet content                     │
└─────────────────────────────────────────────────────────────┘
```

### Index Selection Guide

```
┌─────────────────────────────────────────────────────────────┐
│              WHEN TO ADD INDEXES                            │
│                                                             │
│  ADD INDEX WHEN:                                            │
│  ├── Column used in WHERE clause                           │
│  ├── Column used in JOIN condition                         │
│  ├── Column used in ORDER BY                               │
│  └── High cardinality (many unique values)                │
│                                                             │
│  DON'T ADD INDEX WHEN:                                      │
│  ├── Small table (< 1000 rows)                             │
│  ├── Low cardinality (few unique values)                   │
│  ├── Frequently updated columns                            │
│  └── Table is write-heavy                                  │
│                                                             │
│  INDEXES HAVE COST:                                         │
│  - Slower writes (index must be updated)                   │
│  - More storage space                                      │
│  - Memory usage                                            │
└─────────────────────────────────────────────────────────────┘
```

### Twitter Index Example

```sql
-- Query: Get user's recent tweets
SELECT * FROM tweets
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;

-- Optimal index:
CREATE INDEX idx_user_tweets ON tweets(user_id, created_at DESC);
-- Why: Covers both WHERE and ORDER BY, returns in order

-- Query: Search users by username prefix
SELECT * FROM users WHERE username LIKE 'john%';

-- Optimal index:
CREATE INDEX idx_username ON users(username);
-- Why: B-tree index works with prefix search

-- Query: Get tweet with author info (JOIN)
SELECT t.*, u.username, u.avatar_url
FROM tweets t
JOIN users u ON t.user_id = u.user_id
WHERE t.tweet_id = ?;

-- Indexes needed:
-- tweets: PRIMARY KEY on tweet_id (automatic)
-- users: PRIMARY KEY on user_id (automatic)
-- Optional: INDEX(user_id) on tweets if not primary key
```

## Data Modeling for Scale

### Sharding Considerations

```
┌─────────────────────────────────────────────────────────────┐
│          DESIGN FOR SHARDING                                │
│                                                             │
│  GOOD SHARD KEY:                                            │
│  ├── High cardinality                                       │
│  ├── Even distribution                                      │
│  ├── Matches access patterns                               │
│  └── Immutable                                              │
│                                                             │
│  Twitter Example:                                           │
│                                                             │
│  tweets table sharded by user_id:                          │
│  ✅ User's tweets on same shard (locality)                 │
│  ❌ Home timeline requires cross-shard queries             │
│                                                             │
│  tweets table sharded by tweet_id:                         │
│  ✅ Even distribution                                       │
│  ❌ User's tweets scattered (poor locality)                │
│                                                             │
│  SOLUTION: Multiple storage strategies                     │
│  - tweets by tweet_id (for individual lookups)            │
│  - timeline cache by user_id (for feeds)                  │
└─────────────────────────────────────────────────────────────┘
```

### Handling Hot Data

```
┌─────────────────────────────────────────────────────────────┐
│             HOT DATA STRATEGIES                             │
│                                                             │
│  PROBLEM: Celebrity user = hot partition                   │
│                                                             │
│  SOLUTIONS:                                                 │
│                                                             │
│  1. SCATTER-GATHER                                          │
│     - Split hot user's data across multiple shards        │
│     - Query all shards, merge results                      │
│                                                             │
│  2. CACHING                                                 │
│     - Cache hot user's data in memory                      │
│     - Replicate cache across regions                       │
│                                                             │
│  3. SEPARATE STORAGE                                        │
│     - Special handling for users > 1M followers           │
│     - Different fan-out strategy                           │
│                                                             │
│  4. DENORMALIZATION                                         │
│     - Pre-compute popular data                             │
│     - Store in fast, replicated cache                      │
└─────────────────────────────────────────────────────────────┘
```

## Common Data Modeling Mistakes

### Mistake 1: Ignoring Access Patterns

```
❌ WRONG:
"Let me normalize everything first, we'll optimize later"

✅ RIGHT:
"What are my top 5 queries? Let me design for those."
```

### Mistake 2: Over-Normalization

```
❌ WRONG:
Creating 15 tables for perfect normalization

✅ RIGHT:
Denormalize when:
- Read:write > 10:1
- Data changes rarely
- Query performance is critical
```

### Mistake 3: Unbounded Arrays

```
❌ WRONG (Document DB):
{
  "user_id": "123",
  "followers": ["user_1", "user_2", ... "user_10000000"]
}

✅ RIGHT:
Separate collection for followers
Use pagination for large lists
```

### Mistake 4: Missing Indexes

```
❌ WRONG:
"We'll add indexes when queries get slow"

✅ RIGHT:
Design indexes with schema
Test with realistic data volume
```

## Interview Tips for Data Modeling

1. **Always start with access patterns**
   > "Before designing the schema, let me list the main queries..."

2. **Justify normalization decisions**
   > "I'm denormalizing the author info because read:write is 100:1"

3. **Consider scale implications**
   > "This schema shards well by user_id because..."

4. **Mention indexes proactively**
   > "We'll need a composite index on (user_id, created_at) for the timeline query"

5. **Discuss tradeoffs**
   > "Denormalization speeds reads but complicates updates when users change their username"

---

**Next:** [06_Database_Selection.md](./06_Database_Selection.md) - Choose the right database for your model
