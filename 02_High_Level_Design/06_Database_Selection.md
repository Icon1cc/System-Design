# Database Selection Guide

## The Database Landscape

Choosing the right database is one of the most impactful decisions in system design. There's no "best" database—only the **best fit for your requirements**.

```
┌─────────────────────────────────────────────────────────────┐
│                 DATABASE LANDSCAPE                          │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                    RELATIONAL (SQL)                  │   │
│  │  MySQL, PostgreSQL, Oracle, SQL Server              │   │
│  │  → Structured data, ACID, complex queries           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                    NOSQL                             │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │   │
│  │  │  Document   │ │  Key-Value  │ │   Column    │   │   │
│  │  │  MongoDB    │ │  Redis      │ │  Cassandra  │   │   │
│  │  │  DynamoDB   │ │  Memcached  │ │  HBase      │   │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘   │   │
│  │  ┌─────────────┐ ┌─────────────┐                   │   │
│  │  │   Graph     │ │   Search    │                   │   │
│  │  │   Neo4j     │ │  Elastic    │                   │   │
│  │  │   Neptune   │ │   search    │                   │   │
│  │  └─────────────┘ └─────────────┘                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                  SPECIALIZED                         │   │
│  │  Time-series: InfluxDB, TimescaleDB                 │   │
│  │  Message Queue: Kafka, RabbitMQ                     │   │
│  │  Object Storage: S3, GCS, Azure Blob               │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## The Decision Framework

### Quick Selection Flowchart

```
┌─────────────────────────────────────────────────────────────┐
│              DATABASE SELECTION FLOWCHART                   │
│                                                             │
│  START: What does your data look like?                     │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────┐                                   │
│  │ Structured with     │──YES──▶ Need ACID?               │
│  │ relationships?      │              │                    │
│  └─────────────────────┘              │                    │
│           │                           ▼                    │
│          NO              ┌──YES── SQL (MySQL, Postgres)   │
│           │              │                                 │
│           ▼              │                                 │
│  ┌─────────────────────┐ │                                 │
│  │ Simple key→value?  │──YES──▶ Redis / DynamoDB          │
│  └─────────────────────┘                                   │
│           │                                                 │
│          NO                                                 │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────┐                                   │
│  │ Document/flexible   │──YES──▶ MongoDB / DynamoDB       │
│  │ schema?             │                                   │
│  └─────────────────────┘                                   │
│           │                                                 │
│          NO                                                 │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────┐                                   │
│  │ Heavy writes,       │──YES──▶ Cassandra / ScyllaDB     │
│  │ time-series?        │                                   │
│  └─────────────────────┘                                   │
│           │                                                 │
│          NO                                                 │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────┐                                   │
│  │ Graph relationships?│──YES──▶ Neo4j / Neptune          │
│  └─────────────────────┘                                   │
│           │                                                 │
│          NO                                                 │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────┐                                   │
│  │ Full-text search?   │──YES──▶ Elasticsearch            │
│  └─────────────────────┘                                   │
└─────────────────────────────────────────────────────────────┘
```

## Relational Databases (SQL)

### When to Use SQL

```
✅ USE SQL WHEN:
├── Data has clear structure and relationships
├── Need ACID transactions
├── Complex queries with JOINs
├── Data integrity is critical
├── Reporting and analytics
└── Schema is relatively stable

❌ AVOID SQL WHEN:
├── Schema changes frequently
├── Horizontal scaling is primary concern
├── Simple key-value access patterns
├── Write-heavy workload (millions/sec)
└── Unstructured or semi-structured data
```

### MySQL vs PostgreSQL

```
┌─────────────────────────────────────────────────────────────┐
│              MYSQL vs POSTGRESQL                            │
│                                                             │
│  ASPECT           MySQL           PostgreSQL                │
│  ───────────────────────────────────────────────           │
│  Performance      Faster reads    Better complex queries   │
│  JSON support     Basic           Excellent (JSONB)        │
│  Replication      Strong          Very strong              │
│  Extensions       Limited         Rich (PostGIS, etc)      │
│  ACID             Yes             Yes (stricter)           │
│  Scalability      Read replicas   Read replicas            │
│  Learning curve   Easier          Moderate                 │
│                                                             │
│  CHOOSE MYSQL FOR:                                          │
│  - Simple web applications                                 │
│  - Read-heavy workloads                                    │
│  - When familiar ecosystem matters                         │
│                                                             │
│  CHOOSE POSTGRESQL FOR:                                     │
│  - Complex queries                                         │
│  - JSON data alongside relational                          │
│  - Advanced features (arrays, custom types)               │
│  - Geospatial data                                         │
└─────────────────────────────────────────────────────────────┘
```

### SQL Scaling Strategies

```
┌─────────────────────────────────────────────────────────────┐
│            SQL SCALING OPTIONS                              │
│                                                             │
│  1. VERTICAL SCALING                                        │
│     Bigger machine (more CPU, RAM, faster disk)            │
│     Limit: Eventually hits hardware ceiling                │
│                                                             │
│  2. READ REPLICAS                                           │
│     ┌────────┐     ┌────────┐                              │
│     │ Primary│────▶│Replica │  Writes to primary          │
│     │  (RW)  │     │  (R)   │  Reads from replicas        │
│     └────────┘     └────────┘                              │
│          │         ┌────────┐                              │
│          └────────▶│Replica │                              │
│                    │  (R)   │                              │
│                    └────────┘                              │
│                                                             │
│  3. SHARDING                                                │
│     Split data across multiple databases                   │
│     More complex, application-level routing               │
│                                                             │
│  4. CONNECTION POOLING                                      │
│     PgBouncer, ProxySQL                                    │
│     Handle more connections efficiently                    │
└─────────────────────────────────────────────────────────────┘
```

## Document Databases

### MongoDB / DynamoDB

```
┌─────────────────────────────────────────────────────────────┐
│              DOCUMENT DATABASE                              │
│                                                             │
│  DATA MODEL:                                                │
│  {                                                          │
│    "_id": "user_123",                                       │
│    "name": "John Doe",                                      │
│    "email": "john@example.com",                            │
│    "addresses": [                    ← Nested documents    │
│      {"type": "home", "city": "NYC"},                      │
│      {"type": "work", "city": "SF"}                        │
│    ],                                                       │
│    "preferences": {                  ← Flexible schema     │
│      "theme": "dark",                                      │
│      "notifications": true                                 │
│    }                                                        │
│  }                                                          │
│                                                             │
│  STRENGTHS:                                                 │
│  ├── Flexible schema                                       │
│  ├── Natural JSON mapping                                  │
│  ├── Horizontal scaling                                    │
│  └── Developer productivity                                │
│                                                             │
│  WEAKNESSES:                                                │
│  ├── No JOINs (application-side)                          │
│  ├── Weaker consistency guarantees                        │
│  └── Potential data duplication                           │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Document DBs

```
✅ USE DOCUMENT DB WHEN:
├── Schema evolves frequently
├── Data is naturally hierarchical
├── Need horizontal scaling
├── One-to-few relationships (embed)
├── Rapid development important
└── Different documents have different fields

❌ AVOID WHEN:
├── Many-to-many relationships
├── Need complex JOINs
├── Strong consistency required
├── Heavy aggregations/reporting
└── Data highly relational
```

### MongoDB vs DynamoDB

```
┌─────────────────────────────────────────────────────────────┐
│           MONGODB vs DYNAMODB                               │
│                                                             │
│  ASPECT           MongoDB           DynamoDB                │
│  ───────────────────────────────────────────────           │
│  Hosting          Self/Atlas        AWS only               │
│  Query language   Rich (MQL)        Limited (key-based)    │
│  Scaling          Manual sharding   Auto-scaling           │
│  Secondary idx    Flexible          Limited (GSI/LSI)      │
│  Pricing          Instance-based    Request-based          │
│  Transactions     Yes               Limited                 │
│                                                             │
│  CHOOSE MONGODB FOR:                                        │
│  - Complex queries                                         │
│  - Self-hosted or multi-cloud                             │
│  - Aggregation pipelines                                   │
│                                                             │
│  CHOOSE DYNAMODB FOR:                                       │
│  - AWS ecosystem                                           │
│  - Predictable, key-based access                          │
│  - Serverless architectures                               │
│  - Extreme scale with minimal ops                         │
└─────────────────────────────────────────────────────────────┘
```

## Key-Value Stores

### Redis

```
┌─────────────────────────────────────────────────────────────┐
│                    REDIS                                    │
│                                                             │
│  DATA STRUCTURES:                                           │
│  ├── Strings:  SET key "value"                             │
│  ├── Lists:    LPUSH list "item"                           │
│  ├── Sets:     SADD set "member"                           │
│  ├── Hashes:   HSET hash field "value"                     │
│  ├── Sorted Sets: ZADD zset 1 "member"                     │
│  └── Streams:  XADD stream * field value                   │
│                                                             │
│  USE CASES:                                                 │
│  ├── Caching (most common)                                 │
│  ├── Session storage                                       │
│  ├── Rate limiting                                         │
│  ├── Leaderboards (sorted sets)                           │
│  ├── Pub/sub messaging                                     │
│  ├── Distributed locks                                     │
│  └── Real-time analytics                                   │
│                                                             │
│  CHARACTERISTICS:                                           │
│  ├── In-memory (fast!)                                     │
│  ├── Single-threaded (simple)                             │
│  ├── Optional persistence                                  │
│  └── Cluster mode for scale                               │
│                                                             │
│  LATENCY: Sub-millisecond reads/writes                    │
│  THROUGHPUT: 100K+ ops/second per node                    │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Redis

```
✅ USE REDIS FOR:
├── Caching hot data
├── Session management
├── Rate limiting
├── Real-time leaderboards
├── Pub/sub messaging
├── Distributed locking
└── Temporary data with TTL

❌ DON'T USE REDIS AS:
├── Primary database (data loss risk)
├── Large object storage
├── Complex querying
├── Long-term storage
└── When RAM cost is prohibitive
```

## Wide-Column Stores

### Cassandra / ScyllaDB

```
┌─────────────────────────────────────────────────────────────┐
│                 WIDE-COLUMN STORES                          │
│                                                             │
│  DATA MODEL:                                                │
│  ┌─────────────────────────────────────────────────┐       │
│  │  Row Key     │  Column1  │  Column2  │ Column3  │       │
│  ├─────────────────────────────────────────────────┤       │
│  │  user:123    │  name:Joe │  age:25   │ city:NYC │       │
│  │  user:456    │  name:Ann │  email:.. │          │       │
│  │  user:789    │  name:Bob │  age:30   │ job:Eng  │       │
│  └─────────────────────────────────────────────────┘       │
│                                                             │
│  Each row can have different columns!                      │
│                                                             │
│  STRENGTHS:                                                 │
│  ├── Massive write throughput                              │
│  ├── Linear horizontal scaling                             │
│  ├── No single point of failure                           │
│  ├── Multi-datacenter replication                         │
│  └── Time-series data                                      │
│                                                             │
│  WEAKNESSES:                                                │
│  ├── Limited query flexibility                             │
│  ├── No JOINs                                              │
│  ├── Eventually consistent (by default)                   │
│  └── Requires careful data modeling                       │
└─────────────────────────────────────────────────────────────┘
```

### Use Cases for Cassandra

```
✅ USE CASSANDRA FOR:
├── Time-series data (IoT, metrics, logs)
├── Write-heavy workloads
├── High availability requirements
├── Multi-region deployments
├── Append-heavy patterns
└── When eventual consistency is OK

EXAMPLES:
├── Netflix: Viewing history
├── Instagram: Feeds
├── Uber: Trip data
└── Apple: 10+ PB in production
```

## Graph Databases

### Neo4j / Amazon Neptune

```
┌─────────────────────────────────────────────────────────────┐
│                  GRAPH DATABASE                             │
│                                                             │
│  DATA MODEL:                                                │
│                                                             │
│        ┌───────┐  FOLLOWS   ┌───────┐                     │
│        │ Alice │───────────▶│  Bob  │                     │
│        └───────┘            └───────┘                     │
│            │                    │                          │
│     LIKES  │                    │ WORKS_AT                 │
│            ▼                    ▼                          │
│        ┌───────┐            ┌───────┐                     │
│        │ Post  │            │ Google│                     │
│        └───────┘            └───────┘                     │
│                                                             │
│  NODES: Entities (User, Post, Company)                    │
│  EDGES: Relationships (FOLLOWS, LIKES, WORKS_AT)          │
│  PROPERTIES: Attributes on nodes and edges                │
│                                                             │
│  QUERY (Cypher):                                           │
│  MATCH (a:User)-[:FOLLOWS]->(b:User)-[:FOLLOWS]->(c:User) │
│  WHERE a.name = 'Alice'                                    │
│  RETURN c.name    -- Friends of friends                   │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Graph DBs

```
✅ USE GRAPH DB FOR:
├── Social networks (friends, followers)
├── Recommendation engines
├── Fraud detection
├── Knowledge graphs
├── Network/IT infrastructure
└── Complex relationship queries

EXAMPLE QUERIES (hard in SQL, easy in Graph):
├── "Friends of friends who like X"
├── "Shortest path between users"
├── "All users within 3 degrees"
├── "Detect circular references"
└── "Find influencers in network"
```

## Search Engines

### Elasticsearch

```
┌─────────────────────────────────────────────────────────────┐
│                ELASTICSEARCH                                │
│                                                             │
│  PURPOSE: Full-text search + analytics                     │
│                                                             │
│  FEATURES:                                                  │
│  ├── Full-text search with relevance scoring              │
│  ├── Fuzzy matching, typo tolerance                       │
│  ├── Faceted search                                        │
│  ├── Real-time indexing                                    │
│  ├── Aggregations and analytics                           │
│  └── Horizontal scaling                                    │
│                                                             │
│  USE CASES:                                                 │
│  ├── Site search                                           │
│  ├── Log analysis (ELK stack)                             │
│  ├── Product search (e-commerce)                          │
│  ├── Autocomplete                                          │
│  └── Analytics dashboards                                  │
│                                                             │
│  NOTE: Usually secondary to primary database              │
│  Pattern: Write to SQL/NoSQL, sync to Elasticsearch       │
└─────────────────────────────────────────────────────────────┘
```

## Polyglot Persistence

Real systems often use **multiple databases**:

```
┌─────────────────────────────────────────────────────────────┐
│            POLYGLOT PERSISTENCE EXAMPLE                     │
│                 (E-commerce Platform)                       │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   APPLICATION                        │   │
│  └──────────┬──────────┬──────────┬──────────┬────────┘   │
│             │          │          │          │             │
│             ▼          ▼          ▼          ▼             │
│        ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐        │
│        │ MySQL  │ │ Redis  │ │Elastic │ │  S3    │        │
│        │        │ │        │ │search  │ │        │        │
│        ├────────┤ ├────────┤ ├────────┤ ├────────┤        │
│        │Users   │ │Sessions│ │Product │ │Images  │        │
│        │Orders  │ │Cart    │ │Search  │ │Videos  │        │
│        │Products│ │Cache   │ │Logs    │ │Backups │        │
│        └────────┘ └────────┘ └────────┘ └────────┘        │
│                                                             │
│  WHY MULTIPLE:                                              │
│  - Each database excels at specific tasks                  │
│  - No single DB does everything well                       │
│  - Optimize for each access pattern                        │
└─────────────────────────────────────────────────────────────┘
```

## Quick Reference Matrix

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DATABASE SELECTION MATRIX                          │
│                                                                       │
│  NEED                        CONSIDER                                 │
│  ─────────────────────────────────────────────                       │
│  ACID transactions           PostgreSQL, MySQL                        │
│  Flexible schema             MongoDB, DynamoDB                        │
│  Caching                     Redis, Memcached                         │
│  Write-heavy, scale          Cassandra, ScyllaDB                      │
│  Full-text search            Elasticsearch                            │
│  Graph relationships         Neo4j, Neptune                           │
│  Time-series                 InfluxDB, TimescaleDB                    │
│  File/blob storage           S3, GCS, Azure Blob                      │
│  Message streaming           Kafka, Kinesis                           │
│  Simple key-value            Redis, DynamoDB                          │
│  Geospatial                  PostgreSQL+PostGIS, MongoDB              │
│  Real-time sync              Firebase, Supabase                       │
└──────────────────────────────────────────────────────────────────────┘
```

## Database Selection by System Type

```
┌─────────────────────────────────────────────────────────────┐
│         DATABASE BY SYSTEM TYPE                             │
│                                                             │
│  SOCIAL MEDIA (Twitter, Facebook):                         │
│  ├── MySQL/PostgreSQL: User data, relationships           │
│  ├── Cassandra: Feeds, timelines                          │
│  ├── Redis: Cache, sessions, rate limiting                │
│  └── Elasticsearch: Search                                 │
│                                                             │
│  E-COMMERCE (Amazon):                                       │
│  ├── MySQL: Transactions, orders, inventory               │
│  ├── Redis: Cart, sessions, cache                         │
│  ├── Elasticsearch: Product search                        │
│  └── S3: Product images                                    │
│                                                             │
│  MESSAGING (WhatsApp):                                      │
│  ├── Cassandra: Messages (write-heavy)                    │
│  ├── Redis: Online status, typing indicators             │
│  └── MySQL: User accounts                                  │
│                                                             │
│  RIDE-SHARING (Uber):                                       │
│  ├── MySQL/PostgreSQL: Users, trips, payments             │
│  ├── Redis: Driver locations (geospatial)                 │
│  └── Cassandra: Trip history, analytics                   │
│                                                             │
│  VIDEO STREAMING (Netflix):                                 │
│  ├── MySQL: User data, billing                            │
│  ├── Cassandra: Viewing history                           │
│  ├── Redis: Cache, recommendations                        │
│  └── S3: Video files (via CDN)                            │
└─────────────────────────────────────────────────────────────┘
```

## Interview Tips

### What Interviewers Look For

```
1. UNDERSTANDING TRADEOFFS
   "I'm choosing PostgreSQL because we need ACID for payments,
    even though it's harder to scale horizontally."

2. MATCHING TO REQUIREMENTS
   "Given the 100:1 read:write ratio and need for flexible
    schema, MongoDB is a good fit."

3. KNOWING LIMITATIONS
   "Redis is great for caching, but I wouldn't use it as the
    primary store because we could lose data."

4. POLYGLOT THINKING
   "We'll use MySQL for transactions, Redis for caching, and
    Elasticsearch for search—each for what it does best."
```

### Common Interview Questions

```
Q: "Why not just use MySQL for everything?"
A: "MySQL is excellent for ACID transactions and structured data,
    but struggles with:
    - Massive write throughput (Cassandra better)
    - Full-text search (Elasticsearch better)
    - Caching hot data (Redis better)
    Using specialized databases improves performance and scale."

Q: "When would you choose NoSQL over SQL?"
A: "NoSQL when:
    - Schema flexibility needed
    - Horizontal scaling is priority
    - Simple access patterns (key-based)
    - Eventual consistency acceptable

    SQL when:
    - Complex queries and JOINs
    - ACID transactions required
    - Data integrity critical
    - Reporting needs"

Q: "How do you keep multiple databases in sync?"
A: "Several approaches:
    - Change Data Capture (CDC) from primary
    - Application-level dual writes (careful with failures)
    - Event sourcing with consumers
    - Background sync jobs
    Accept eventual consistency between systems."
```

---

**Next:** [07_Designing_For_Scale.md](./07_Designing_For_Scale.md) - Scale your database choices
