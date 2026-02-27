# Designing for Scale

## What Does "Scale" Mean?

Scaling is handling growth—more users, more data, more requests—while maintaining performance and reliability.

```
┌─────────────────────────────────────────────────────────────┐
│                  DIMENSIONS OF SCALE                        │
│                                                             │
│  1. TRAFFIC SCALE                                           │
│     100 → 1,000 → 1,000,000 requests/second                │
│                                                             │
│  2. DATA SCALE                                              │
│     1GB → 1TB → 1PB of storage                             │
│                                                             │
│  3. USER SCALE                                              │
│     1K → 1M → 1B users                                     │
│                                                             │
│  4. GEOGRAPHIC SCALE                                        │
│     Single region → Multi-region → Global                  │
│                                                             │
│  5. TEAM SCALE                                              │
│     1 team → 10 teams → 100 teams (affects architecture)  │
└─────────────────────────────────────────────────────────────┘
```

## Vertical vs Horizontal Scaling

```
┌─────────────────────────────────────────────────────────────┐
│           VERTICAL vs HORIZONTAL SCALING                    │
│                                                             │
│  VERTICAL (Scale Up)           HORIZONTAL (Scale Out)       │
│  ┌───────────────────┐        ┌───┐ ┌───┐ ┌───┐ ┌───┐    │
│  │                   │        │ S │ │ S │ │ S │ │ S │    │
│  │    BIG SERVER     │   vs   │ E │ │ E │ │ E │ │ E │    │
│  │                   │        │ R │ │ R │ │ R │ │ R │    │
│  │ More CPU, RAM,    │        │ V │ │ V │ │ V │ │ V │    │
│  │ faster disk       │        │ E │ │ E │ │ E │ │ E │    │
│  │                   │        │ R │ │ R │ │ R │ │ R │    │
│  └───────────────────┘        └───┘ └───┘ └───┘ └───┘    │
│                                                             │
│  VERTICAL:                    HORIZONTAL:                   │
│  ✓ Simple                     ✓ No single point failure    │
│  ✓ No code changes            ✓ Theoretically unlimited    │
│  ✗ Has ceiling                ✓ Cost-effective (commodity) │
│  ✗ Single point of failure    ✗ More complex               │
│  ✗ Expensive at top           ✗ Requires distributed logic │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Each

```
START VERTICAL:
├── Small to medium scale
├── Simple architecture
├── Faster time to market
└── Budget allows for bigger hardware

THEN GO HORIZONTAL:
├── Vertical ceiling reached
├── Need high availability
├── Cost optimization at scale
└── Geographic distribution needed
```

## The Scaling Playbook

### Level 1: Single Server (0-1K users)

```
┌─────────────────────────────────────────────────────────────┐
│                LEVEL 1: SINGLE SERVER                       │
│                                                             │
│           ┌───────────────────────────────────┐            │
│           │         SINGLE SERVER             │            │
│           │  ┌──────┐ ┌────────┐ ┌────────┐  │            │
│           │  │ Web  │ │  App   │ │   DB   │  │            │
│           │  │Server│ │ Server │ │        │  │            │
│           │  └──────┘ └────────┘ └────────┘  │            │
│           └───────────────────────────────────┘            │
│                                                             │
│  Good for: Prototypes, MVPs, side projects                 │
│  Scale: ~100-1,000 concurrent users                        │
│  Cost: $20-100/month                                        │
└─────────────────────────────────────────────────────────────┘
```

### Level 2: Separate Database (1K-10K users)

```
┌─────────────────────────────────────────────────────────────┐
│             LEVEL 2: SEPARATE DATABASE                      │
│                                                             │
│      ┌──────────────────┐      ┌──────────────────┐       │
│      │   APP SERVER     │      │    DATABASE      │       │
│      │  ┌──────────┐   │      │   ┌────────┐    │       │
│      │  │ Web/App  │   │─────▶│   │ MySQL/ │    │       │
│      │  │ Server   │   │      │   │ Postgres│    │       │
│      │  └──────────┘   │      │   └────────┘    │       │
│      └──────────────────┘      └──────────────────┘       │
│                                                             │
│  Benefits:                                                  │
│  ├── Independent scaling                                   │
│  ├── Better resource utilization                          │
│  └── Database can be optimized separately                 │
│                                                             │
│  Scale: ~1,000-10,000 concurrent users                    │
└─────────────────────────────────────────────────────────────┘
```

### Level 3: Add Caching (10K-100K users)

```
┌─────────────────────────────────────────────────────────────┐
│               LEVEL 3: ADD CACHING                          │
│                                                             │
│      ┌──────────────────┐      ┌──────────────────┐       │
│      │   APP SERVER     │      │      CACHE       │       │
│      │  ┌──────────┐   │      │   ┌────────┐    │       │
│      │  │ Web/App  │───┼─────▶│   │ Redis  │    │       │
│      │  │ Server   │   │      │   └────────┘    │       │
│      │  └──────────┘   │      └──────────────────┘       │
│      └────────┬─────────┘              │                   │
│               │                         │ Cache miss       │
│               │                         ▼                   │
│               │              ┌──────────────────┐          │
│               └─────────────▶│    DATABASE      │          │
│                              │   ┌────────┐    │          │
│                              │   │ MySQL  │    │          │
│                              │   └────────┘    │          │
│                              └──────────────────┘          │
│                                                             │
│  Cache Strategy:                                            │
│  1. Check cache first                                       │
│  2. If miss, query database                                │
│  3. Store result in cache                                  │
│  4. Return to user                                          │
│                                                             │
│  Typical cache hit ratio: 80-95%                           │
│  Database load reduction: 5-20x                            │
└─────────────────────────────────────────────────────────────┘
```

### Level 4: Multiple App Servers + Load Balancer (100K-1M users)

```
┌─────────────────────────────────────────────────────────────┐
│         LEVEL 4: LOAD BALANCER + MULTIPLE SERVERS          │
│                                                             │
│                    ┌──────────────┐                        │
│                    │     CDN      │                        │
│                    │ Static files │                        │
│                    └──────┬───────┘                        │
│                           │                                 │
│                    ┌──────▼───────┐                        │
│                    │    LOAD      │                        │
│                    │  BALANCER    │                        │
│                    └──────┬───────┘                        │
│            ┌──────────────┼──────────────┐                 │
│            │              │              │                 │
│      ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼─────┐           │
│      │  Server 1 │ │  Server 2 │ │  Server 3 │           │
│      └─────┬─────┘ └─────┬─────┘ └─────┬─────┘           │
│            │              │              │                 │
│            └──────────────┼──────────────┘                 │
│                           │                                 │
│                    ┌──────▼───────┐                        │
│                    │    CACHE     │                        │
│                    │   (Redis)    │                        │
│                    └──────┬───────┘                        │
│                           │                                 │
│              ┌────────────┼────────────┐                   │
│              │                         │                   │
│       ┌──────▼──────┐          ┌──────▼──────┐            │
│       │   Primary   │─────────▶│   Replica   │            │
│       │     DB      │          │     DB      │            │
│       └─────────────┘          └─────────────┘            │
│                                                             │
│  New components:                                           │
│  ├── CDN: Serve static content globally                   │
│  ├── Load Balancer: Distribute traffic                    │
│  ├── Multiple app servers: Handle more traffic            │
│  └── Read replicas: Handle read load                      │
└─────────────────────────────────────────────────────────────┘
```

### Level 5: Database Sharding (1M-10M+ users)

```
┌─────────────────────────────────────────────────────────────┐
│            LEVEL 5: DATABASE SHARDING                       │
│                                                             │
│                    ┌──────────────┐                        │
│                    │    LOAD      │                        │
│                    │  BALANCER    │                        │
│                    └──────┬───────┘                        │
│                           │                                 │
│      ┌─────────┬──────────┼──────────┬─────────┐          │
│      │         │          │          │         │          │
│  ┌───▼───┐ ┌───▼───┐ ┌───▼───┐ ┌───▼───┐ ┌───▼───┐      │
│  │Server1│ │Server2│ │Server3│ │Server4│ │Server5│      │
│  └───┬───┘ └───┬───┘ └───┬───┘ └───┬───┘ └───┬───┘      │
│      │         │          │          │         │          │
│      └─────────┴──────────┴──────────┴─────────┘          │
│                           │                                 │
│                    ┌──────▼───────┐                        │
│                    │   ROUTING    │                        │
│                    │    LAYER     │                        │
│                    └──────┬───────┘                        │
│            ┌──────────────┼──────────────┐                 │
│            │              │              │                 │
│      ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼─────┐           │
│      │  Shard 1  │ │  Shard 2  │ │  Shard 3  │           │
│      │ Users A-H │ │ Users I-P │ │ Users Q-Z │           │
│      └───────────┘ └───────────┘ └───────────┘           │
│                                                             │
│  Sharding strategies:                                      │
│  ├── Range-based (A-H, I-P, Q-Z)                          │
│  ├── Hash-based (hash(user_id) % num_shards)             │
│  └── Directory-based (lookup table)                       │
└─────────────────────────────────────────────────────────────┘
```

### Level 6: Microservices (10M+ users, multiple teams)

```
┌─────────────────────────────────────────────────────────────┐
│            LEVEL 6: MICROSERVICES                           │
│                                                             │
│                    ┌──────────────┐                        │
│                    │ API GATEWAY  │                        │
│                    └──────┬───────┘                        │
│            ┌──────────────┼──────────────┐                 │
│            │              │              │                 │
│      ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼─────┐           │
│      │   User    │ │   Post    │ │  Timeline │           │
│      │  Service  │ │  Service  │ │  Service  │           │
│      └─────┬─────┘ └─────┬─────┘ └─────┬─────┘           │
│            │              │              │                 │
│      ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼─────┐           │
│      │  User DB  │ │  Post DB  │ │ Timeline  │           │
│      │  (MySQL)  │ │  (Mongo)  │ │  (Cass)   │           │
│      └───────────┘ └───────────┘ └───────────┘           │
│                                                             │
│  Benefits:                                                  │
│  ├── Independent scaling                                   │
│  ├── Technology flexibility                               │
│  ├── Team autonomy                                         │
│  └── Fault isolation                                       │
│                                                             │
│  Costs:                                                     │
│  ├── Operational complexity                               │
│  ├── Network latency                                       │
│  ├── Distributed transactions                             │
│  └── Service discovery                                     │
└─────────────────────────────────────────────────────────────┘
```

## Scaling Patterns

### Pattern 1: Read Replicas

```
┌─────────────────────────────────────────────────────────────┐
│                   READ REPLICAS                             │
│                                                             │
│         WRITES                    READS                     │
│           │                         │                       │
│           ▼                         │                       │
│     ┌──────────┐                   │                       │
│     │ PRIMARY  │                   │                       │
│     │    DB    │                   │                       │
│     └────┬─────┘                   │                       │
│          │                          │                       │
│          │ Replication             │                       │
│          │                          │                       │
│    ┌─────┴─────┬─────────┐        │                       │
│    │           │         │        │                       │
│    ▼           ▼         ▼        ▼                       │
│ ┌──────┐  ┌──────┐  ┌──────┐     │                       │
│ │Replica│  │Replica│  │Replica│◀──┘                       │
│ │  1    │  │  2    │  │  3    │                           │
│ └──────┘  └──────┘  └──────┘                             │
│                                                             │
│  USE WHEN:                                                  │
│  ├── Read-heavy workload (10:1+ ratio)                    │
│  ├── Can tolerate slight replication lag                  │
│  └── Need high read availability                          │
│                                                             │
│  REPLICATION LAG: Usually milliseconds to seconds         │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 2: Write-Through Cache

```
┌─────────────────────────────────────────────────────────────┐
│               WRITE-THROUGH CACHE                           │
│                                                             │
│  WRITE FLOW:                                                │
│  1. Write to cache                                          │
│  2. Cache writes to database                               │
│  3. Return success                                          │
│                                                             │
│        App                                                  │
│         │                                                   │
│         │ Write                                             │
│         ▼                                                   │
│     ┌──────┐                                               │
│     │Cache │                                               │
│     └──┬───┘                                               │
│        │                                                    │
│        │ Write-through                                      │
│        ▼                                                    │
│     ┌──────┐                                               │
│     │  DB  │                                               │
│     └──────┘                                               │
│                                                             │
│  PROS: Cache always consistent                             │
│  CONS: Write latency (two writes)                         │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 3: Write-Behind Cache (Write-Back)

```
┌─────────────────────────────────────────────────────────────┐
│               WRITE-BEHIND CACHE                            │
│                                                             │
│  WRITE FLOW:                                                │
│  1. Write to cache                                          │
│  2. Return success immediately                             │
│  3. Async: Cache writes to database later                  │
│                                                             │
│        App                                                  │
│         │                                                   │
│         │ Write                                             │
│         ▼                                                   │
│     ┌──────┐                                               │
│     │Cache │──────┐                                        │
│     └──────┘      │ Async batch write                     │
│         │         │                                        │
│     Return        ▼                                        │
│     Success   ┌──────┐                                     │
│               │  DB  │                                     │
│               └──────┘                                     │
│                                                             │
│  PROS: Fast writes, batching possible                     │
│  CONS: Data loss risk if cache fails                      │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 4: CQRS (Command Query Responsibility Segregation)

```
┌─────────────────────────────────────────────────────────────┐
│                      CQRS                                   │
│                                                             │
│  COMMANDS (Writes)           QUERIES (Reads)               │
│        │                           │                        │
│        ▼                           ▼                        │
│  ┌──────────┐               ┌──────────┐                   │
│  │ Command  │               │  Query   │                   │
│  │ Handler  │               │ Handler  │                   │
│  └────┬─────┘               └────┬─────┘                   │
│       │                          │                         │
│       ▼                          ▼                         │
│  ┌──────────┐               ┌──────────┐                   │
│  │  Write   │──Sync/Async──▶│  Read    │                   │
│  │   DB     │               │   DB     │                   │
│  │(Optimized│               │(Optimized│                   │
│  │for write)│               │for read) │                   │
│  └──────────┘               └──────────┘                   │
│                                                             │
│  USE WHEN:                                                  │
│  ├── Vastly different read/write patterns                 │
│  ├── Read model different from write model                │
│  └── Need independent scaling                              │
│                                                             │
│  EXAMPLE:                                                   │
│  Write DB: Normalized PostgreSQL                          │
│  Read DB: Denormalized Elasticsearch                      │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 5: Event-Driven Architecture

```
┌─────────────────────────────────────────────────────────────┐
│             EVENT-DRIVEN ARCHITECTURE                       │
│                                                             │
│     Service A          Event Bus           Service B       │
│         │              (Kafka)                 │           │
│         │                │                     │           │
│  User   │  ┌─────────────┼─────────────┐     │           │
│ Created │  │             │             │     │           │
│         ▼  ▼             ▼             ▼     ▼           │
│     ┌──────────────────────────────────────────┐         │
│     │           MESSAGE QUEUE                   │         │
│     │  ┌────────┐ ┌────────┐ ┌────────┐       │         │
│     │  │Event 1 │ │Event 2 │ │Event 3 │       │         │
│     │  └────────┘ └────────┘ └────────┘       │         │
│     └──────────────────────────────────────────┘         │
│              │              │              │              │
│              ▼              ▼              ▼              │
│         ┌────────┐    ┌────────┐    ┌────────┐          │
│         │Consumer│    │Consumer│    │Consumer│          │
│         │   A    │    │   B    │    │   C    │          │
│         │(Email) │    │(Search)│    │(Analyt)│          │
│         └────────┘    └────────┘    └────────┘          │
│                                                             │
│  BENEFITS:                                                  │
│  ├── Loose coupling                                        │
│  ├── Independent scaling                                   │
│  ├── Resilience (retry failed events)                     │
│  └── Audit trail                                           │
└─────────────────────────────────────────────────────────────┘
```

## Scaling Cheat Sheet

```
┌─────────────────────────────────────────────────────────────┐
│              SCALING CHEAT SHEET                            │
│                                                             │
│  PROBLEM                  SOLUTION                          │
│  ────────────────────────────────────────────              │
│  Slow reads               Add caching (Redis)              │
│  Database overloaded      Read replicas                    │
│  Too many connections     Connection pooling               │
│  Single server limit      Horizontal scaling + LB          │
│  Data too large           Database sharding                │
│  Global latency           Multi-region + CDN               │
│  Spike traffic            Auto-scaling + queue            │
│  Write bottleneck         Write sharding, async            │
│  Complex queries slow     CQRS, materialized views        │
│  Service coupling         Event-driven, message queue     │
│                                                             │
│  SCALING SEQUENCE:                                          │
│  1. Optimize queries and indexes                           │
│  2. Add caching                                            │
│  3. Add read replicas                                      │
│  4. Horizontal app scaling                                 │
│  5. Database sharding                                      │
│  6. Microservices (if needed)                             │
└─────────────────────────────────────────────────────────────┘
```

## Common Scaling Mistakes

### Mistake 1: Premature Optimization

```
❌ WRONG:
"Let's start with 50 microservices and sharding"

✅ RIGHT:
"Start simple, scale when we hit bottlenecks"

Rule: Don't optimize for 1M users when you have 1K
```

### Mistake 2: Ignoring the Database

```
❌ WRONG:
"Add more app servers" (when DB is the bottleneck)

✅ RIGHT:
First check: Is the database the bottleneck?
- Add indexes
- Optimize queries
- Add caching
- Then consider read replicas
```

### Mistake 3: Not Testing at Scale

```
❌ WRONG:
"It works on my machine with 10 records"

✅ RIGHT:
Load test with realistic data volumes
- Millions of rows
- Thousands of concurrent users
- Network latency simulation
```

## Interview Tips for Scale

```
1. ALWAYS QUANTIFY
   "This design handles 100K QPS with 5 servers"

2. SHOW PROGRESSION
   "We start here, then scale to X, then to Y"

3. KNOW THE BOTTLENECKS
   "The bottleneck will be database writes, so..."

4. JUSTIFY COMPLEXITY
   "Sharding adds complexity, so we only do it
    when we exceed single-DB capacity"

5. MENTION COSTS
   "More servers = more cost, so we use caching first"
```

---

**Next:** [08_High_Availability.md](./08_High_Availability.md) - Keep your scaled system running
