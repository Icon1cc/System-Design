# System Design Foundations

## Welcome to the Foundation

This section covers the **fundamental concepts** you MUST understand before tackling system design interviews. These are the building blocks that every large-scale system is built upon.

```
┌─────────────────────────────────────────────────────────────┐
│                 WHY FOUNDATIONS MATTER                      │
│                                                             │
│   Without foundations, you'll be:                          │
│   ❌ Memorizing solutions without understanding            │
│   ❌ Unable to adapt when requirements change              │
│   ❌ Lost when interviewers ask "why?"                     │
│                                                             │
│   With foundations, you'll be:                             │
│   ✅ Making informed tradeoff decisions                    │
│   ✅ Explaining your reasoning clearly                     │
│   ✅ Adapting designs to new problems                      │
│   ✅ Confident in technical discussions                    │
└─────────────────────────────────────────────────────────────┘
```

## What You'll Learn

### Core Concepts
| File | Topic | Key Question It Answers |
|------|-------|------------------------|
| [what_is_system_design.md](./what_is_system_design.md) | What is System Design | "What am I actually learning?" |
| [how_internet_works.md](./how_internet_works.md) | How the Internet Works | "How do requests travel?" |
| [latency_throughput.md](./latency_throughput.md) | Latency vs Throughput | "What makes a system fast?" |
| [availability_consistency.md](./availability_consistency.md) | Availability vs Consistency | "Can we have both?" |
| [cap_theorem.md](./cap_theorem.md) | CAP Theorem | "What are the fundamental tradeoffs?" |

### Scaling & Performance
| File | Topic | Key Question It Answers |
|------|-------|------------------------|
| [scaling.md](./scaling.md) | Vertical vs Horizontal Scaling | "How do we handle more users?" |
| [load_balancing.md](./load_balancing.md) | Load Balancing | "How do we distribute traffic?" |
| [caching.md](./caching.md) | Caching Fundamentals | "How do we make reads fast?" |

### Data & Storage
| File | Topic | Key Question It Answers |
|------|-------|------------------------|
| [storage_types.md](./storage_types.md) | Storage Types | "Where do we put data?" |
| [sql_vs_nosql.md](./sql_vs_nosql.md) | SQL vs NoSQL | "Which database should I use?" |
| [indexing.md](./indexing.md) | Database Indexing | "How do we make queries fast?" |
| [consistent_hashing.md](./consistent_hashing.md) | Consistent Hashing | "How do we distribute data evenly?" |

## The Foundation Map

```
┌─────────────────────────────────────────────────────────────┐
│                  FOUNDATIONS DEPENDENCY TREE                │
│                                                             │
│                   What is System Design                     │
│                           │                                 │
│              ┌────────────┼────────────┐                   │
│              │            │            │                   │
│              ▼            ▼            ▼                   │
│         Internet      Latency &    Availability            │
│          Basics       Throughput   & Consistency           │
│              │            │            │                   │
│              │            │            ▼                   │
│              │            │       CAP Theorem              │
│              │            │            │                   │
│              └────────────┼────────────┘                   │
│                           │                                 │
│              ┌────────────┼────────────┐                   │
│              │            │            │                   │
│              ▼            ▼            ▼                   │
│          Scaling     Caching      Databases                │
│              │            │       (SQL/NoSQL)              │
│              │            │            │                   │
│              ▼            │            ▼                   │
│      Load Balancing       │       Indexing                 │
│              │            │            │                   │
│              └────────────┴────────────┘                   │
│                           │                                 │
│                           ▼                                 │
│                  Consistent Hashing                        │
│                (ties everything together)                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Recommended Study Order

### Week 1: Core Concepts (Days 1-4)

```
DAY 1: The Basics
├── Read: what_is_system_design.md
├── Read: how_internet_works.md
└── Practice: Trace what happens when you type google.com

DAY 2: Performance Fundamentals
├── Read: latency_throughput.md
├── Memorize: Key latency numbers
└── Practice: Calculate throughput for sample scenarios

DAY 3: The Big Tradeoff
├── Read: availability_consistency.md
├── Read: cap_theorem.md
└── Practice: Identify CA, CP, AP for common systems

DAY 4: Review & Reinforce
├── Re-read anything unclear
├── Draw the concepts from memory
└── Explain to someone (or rubber duck)
```

### Week 1: Building Blocks (Days 5-7)

```
DAY 5: Scaling
├── Read: scaling.md
├── Read: load_balancing.md
└── Practice: Design scaling strategy for 10K → 1M users

DAY 6: Data
├── Read: storage_types.md
├── Read: sql_vs_nosql.md
└── Practice: Choose database for 5 different scenarios

DAY 7: Performance Optimization
├── Read: caching.md
├── Read: indexing.md
└── Practice: Identify caching opportunities in a system
```

### Week 2: Advanced Foundations

```
DAY 8-9: Consistent Hashing
├── Read: consistent_hashing.md (multiple times!)
├── Implement: Simple consistent hashing in code
└── Practice: Walk through node addition/removal scenarios

DAY 10: Integration
├── Review all topics
├── Connect concepts together
└── Practice: Explain how concepts relate to each other
```

## Key Numbers to Memorize

```
┌─────────────────────────────────────────────────────────────┐
│               NUMBERS EVERY ENGINEER SHOULD KNOW            │
│                                                             │
│  LATENCY:                                                   │
│  ├── L1 cache reference: 0.5 ns                            │
│  ├── RAM reference: 100 ns                                 │
│  ├── SSD read: 150 μs                                      │
│  ├── HDD seek: 10 ms                                       │
│  ├── Same datacenter round trip: 0.5 ms                   │
│  └── Cross-continent round trip: 150 ms                   │
│                                                             │
│  STORAGE:                                                   │
│  ├── 1 character: 1 byte                                   │
│  ├── 1 tweet: ~500 bytes                                   │
│  ├── 1 image: 200 KB - 2 MB                               │
│  ├── 1 minute video (HD): 100-500 MB                      │
│                                                             │
│  TIME:                                                      │
│  ├── Seconds in a day: 86,400 ≈ 100,000                   │
│  ├── Seconds in a year: ~31 million                       │
│                                                             │
│  AVAILABILITY:                                              │
│  ├── 99% = 3.65 days downtime/year                        │
│  ├── 99.9% = 8.76 hours downtime/year                     │
│  ├── 99.99% = 52.6 minutes downtime/year                  │
│  └── 99.999% = 5.26 minutes downtime/year                 │
└─────────────────────────────────────────────────────────────┘
```

## Quick Concept Reference

### Latency vs Throughput
```
LATENCY: How long for ONE request (milliseconds)
THROUGHPUT: How many requests per second (QPS)

High throughput ≠ Low latency!
Example: Batch processing has high throughput but high latency
```

### CAP Theorem in 30 Seconds
```
In a distributed system, during a network partition:
├── CP: Choose Consistency (reject requests)
├── AP: Choose Availability (serve stale data)
└── You CANNOT have both!

Normal operation (no partition): You can have C + A
```

### When to Use SQL vs NoSQL
```
SQL (PostgreSQL, MySQL):
├── Complex queries with JOINs
├── ACID transactions needed
├── Data integrity critical
└── Structured, relational data

NoSQL (MongoDB, Cassandra, Redis):
├── Flexible/changing schema
├── Horizontal scaling priority
├── High write throughput
└── Simple query patterns
```

### Caching Rules of Thumb
```
Cache when:
├── Data is read much more than written (10:1+)
├── Same data requested repeatedly
├── Computation is expensive
└── Source is slow (database, API)

Don't cache when:
├── Data changes frequently
├── Each request is unique
├── Consistency is critical
└── Data is too large
```

## Self-Assessment Checklist

Before moving to the next section, make sure you can:

```
□ Explain what happens when you type a URL (DNS, TCP, HTTP)
□ Define latency and throughput with examples
□ Explain CAP theorem and give real-world examples
□ Compare vertical vs horizontal scaling
□ Explain how load balancers work and name 3 algorithms
□ Describe cache-aside pattern and cache invalidation strategies
□ Compare block, file, and object storage with use cases
□ Explain when to use SQL vs NoSQL with specific examples
□ Describe how database indexes work (B-tree basics)
□ Walk through consistent hashing with node addition/removal
```

## Common Mistakes to Avoid

```
┌─────────────────────────────────────────────────────────────┐
│              FOUNDATION PITFALLS                            │
│                                                             │
│  ❌ MISTAKE: Skipping foundations to learn "cool" systems  │
│  ✅ FIX: Foundations make everything else easier           │
│                                                             │
│  ❌ MISTAKE: Memorizing without understanding              │
│  ✅ FIX: Ask "why?" for every concept                      │
│                                                             │
│  ❌ MISTAKE: Reading without practicing                    │
│  ✅ FIX: Draw diagrams, calculate numbers, explain aloud   │
│                                                             │
│  ❌ MISTAKE: Thinking CAP means "pick 2 of 3"              │
│  ✅ FIX: It's about what happens DURING partitions        │
│                                                             │
│  ❌ MISTAKE: "NoSQL is always faster than SQL"            │
│  ✅ FIX: Each has strengths; choose based on use case     │
└─────────────────────────────────────────────────────────────┘
```

## Interview Connection

Every system design interview uses these foundations:

```
"Design Twitter"
├── Latency: Timeline must load fast
├── Throughput: Handle millions of tweets
├── Availability: Must always work
├── Caching: Hot tweets, user timelines
├── SQL: User data, relationships
├── NoSQL: Timeline storage
└── Consistent Hashing: Shard tweets

"Design URL Shortener"
├── Throughput: Millions of redirects
├── Storage: Billions of URLs
├── Caching: Popular URLs
├── Database: SQL or NoSQL?
└── Consistent Hashing: Distribute data
```

## Ready to Start?

Begin with [what_is_system_design.md](./what_is_system_design.md) and work through each file in order. Take your time - these concepts are the foundation for everything else!

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   "Give me six hours to chop down a tree and I will       │
│    spend the first four sharpening the axe."              │
│                                        - Abraham Lincoln   │
│                                                             │
│   These foundations ARE your axe. Sharpen them well.      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

**Next Section:** [02_High_Level_Design](../02_High_Level_Design/README.md) - Apply these foundations to design complete systems
