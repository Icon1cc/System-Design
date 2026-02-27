# Bottleneck Detection and Resolution

## What is a Bottleneck?

A bottleneck is the **constraint that limits your system's overall throughput**. Like the narrowest part of a bottle that limits how fast liquid flows out.

```
┌─────────────────────────────────────────────────────────────┐
│                 THE BOTTLENECK PRINCIPLE                    │
│                                                             │
│       ──────────────┐                                       │
│                     │                                       │
│       Wide pipe     │                                       │
│       (Fast)        │                                       │
│                     │                                       │
│       ──────────────┤                                       │
│                     ├─── Narrow section (BOTTLENECK)       │
│       ──────────────┤     Everything waits here            │
│                     │                                       │
│       Wide pipe     │                                       │
│       (Fast)        │                                       │
│                     │                                       │
│       ──────────────┘                                       │
│                                                             │
│  KEY INSIGHT:                                               │
│  System throughput = Bottleneck throughput                 │
│  Optimizing non-bottlenecks has ZERO impact               │
└─────────────────────────────────────────────────────────────┘
```

## Common Bottleneck Locations

```
┌─────────────────────────────────────────────────────────────┐
│            TYPICAL BOTTLENECK LOCATIONS                     │
│                                                             │
│                                                             │
│  1. DATABASE                                                │
│     └── Most common bottleneck                             │
│     └── CPU, connections, disk I/O, locks                 │
│                                                             │
│  2. APPLICATION SERVER                                      │
│     └── CPU-intensive operations                           │
│     └── Memory pressure                                    │
│     └── Thread pool exhaustion                            │
│                                                             │
│  3. NETWORK                                                 │
│     └── Bandwidth limits                                   │
│     └── Latency (especially cross-region)                 │
│     └── Connection limits                                  │
│                                                             │
│  4. EXTERNAL SERVICES                                       │
│     └── Third-party API rate limits                       │
│     └── Slow dependencies                                  │
│                                                             │
│  5. LOAD BALANCER                                           │
│     └── Connection limits                                  │
│     └── SSL termination overhead                          │
│                                                             │
│  6. CACHE                                                   │
│     └── Memory limits                                      │
│     └── Connection limits                                  │
│     └── Serialization overhead                            │
└─────────────────────────────────────────────────────────────┘
```

## Identifying Bottlenecks

### The Four Golden Signals

```
┌─────────────────────────────────────────────────────────────┐
│             THE FOUR GOLDEN SIGNALS                         │
│                                                             │
│  1. LATENCY                                                 │
│     Time to serve a request                                │
│     Track: P50, P95, P99 percentiles                      │
│     Alert: P99 > 500ms                                    │
│                                                             │
│  2. TRAFFIC                                                 │
│     Demand on your system                                  │
│     Track: Requests/second, QPS                           │
│     Context for other metrics                             │
│                                                             │
│  3. ERRORS                                                  │
│     Rate of failed requests                               │
│     Track: 5xx errors, timeouts, exceptions              │
│     Alert: Error rate > 1%                                │
│                                                             │
│  4. SATURATION                                              │
│     How "full" your system is                             │
│     Track: CPU %, Memory %, Disk I/O, Queue depth        │
│     Alert: Sustained > 80%                                │
│                                                             │
│  CORRELATION:                                               │
│  High saturation + High latency = Found the bottleneck!  │
└─────────────────────────────────────────────────────────────┘
```

### Resource Utilization Analysis

```
┌─────────────────────────────────────────────────────────────┐
│           RESOURCE SATURATION CHECKLIST                     │
│                                                             │
│  COMPONENT        METRICS TO CHECK                         │
│  ─────────────────────────────────────────────             │
│  CPU              User %, System %, Load average           │
│  Memory           Used %, Swap usage, OOM kills           │
│  Disk             Read/Write IOPS, Latency, Queue depth   │
│  Network          Bandwidth %, Packet drops, Errors       │
│  Database         Connections, Lock waits, Query time     │
│  Application      Thread pool usage, GC pauses            │
│                                                             │
│  INVESTIGATION FLOW:                                        │
│                                                             │
│  Is latency high? ──▶ Where is time spent?               │
│        │                                                   │
│        ├── App server? ──▶ Check CPU, memory, threads    │
│        │                                                   │
│        ├── Database? ──▶ Check query times, connections  │
│        │                                                   │
│        ├── Cache? ──▶ Check hit rate, latency           │
│        │                                                   │
│        └── Network? ──▶ Check bandwidth, latency        │
└─────────────────────────────────────────────────────────────┘
```

### Request Tracing

```
┌─────────────────────────────────────────────────────────────┐
│              DISTRIBUTED TRACING                            │
│                                                             │
│  Track a request through all services                      │
│                                                             │
│  Request: GET /api/timeline                                │
│  Total: 450ms                                               │
│                                                             │
│  ├─ Load Balancer: 2ms                                    │
│  │                                                         │
│  ├─ API Gateway: 5ms                                      │
│  │                                                         │
│  ├─ Auth Service: 25ms                                    │
│  │   └─ Redis lookup: 2ms                                 │
│  │   └─ JWT validation: 20ms                              │
│  │                                                         │
│  ├─ Timeline Service: 400ms  ◀─── BOTTLENECK!            │
│  │   └─ Cache lookup: 1ms (miss)                         │
│  │   └─ Database query: 380ms  ◀─── ROOT CAUSE          │
│  │   └─ Response assembly: 15ms                          │
│  │                                                         │
│  └─ Response: 3ms                                         │
│                                                             │
│  TOOLS: Jaeger, Zipkin, AWS X-Ray, Datadog APM          │
└─────────────────────────────────────────────────────────────┘
```

## Common Bottlenecks and Solutions

### Bottleneck 1: Database

```
┌─────────────────────────────────────────────────────────────┐
│           DATABASE BOTTLENECKS                              │
│                                                             │
│  SYMPTOMS:                                                  │
│  - High query latency                                      │
│  - Connection pool exhausted                               │
│  - Lock wait timeouts                                      │
│  - High CPU on database server                            │
│                                                             │
│  DIAGNOSIS:                                                 │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ -- Find slow queries                                 │ │
│  │ SELECT * FROM pg_stat_statements                     │ │
│  │ ORDER BY total_time DESC LIMIT 10;                   │ │
│  │                                                       │ │
│  │ -- Check for missing indexes                         │ │
│  │ EXPLAIN ANALYZE SELECT ...                           │ │
│  │                                                       │ │
│  │ -- Check lock contention                             │ │
│  │ SELECT * FROM pg_locks WHERE NOT granted;           │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  SOLUTIONS:                                                 │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ QUICK WINS:                                          │  │
│  │ ├── Add missing indexes                             │  │
│  │ ├── Optimize slow queries                           │  │
│  │ ├── Increase connection pool                        │  │
│  │ └── Add query caching (Redis)                       │  │
│  │                                                       │  │
│  │ MEDIUM EFFORT:                                        │  │
│  │ ├── Add read replicas                                │  │
│  │ ├── Denormalize hot paths                           │  │
│  │ └── Archive old data                                │  │
│  │                                                       │  │
│  │ MAJOR CHANGES:                                        │  │
│  │ ├── Database sharding                                │  │
│  │ ├── Switch to different database                    │  │
│  │ └── CQRS (separate read/write stores)              │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Bottleneck 2: Application Server

```
┌─────────────────────────────────────────────────────────────┐
│           APPLICATION SERVER BOTTLENECKS                    │
│                                                             │
│  SYMPTOMS:                                                  │
│  - High CPU utilization                                    │
│  - Long GC pauses                                          │
│  - Thread pool exhausted                                   │
│  - Out of memory errors                                    │
│                                                             │
│  DIAGNOSIS:                                                 │
│  - CPU profiling (async-profiler, perf)                   │
│  - Memory profiling (heap dumps)                          │
│  - Thread dumps (jstack, kill -3)                        │
│                                                             │
│  COMMON CAUSES:                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ CPU:                                                  │  │
│  │ ├── JSON serialization (use faster library)         │  │
│  │ ├── Regex compilation (pre-compile patterns)        │  │
│  │ ├── Logging overhead (async logging)               │  │
│  │ └── Inefficient algorithms                          │  │
│  │                                                       │  │
│  │ MEMORY:                                               │  │
│  │ ├── Memory leaks                                     │  │
│  │ ├── Large object creation in loops                  │  │
│  │ ├── Unbounded caches                                │  │
│  │ └── String concatenation in loops                   │  │
│  │                                                       │  │
│  │ THREADS:                                              │  │
│  │ ├── Blocking I/O on main threads                    │  │
│  │ ├── Deadlocks                                        │  │
│  │ └── Too few threads for workload                    │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  SOLUTIONS:                                                 │
│  - Horizontal scaling (more instances)                    │
│  - Code optimization (profile first!)                     │
│  - Async processing for slow operations                   │
│  - Increase thread pool sizes                             │
│  - JVM tuning (heap size, GC algorithm)                  │
└─────────────────────────────────────────────────────────────┘
```

### Bottleneck 3: Cache

```
┌─────────────────────────────────────────────────────────────┐
│              CACHE BOTTLENECKS                              │
│                                                             │
│  SYMPTOMS:                                                  │
│  - Low cache hit rate (< 80%)                             │
│  - Cache evictions                                         │
│  - High cache latency                                      │
│  - Cache connection errors                                 │
│                                                             │
│  DIAGNOSIS:                                                 │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ -- Redis diagnostics                                 │ │
│  │ INFO stats          -- Hit/miss ratio               │ │
│  │ INFO memory         -- Memory usage                 │ │
│  │ INFO clients        -- Connection count             │ │
│  │ SLOWLOG GET 10      -- Slow operations              │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  LOW HIT RATE CAUSES:                                       │
│  ├── TTL too short                                        │
│  ├── Cache size too small                                 │
│  ├── Wrong cache key design                               │
│  └── Cache not being populated                            │
│                                                             │
│  SOLUTIONS:                                                 │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ HIT RATE:                                             │  │
│  │ ├── Increase TTL                                     │  │
│  │ ├── Warm cache on deploy                            │  │
│  │ ├── Pre-compute popular queries                     │  │
│  │                                                       │  │
│  │ CAPACITY:                                             │  │
│  │ ├── Increase memory                                  │  │
│  │ ├── Use Redis Cluster                               │  │
│  │ ├── Compress cached values                          │  │
│  │                                                       │  │
│  │ PERFORMANCE:                                          │  │
│  │ ├── Use pipelining for batch operations            │  │
│  │ ├── Reduce value sizes                              │  │
│  │ └── Local cache for hot keys                       │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Bottleneck 4: Network

```
┌─────────────────────────────────────────────────────────────┐
│              NETWORK BOTTLENECKS                            │
│                                                             │
│  SYMPTOMS:                                                  │
│  - High latency for all requests                          │
│  - Timeouts                                                │
│  - Connection refused errors                               │
│  - Packet loss                                             │
│                                                             │
│  DIAGNOSIS:                                                 │
│  - traceroute / mtr to identify slow hops                │
│  - iftop / nethogs to see bandwidth usage                │
│  - netstat / ss for connection counts                     │
│                                                             │
│  COMMON CAUSES:                                             │
│  ├── Bandwidth saturation                                 │
│  ├── Too many small requests (chattiness)                │
│  ├── Cross-region communication                           │
│  ├── DNS resolution delays                                │
│  └── SSL/TLS handshake overhead                          │
│                                                             │
│  SOLUTIONS:                                                 │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ LATENCY:                                              │  │
│  │ ├── Move services closer together                   │  │
│  │ ├── Use connection pooling                          │  │
│  │ ├── Enable HTTP keep-alive                          │  │
│  │ └── Pre-resolve DNS                                  │  │
│  │                                                       │  │
│  │ BANDWIDTH:                                            │  │
│  │ ├── Compress responses (gzip)                       │  │
│  │ ├── Reduce payload sizes                            │  │
│  │ ├── Use CDN for static content                      │  │
│  │ └── Batch API calls                                  │  │
│  │                                                       │  │
│  │ CONNECTIONS:                                          │  │
│  │ ├── Connection pooling                               │  │
│  │ ├── HTTP/2 multiplexing                              │  │
│  │ └── Increase file descriptor limits                 │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Bottleneck 5: External Dependencies

```
┌─────────────────────────────────────────────────────────────┐
│           EXTERNAL DEPENDENCY BOTTLENECKS                   │
│                                                             │
│  SYMPTOMS:                                                  │
│  - Inconsistent latency                                    │
│  - Timeout errors to specific services                    │
│  - Rate limit errors (429)                                │
│                                                             │
│  COMMON CULPRITS:                                           │
│  ├── Payment processors                                   │
│  ├── Email services                                       │
│  ├── SMS gateways                                         │
│  ├── Third-party APIs                                     │
│  └── Cloud services (S3, etc.)                           │
│                                                             │
│  SOLUTIONS:                                                 │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ RESILIENCE:                                           │  │
│  │ ├── Circuit breakers                                 │  │
│  │ ├── Timeouts and retries                            │  │
│  │ ├── Fallback responses                              │  │
│  │ └── Bulkhead isolation                              │  │
│  │                                                       │  │
│  │ PERFORMANCE:                                          │  │
│  │ ├── Cache external API responses                    │  │
│  │ ├── Batch requests where possible                   │  │
│  │ ├── Async processing (queue + workers)             │  │
│  │ └── Use webhooks instead of polling                │  │
│  │                                                       │  │
│  │ RATE LIMITS:                                          │  │
│  │ ├── Implement client-side rate limiting             │  │
│  │ ├── Queue and process at allowed rate              │  │
│  │ └── Negotiate higher limits                         │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Bottleneck Analysis Framework

### Step-by-Step Process

```
┌─────────────────────────────────────────────────────────────┐
│           BOTTLENECK ANALYSIS PROCESS                       │
│                                                             │
│  1. OBSERVE                                                 │
│     └── Collect metrics (latency, throughput, errors)     │
│     └── Identify anomalies                                 │
│                                                             │
│  2. HYPOTHESIZE                                             │
│     └── Based on symptoms, guess the bottleneck           │
│     └── Common order: DB → App → Cache → Network         │
│                                                             │
│  3. MEASURE                                                 │
│     └── Validate hypothesis with specific metrics         │
│     └── Use tracing to pinpoint exact location           │
│                                                             │
│  4. OPTIMIZE                                                │
│     └── Fix the identified bottleneck                     │
│     └── Start with quick wins                             │
│                                                             │
│  5. REPEAT                                                  │
│     └── New bottleneck will emerge                        │
│     └── Continue until performance goals met              │
│                                                             │
│  EXAMPLE:                                                   │
│  1. Observe: P99 latency is 2 seconds                     │
│  2. Hypothesize: Database is slow                         │
│  3. Measure: DB query takes 1.8s (confirmed!)            │
│  4. Optimize: Add index, query now 50ms                  │
│  5. Repeat: New P99 is 400ms, check next layer          │
└─────────────────────────────────────────────────────────────┘
```

### Amdahl's Law

```
┌─────────────────────────────────────────────────────────────┐
│                 AMDAHL'S LAW                                │
│                                                             │
│  "The speedup from optimizing a component is limited       │
│   by how much time is spent in that component"            │
│                                                             │
│  FORMULA:                                                   │
│  Speedup = 1 / ((1 - P) + P/S)                            │
│                                                             │
│  P = Fraction of time in optimized component              │
│  S = Speedup of that component                            │
│                                                             │
│  EXAMPLE:                                                   │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Current: 1000ms total                                │  │
│  │ - Database: 800ms (80%)                             │  │
│  │ - Everything else: 200ms (20%)                      │  │
│  │                                                       │  │
│  │ If we make DB 10x faster:                           │  │
│  │ New DB time: 80ms                                    │  │
│  │ Total: 80ms + 200ms = 280ms                         │  │
│  │ Speedup: 1000/280 = 3.6x                            │  │
│  │                                                       │  │
│  │ INSIGHT: Even infinite DB speedup = max 5x overall  │  │
│  │ (because 20% is NOT in DB)                          │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  KEY TAKEAWAY:                                              │
│  Focus on the BIGGEST component first                     │
│  Small components don't matter much                       │
└─────────────────────────────────────────────────────────────┘
```

## System Design Interview: Bottleneck Discussion

### How to Talk About Bottlenecks

```
┌─────────────────────────────────────────────────────────────┐
│       INTERVIEW BOTTLENECK DISCUSSION                       │
│                                                             │
│  PROACTIVE IDENTIFICATION:                                  │
│  "Let me identify potential bottlenecks in this design..." │
│                                                             │
│  STRUCTURED ANALYSIS:                                       │
│  "At 100K QPS, the bottleneck will likely be:             │
│   1. Database - single MySQL can't handle this load       │
│   2. Solution: Add read replicas and caching              │
│                                                             │
│   At 1M QPS, new bottleneck:                              │
│   1. Cache - Redis cluster needed                         │
│   2. Load balancer - may need multiple LBs                │
│   3. Solution: Shard database, scale cache cluster"       │
│                                                             │
│  TRADEOFF AWARENESS:                                        │
│  "Each solution introduces complexity:                     │
│   - Read replicas add replication lag                     │
│   - Caching requires invalidation strategy                │
│   - Sharding complicates queries"                         │
└─────────────────────────────────────────────────────────────┘
```

### Common Interview Bottlenecks by System

```
┌─────────────────────────────────────────────────────────────┐
│       BOTTLENECKS BY SYSTEM TYPE                            │
│                                                             │
│  TWITTER:                                                   │
│  - Home timeline generation (fan-out)                     │
│  - Celebrity followers (hot partition)                    │
│  - Search indexing at scale                               │
│                                                             │
│  URL SHORTENER:                                             │
│  - Database for high read QPS                             │
│  - ID generation (uniqueness at scale)                    │
│                                                             │
│  CHAT SYSTEM:                                               │
│  - WebSocket connections (millions)                       │
│  - Message delivery (exactly-once)                        │
│  - Presence system (who's online)                         │
│                                                             │
│  VIDEO STREAMING:                                           │
│  - Storage (petabytes)                                    │
│  - Bandwidth (CDN costs)                                  │
│  - Encoding pipeline                                       │
│                                                             │
│  RIDE SHARING:                                              │
│  - Location updates (high write)                          │
│  - Matching algorithm (compute)                           │
│  - Geospatial queries                                     │
└─────────────────────────────────────────────────────────────┘
```

## Quick Reference

```
┌─────────────────────────────────────────────────────────────┐
│           BOTTLENECK QUICK REFERENCE                        │
│                                                             │
│  SYMPTOM              LIKELY BOTTLENECK    FIRST FIX       │
│  ─────────────────────────────────────────────────────     │
│  High DB latency      Missing indexes      Add indexes     │
│  Connection errors    Pool exhausted       Increase pool   │
│  High CPU             Inefficient code     Profile + fix   │
│  OOM errors           Memory leak          Heap analysis   │
│  Cache misses         TTL/size wrong       Tune cache      │
│  Timeout errors       Slow dependency      Add timeout     │
│  High latency all     Network             Check bandwidth  │
│  Lock timeouts        Contention          Optimize txns    │
│                                                             │
│  OPTIMIZATION PRIORITY:                                     │
│  1. Measure first (don't guess!)                          │
│  2. Fix the biggest component                             │
│  3. Quick wins before big changes                         │
│  4. One change at a time                                  │
│  5. Verify improvement                                     │
└─────────────────────────────────────────────────────────────┘
```

---

**Next:** [12_HLD_Example_Twitter.md](./12_HLD_Example_Twitter.md) - Complete Twitter system design
