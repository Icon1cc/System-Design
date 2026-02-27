# Read-Heavy vs Write-Heavy Systems

## Understanding Your Workload

The read:write ratio fundamentally shapes your architecture. Different ratios require completely different optimization strategies.

```
┌─────────────────────────────────────────────────────────────┐
│             READ:WRITE RATIO SPECTRUM                       │
│                                                             │
│  100% Reads              Balanced              100% Writes  │
│      │                      │                      │        │
│      ▼                      ▼                      ▼        │
│  ┌───────┐              ┌───────┐              ┌───────┐   │
│  │ CDN   │              │ E-com │              │ IoT   │   │
│  │Wikipedia             │ Social│              │Logging│   │
│  │News   │              │       │              │Metrics│   │
│  └───────┘              └───────┘              └───────┘   │
│                                                             │
│  TYPICAL RATIOS:                                            │
│  ├── Wikipedia: 1000:1 (reads dominate)                   │
│  ├── Twitter: 100:1 (mostly reading feeds)                │
│  ├── E-commerce: 10:1 (browse vs buy)                     │
│  ├── Email: 2:1 (read and compose)                        │
│  ├── Chat: 1:1 (send and receive)                         │
│  ├── IoT sensors: 1:100 (constant writes)                 │
│  └── Logging: 1:1000 (writes dominate)                    │
└─────────────────────────────────────────────────────────────┘
```

## Read-Heavy Systems

### Characteristics

```
┌─────────────────────────────────────────────────────────────┐
│           READ-HEAVY CHARACTERISTICS                        │
│                                                             │
│  EXAMPLES:                                                  │
│  ├── Social media feeds                                    │
│  ├── News websites                                         │
│  ├── Product catalogs                                      │
│  ├── Wikipedia/documentation                               │
│  └── Search engines                                        │
│                                                             │
│  METRICS:                                                   │
│  - Read:Write ratio > 10:1                                │
│  - Read latency critical (< 100ms)                        │
│  - Write latency less critical                            │
│  - Same content served to many users                      │
│                                                             │
│  CHALLENGES:                                                │
│  - Database overwhelmed by reads                          │
│  - Network bandwidth for responses                        │
│  - Keeping cached data fresh                              │
└─────────────────────────────────────────────────────────────┘
```

### Optimization Strategies

```
┌─────────────────────────────────────────────────────────────┐
│         READ-HEAVY OPTIMIZATIONS                            │
│                                                             │
│  1. AGGRESSIVE CACHING                                      │
│     ┌──────────────────────────────────────────┐          │
│     │                   CDN                     │          │
│     │            (Static + API cache)          │          │
│     └──────────────────┬───────────────────────┘          │
│                        │ Cache miss                        │
│     ┌──────────────────▼───────────────────────┐          │
│     │             Application Cache             │          │
│     │                (Redis)                   │          │
│     └──────────────────┬───────────────────────┘          │
│                        │ Cache miss                        │
│     ┌──────────────────▼───────────────────────┐          │
│     │               Read Replicas              │          │
│     │         (Multiple DB replicas)           │          │
│     └──────────────────────────────────────────┘          │
│                                                             │
│  TARGET: 90-99% cache hit rate                            │
│  DB only serves 1-10% of original traffic                │
│                                                             │
│  2. READ REPLICAS                                           │
│     - Scale reads horizontally                            │
│     - Geographic distribution                             │
│     - Accept slight replication lag                       │
│                                                             │
│  3. DENORMALIZATION                                         │
│     - Pre-join data for fast reads                        │
│     - Accept data duplication                             │
│     - Slower writes, faster reads                         │
│                                                             │
│  4. MATERIALIZED VIEWS                                      │
│     - Pre-compute complex queries                         │
│     - Update incrementally on writes                      │
│     - Serve pre-computed results                          │
└─────────────────────────────────────────────────────────────┘
```

### Read-Heavy Architecture: Twitter Timeline

```
┌─────────────────────────────────────────────────────────────┐
│          TWITTER TIMELINE ARCHITECTURE                      │
│                                                             │
│  PROBLEM: 100K+ read QPS for timelines                    │
│                                                             │
│  TWO APPROACHES:                                            │
│                                                             │
│  A) FAN-OUT ON READ (Pull model)                          │
│     User requests timeline:                               │
│     1. Get list of followed users (100 users)            │
│     2. Fetch recent tweets from each (100 queries!)      │
│     3. Merge and sort                                      │
│     4. Return top 50                                       │
│                                                             │
│     ❌ Slow: N queries per request                        │
│     ❌ Doesn't scale                                       │
│     ✅ Simple                                              │
│     ✅ Writes are fast                                     │
│                                                             │
│  B) FAN-OUT ON WRITE (Push model)                         │
│     When user tweets:                                      │
│     1. Write tweet to tweets table                        │
│     2. Push to each follower's timeline cache            │
│     3. (Async) Update all follower timelines             │
│                                                             │
│     ✅ Reads are instant (pre-computed)                   │
│     ❌ Celebrity problem (1M followers = 1M writes)       │
│     ❌ Complex                                             │
│                                                             │
│  HYBRID SOLUTION (Twitter's actual approach):             │
│  - Fan-out for normal users                               │
│  - Pull for celebrities (>10K followers)                  │
│  - Merge at read time                                      │
│                                                             │
│     Timeline = Cached timeline + Pull celebrity tweets    │
└─────────────────────────────────────────────────────────────┘
```

### Caching Patterns for Reads

```
┌─────────────────────────────────────────────────────────────┐
│            CACHING PATTERNS                                 │
│                                                             │
│  CACHE-ASIDE (Lazy Loading):                               │
│  ┌─────────────────────────────────────────────┐          │
│  │  1. App checks cache                         │          │
│  │  2. Cache miss → Query DB                   │          │
│  │  3. Store result in cache                   │          │
│  │  4. Return to user                          │          │
│  └─────────────────────────────────────────────┘          │
│  ✅ Only caches what's needed                             │
│  ❌ First request always slow                             │
│                                                             │
│  READ-THROUGH:                                              │
│  ┌─────────────────────────────────────────────┐          │
│  │  1. App queries cache                        │          │
│  │  2. Cache handles DB query if miss          │          │
│  │  3. Cache stores and returns                │          │
│  └─────────────────────────────────────────────┘          │
│  ✅ Simpler app code                                      │
│  ❌ Cache needs DB access                                 │
│                                                             │
│  CACHE WARMING:                                             │
│  ┌─────────────────────────────────────────────┐          │
│  │  Pre-populate cache before traffic          │          │
│  │  - On deployment                            │          │
│  │  - On cache restart                         │          │
│  │  - Predictive (trending content)            │          │
│  └─────────────────────────────────────────────┘          │
│  ✅ No cold-start problem                                 │
│  ❌ May cache unused data                                 │
└─────────────────────────────────────────────────────────────┘
```

## Write-Heavy Systems

### Characteristics

```
┌─────────────────────────────────────────────────────────────┐
│           WRITE-HEAVY CHARACTERISTICS                       │
│                                                             │
│  EXAMPLES:                                                  │
│  ├── Logging systems                                       │
│  ├── IoT data collection                                   │
│  ├── Metrics/telemetry                                     │
│  ├── Financial transactions                                │
│  └── Real-time bidding                                     │
│                                                             │
│  METRICS:                                                   │
│  - Read:Write ratio < 1:1                                 │
│  - Write throughput critical                              │
│  - May tolerate read delays                               │
│  - Data often time-series                                 │
│                                                             │
│  CHALLENGES:                                                │
│  - Write amplification (indexes, replicas)               │
│  - Lock contention                                         │
│  - Disk I/O bottleneck                                    │
│  - Replication lag                                         │
└─────────────────────────────────────────────────────────────┘
```

### Optimization Strategies

```
┌─────────────────────────────────────────────────────────────┐
│         WRITE-HEAVY OPTIMIZATIONS                           │
│                                                             │
│  1. WRITE BATCHING                                          │
│     Instead of:  1000 individual inserts                  │
│     Do:          1 batch insert of 1000 rows              │
│                                                             │
│     Reduces:                                                │
│     - Network round trips                                  │
│     - Transaction overhead                                 │
│     - Lock acquisitions                                    │
│                                                             │
│  2. ASYNCHRONOUS WRITES                                     │
│     ┌─────────┐     ┌─────────┐     ┌─────────┐          │
│     │ Client  │────▶│  Queue  │────▶│ Database│          │
│     └─────────┘     │ (Kafka) │     └─────────┘          │
│         │           └─────────┘                            │
│         │                                                   │
│     Return immediately                                     │
│     (acknowledge to queue)                                 │
│                                                             │
│  3. APPEND-ONLY DESIGN                                      │
│     ┌───────────────────────────────────────┐             │
│     │  Instead of UPDATE, INSERT new row    │             │
│     │  Append is faster than random write   │             │
│     │  Compact later (LSM trees)           │             │
│     └───────────────────────────────────────┘             │
│     Used by: Cassandra, RocksDB, LevelDB                 │
│                                                             │
│  4. REDUCE INDEXES                                          │
│     Each index = additional write                         │
│     Only index what's queried                             │
│     Consider: Build indexes async                         │
│                                                             │
│  5. PARTITIONING/SHARDING                                   │
│     Split writes across multiple nodes                    │
│     Each shard handles subset of data                     │
└─────────────────────────────────────────────────────────────┘
```

### Write-Heavy Architecture: Logging System

```
┌─────────────────────────────────────────────────────────────┐
│          LOGGING SYSTEM ARCHITECTURE                        │
│                                                             │
│  REQUIREMENTS:                                              │
│  - 1 million log events per second                        │
│  - 30 day retention                                        │
│  - Searchable within minutes                              │
│                                                             │
│          Application Servers                               │
│     ┌────┐ ┌────┐ ┌────┐ ┌────┐                          │
│     │App1│ │App2│ │App3│ │App4│                          │
│     └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘                          │
│        │      │      │      │  Log events                │
│        └──────┴──────┴──────┘                             │
│                   │                                        │
│            ┌──────▼──────┐                                 │
│            │   KAFKA     │  Buffer writes                 │
│            │  (Queue)    │  Handle bursts                 │
│            └──────┬──────┘                                 │
│                   │                                        │
│     ┌─────────────┼─────────────┐                         │
│     │             │             │                         │
│     ▼             ▼             ▼                         │
│  ┌──────┐    ┌──────┐    ┌──────┐                        │
│  │Elastic    │Elastic    │Elastic                        │
│  │Node 1│    │Node 2│    │Node 3│  Write in parallel    │
│  └──────┘    └──────┘    └──────┘                        │
│                                                             │
│  WHY THIS WORKS:                                            │
│  - Kafka buffers bursts, smooths write rate              │
│  - Elasticsearch optimized for append                    │
│  - Horizontal scaling via sharding                       │
│  - Batch writes (bulk API)                               │
└─────────────────────────────────────────────────────────────┘
```

### Database Choices for Write-Heavy

```
┌─────────────────────────────────────────────────────────────┐
│         DATABASES FOR WRITE-HEAVY                           │
│                                                             │
│  1. CASSANDRA / ScyllaDB                                   │
│     - Optimized for writes                                 │
│     - LSM tree (append-only)                              │
│     - Linear scale                                         │
│     - 50K+ writes/sec per node                           │
│                                                             │
│  2. TIMESCALEDB / InfluxDB                                 │
│     - Purpose-built for time-series                       │
│     - Automatic partitioning by time                      │
│     - Compression                                          │
│                                                             │
│  3. KAFKA + Downstream                                      │
│     - Not a database, but buffer                          │
│     - Handles millions of writes/sec                      │
│     - Consumers process asynchronously                    │
│                                                             │
│  4. MONGODB with Write Concern 0                          │
│     - Fire and forget                                      │
│     - Very fast, no durability                            │
│     - Good for metrics you can lose                       │
│                                                             │
│  AVOID FOR WRITE-HEAVY:                                     │
│  - Single MySQL/PostgreSQL (write bottleneck)            │
│  - Heavily indexed tables                                 │
│  - Synchronous replication                                │
└─────────────────────────────────────────────────────────────┘
```

## Mixed Workloads

### CQRS Pattern

```
┌─────────────────────────────────────────────────────────────┐
│    CQRS (Command Query Responsibility Segregation)         │
│                                                             │
│  Separate read and write models completely                 │
│                                                             │
│          COMMANDS                    QUERIES               │
│          (Writes)                    (Reads)               │
│             │                           │                  │
│             ▼                           ▼                  │
│     ┌───────────────┐          ┌───────────────┐          │
│     │    Write      │          │    Read       │          │
│     │    Model      │          │    Model      │          │
│     │  (Normalized) │          │(Denormalized) │          │
│     └───────┬───────┘          └───────────────┘          │
│             │                           ▲                  │
│             │      Event/Sync           │                  │
│             └───────────────────────────┘                  │
│                                                             │
│  EXAMPLE: E-commerce                                        │
│                                                             │
│  WRITE MODEL (MySQL):                                       │
│  - orders, order_items, inventory                         │
│  - Normalized, ACID                                        │
│  - Handles: place order, update inventory                 │
│                                                             │
│  READ MODEL (Elasticsearch):                               │
│  - Denormalized order documents                           │
│  - Optimized for search and display                       │
│  - Handles: order history, search, analytics             │
│                                                             │
│  SYNC: Events from write → Update read model             │
└─────────────────────────────────────────────────────────────┘
```

### Event Sourcing for Mixed Workloads

```
┌─────────────────────────────────────────────────────────────┐
│              EVENT SOURCING                                 │
│                                                             │
│  Store events, not current state                          │
│                                                             │
│  TRADITIONAL:                     EVENT SOURCING:          │
│  ┌─────────────────┐             ┌─────────────────┐      │
│  │ account: $100   │             │ Events:         │      │
│  │ (current state) │             │ 1. Created $0   │      │
│  └─────────────────┘             │ 2. Deposit $200 │      │
│                                  │ 3. Withdraw $50 │      │
│                                  │ 4. Withdraw $50 │      │
│                                  │ = $100 current  │      │
│                                  └─────────────────┘      │
│                                                             │
│  ARCHITECTURE:                                              │
│     ┌─────────────────────────────────────────────┐       │
│     │                 COMMANDS                     │       │
│     └─────────────────────┬───────────────────────┘       │
│                           │                                │
│                           ▼                                │
│     ┌─────────────────────────────────────────────┐       │
│     │              EVENT STORE                     │       │
│     │  (Append-only log - very write efficient)  │       │
│     └─────────────────────┬───────────────────────┘       │
│                           │                                │
│           ┌───────────────┼───────────────┐               │
│           │               │               │               │
│           ▼               ▼               ▼               │
│     ┌──────────┐   ┌──────────┐   ┌──────────┐          │
│     │ Read     │   │ Analytics│   │ Search   │          │
│     │ Model    │   │ Model    │   │ Index    │          │
│     │(Current  │   │(Aggreg-  │   │(Elastic- │          │
│     │ State)   │   │ ations)  │   │ search)  │          │
│     └──────────┘   └──────────┘   └──────────┘          │
│                                                             │
│  BENEFITS:                                                  │
│  - Writes are super fast (append only)                    │
│  - Complete audit trail                                    │
│  - Build any read model from events                       │
│  - Time travel (replay to any point)                      │
│                                                             │
│  CHALLENGES:                                                │
│  - Eventual consistency                                    │
│  - Event schema evolution                                  │
│  - Complexity                                              │
└─────────────────────────────────────────────────────────────┘
```

## Decision Framework

```
┌─────────────────────────────────────────────────────────────┐
│         WORKLOAD OPTIMIZATION DECISION TREE                 │
│                                                             │
│  What's your read:write ratio?                            │
│           │                                                 │
│     ┌─────┴─────┬────────────┐                            │
│     │           │            │                            │
│  >10:1       1:1-10:1      <1:10                          │
│ Read-heavy   Balanced    Write-heavy                      │
│     │           │            │                            │
│     ▼           ▼            ▼                            │
│ ┌────────┐ ┌────────┐  ┌────────┐                        │
│ │Cache   │ │CQRS    │  │LSM DB  │                        │
│ │heavy   │ │Pattern │  │Batching│                        │
│ │Read    │ │Event   │  │Async   │                        │
│ │replicas│ │sourcing│  │Writes  │                        │
│ └────────┘ └────────┘  └────────┘                        │
│                                                             │
│  OPTIMIZATION PRIORITY:                                     │
│                                                             │
│  Read-Heavy:              Write-Heavy:                     │
│  1. Add caching           1. Use message queue            │
│  2. Add read replicas     2. Batch writes                 │
│  3. CDN for static        3. Reduce indexes              │
│  4. Denormalize           4. Async replication           │
│  5. Pre-compute views     5. Partition/shard             │
└─────────────────────────────────────────────────────────────┘
```

## Interview Tips

```
1. ALWAYS IDENTIFY THE RATIO
   "First, let me understand the workload. This seems like
    a read-heavy system with roughly 100:1 ratio..."

2. MATCH STRATEGY TO WORKLOAD
   "Since we're read-heavy, I'll focus on caching and
    read replicas rather than write optimization."

3. KNOW THE TRADEOFFS
   "Denormalization speeds up reads but makes writes more
    complex and can cause consistency issues."

4. MENTION SPECIFIC PATTERNS
   "For this write-heavy log system, I'd use Kafka to
    buffer writes and Cassandra for storage since it's
    optimized for write throughput."

5. DISCUSS EVOLUTION
   "We'd start with a single database, add caching when
    reads become a bottleneck, then consider CQRS if we
    need independent scaling."
```

---

**Next:** [11_Bottleneck_Detection.md](./11_Bottleneck_Detection.md) - Find and fix performance issues
