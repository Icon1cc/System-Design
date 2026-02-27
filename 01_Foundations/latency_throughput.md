# Latency vs Throughput

## The Simple Explanation

Imagine a water pipe:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  LATENCY = How long does ONE drop take to go through?           │
│            (Time from start to finish)                          │
│                                                                 │
│  THROUGHPUT = How many drops can flow through per second?       │
│               (Total volume over time)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

                    LATENCY
            ◄─────────────────────►

    ┌───────────────────────────────────────┐
    │●                                    ● │ ← One drop's journey
    │○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○│
    └───────────────────────────────────────┘

            THROUGHPUT = drops per second
```

### Real-World Analogy

**Highway Traffic:**
- **Latency** = Time for one car to drive from A to B
- **Throughput** = Number of cars that can pass per hour

A highway can have:
- **Low latency, low throughput:** Empty highway, one lane
- **Low latency, high throughput:** Empty highway, ten lanes
- **High latency, high throughput:** Traffic jam, but many cars eventually get through
- **High latency, low throughput:** Traffic jam on a one-lane road (the worst!)

---

## Latency: The Response Time

### What is Latency?

Latency is the time delay between a request and its response.

```
┌─────────────────────────────────────────────────────────────────┐
│                    LATENCY BREAKDOWN                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Request starts ────────────────────────────► Response arrives │
│        │                                              │         │
│        │◄──────────── TOTAL LATENCY ────────────────►│         │
│        │                                              │         │
│        │     ┌────────────────────────────────┐      │         │
│        │     │  Network Transit Time          │      │         │
│        │     │  + Server Processing Time      │      │         │
│        │     │  + Database Query Time         │      │         │
│        │     │  + Network Return Time         │      │         │
│        │     └────────────────────────────────┘      │         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Types of Latency

| Type | Description | Typical Values |
|------|-------------|----------------|
| **Network Latency** | Time for data to travel over network | 1ms - 300ms |
| **Disk Latency** | Time to read/write to storage | 0.1ms - 10ms |
| **Application Latency** | Time for code to execute | Variable |
| **Database Latency** | Time for queries to complete | 1ms - 100ms |

### Critical Latency Numbers

```
┌────────────────────────────────────────────────────────────────┐
│              LATENCY NUMBERS EVERY DEVELOPER SHOULD KNOW       │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  L1 cache reference                           0.5 ns           │
│  Branch mispredict                            5 ns             │
│  L2 cache reference                           7 ns             │
│  Mutex lock/unlock                            25 ns            │
│  Main memory reference                        100 ns           │
│  Compress 1KB with Zippy                      3,000 ns (3 μs)  │
│  Send 1KB over 1 Gbps network                 10,000 ns (10 μs)│
│  Read 4KB randomly from SSD                   150,000 ns       │
│  Read 1 MB sequentially from memory           250,000 ns       │
│  Round trip within same datacenter            500,000 ns       │
│  Read 1 MB sequentially from SSD              1 ms             │
│  Disk seek                                    10 ms            │
│  Read 1 MB sequentially from disk             20 ms            │
│  Send packet CA → Netherlands → CA            150 ms           │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Visualizing Latency Differences

If L1 cache access = 1 second, then:

```
L1 cache reference:        1 second
L2 cache reference:        14 seconds
Main memory reference:     3 minutes, 20 seconds
SSD random read:           3.5 days
HDD seek:                  231 days (7.5 months!)
Cross-continent round trip: 8.5 years
```

This is why caching matters so much!

---

## Throughput: The Bandwidth

### What is Throughput?

Throughput is the amount of work done per unit of time.

```
┌─────────────────────────────────────────────────────────────────┐
│                    THROUGHPUT EXAMPLES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Web Server:      requests/second (RPS)                         │
│  Database:        queries/second (QPS)                          │
│  Network:         bits/second (bps) or bytes/second             │
│  Message Queue:   messages/second                               │
│  API:             transactions/second (TPS)                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Throughput vs. Capacity

```
CAPACITY:    The MAXIMUM throughput possible
THROUGHPUT:  The ACTUAL amount of work being done

Example: Highway
├── Capacity:    5,000 cars/hour (theoretical max)
├── Throughput:  3,000 cars/hour (actual traffic)
└── Utilization: 60% (throughput/capacity)
```

### Factors Affecting Throughput

```
┌─────────────────────────────────────────────────────────────────┐
│                 THROUGHPUT BOTTLENECKS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. CPU                                                         │
│     └── Processing power limits computation speed               │
│                                                                 │
│  2. Memory                                                      │
│     └── RAM limits how much data can be processed               │
│                                                                 │
│  3. Network Bandwidth                                           │
│     └── Pipe size limits data transfer rate                     │
│                                                                 │
│  4. Disk I/O                                                    │
│     └── Storage speed limits read/write operations              │
│                                                                 │
│  5. Database Connections                                        │
│     └── Connection pool limits concurrent queries               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Relationship

### Can You Have Both?

```
┌────────────────────────────────────────────────────────────────┐
│                  LATENCY vs THROUGHPUT                         │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  High Throughput + Low Latency  = IDEAL (but expensive)        │
│  High Throughput + High Latency = Batch Processing             │
│  Low Throughput + Low Latency   = Real-time Systems            │
│  Low Throughput + High Latency  = AVOID                        │
│                                                                │
└────────────────────────────────────────────────────────────────┘

    ▲ Throughput
    │
    │    ┌─────────┐        ┌─────────┐
    │    │  Batch  │        │  IDEAL  │
  H │    │Processing│        │ ($$$$) │
  i │    └─────────┘        └─────────┘
  g │
  h │
    │    ┌─────────┐        ┌─────────┐
  L │    │  AVOID  │        │Real-time│
  o │    │         │        │ Systems │
  w │    └─────────┘        └─────────┘
    │
    └──────────────────────────────────────────►
          High Latency       Low Latency
```

### Little's Law

A fundamental relationship:

```
┌────────────────────────────────────────────────────────────────┐
│                     LITTLE'S LAW                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│        L = λ × W                                               │
│                                                                │
│        L = Average number of items in system                   │
│        λ = Average arrival rate (throughput)                   │
│        W = Average time in system (latency)                    │
│                                                                │
│  Example:                                                      │
│  ├── 100 requests in system                                    │
│  ├── Throughput: 200 requests/second                           │
│  └── Latency: 100/200 = 0.5 seconds = 500ms                    │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## Measuring and Improving

### Latency Percentiles

Average latency can be misleading. Use percentiles:

```
┌────────────────────────────────────────────────────────────────┐
│                   LATENCY PERCENTILES                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Given 100 requests sorted by latency:                         │
│                                                                │
│  p50 (median):  Half the requests are faster than this         │
│  p90:           90% of requests are faster than this           │
│  p95:           95% of requests are faster than this           │
│  p99:           99% of requests are faster than this           │
│  p99.9:         99.9% of requests are faster than this         │
│                                                                │
│  Example Distribution:                                         │
│  ├── p50:    50ms   (typical user experience)                  │
│  ├── p90:   100ms   (most users)                               │
│  ├── p95:   200ms   (almost everyone)                          │
│  ├── p99:   500ms   (edge cases)                               │
│  └── p99.9: 2000ms  (extreme outliers)                         │
│                                                                │
└────────────────────────────────────────────────────────────────┘

     Number of
     Requests
        │
        │ ▓▓▓▓
        │ ▓▓▓▓▓▓
        │ ▓▓▓▓▓▓▓▓
        │ ▓▓▓▓▓▓▓▓▓▓▓
        │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓                    ▓  ▓
        └─────────────────────────────────────────────► Latency
            │       │       │       │         │
          p50      p90     p95     p99      p99.9
```

### Why P99 Matters More Than Average

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  Scenario: E-commerce site with 1 million requests/day         │
│                                                                │
│  Average latency: 100ms (looks great!)                         │
│  P99 latency: 5000ms (1% = 10,000 users wait 5+ seconds!)      │
│                                                                │
│  At high scale, your worst cases affect MANY users.            │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Improving Latency

```
LATENCY REDUCTION STRATEGIES:

1. CACHING
   ├── In-memory caching (Redis, Memcached)
   ├── CDN for static content
   └── Browser caching

2. DATABASE OPTIMIZATION
   ├── Proper indexing
   ├── Query optimization
   └── Connection pooling

3. NETWORK OPTIMIZATION
   ├── Reduce round trips
   ├── Use HTTP/2 multiplexing
   └── Geographic proximity (CDN, edge servers)

4. APPLICATION OPTIMIZATION
   ├── Async processing
   ├── Code optimization
   └── Reduce payload size

5. ARCHITECTURE
   ├── Microservices (parallel processing)
   ├── Event-driven design
   └── Read replicas for read-heavy workloads
```

### Improving Throughput

```
THROUGHPUT IMPROVEMENT STRATEGIES:

1. HORIZONTAL SCALING
   ├── Add more servers
   ├── Load balancing
   └── Stateless services

2. VERTICAL SCALING
   ├── More powerful hardware
   ├── More memory
   └── Faster storage (SSD → NVMe)

3. ASYNC PROCESSING
   ├── Message queues
   ├── Background jobs
   └── Event-driven architecture

4. BATCHING
   ├── Batch database writes
   ├── Batch API calls
   └── Bulk operations

5. OPTIMIZATION
   ├── Connection pooling
   ├── Code profiling
   └── Reduce serialization overhead
```

---

## Trade-offs in System Design

### Latency vs. Consistency

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  STRONG CONSISTENCY                                            │
│  ├── All nodes must agree before responding                    │
│  ├── Higher latency (more coordination)                        │
│  └── Example: Bank transfers                                   │
│                                                                │
│  EVENTUAL CONSISTENCY                                          │
│  ├── Respond immediately, sync later                           │
│  ├── Lower latency                                             │
│  └── Example: Social media likes                               │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Throughput vs. Correctness

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  MORE VALIDATION                                               │
│  ├── More checks before processing                             │
│  ├── Lower throughput                                          │
│  └── Fewer errors                                              │
│                                                                │
│  LESS VALIDATION                                               │
│  ├── Quick processing                                          │
│  ├── Higher throughput                                         │
│  └── More potential errors                                     │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Question 1: "How would you reduce latency for a global web application?"

**Strong Answer:**
```
1. CDN for static content (bring content closer to users)
2. Geographic distribution of servers (reduce network latency)
3. Caching at multiple layers (browser, CDN, application, database)
4. HTTP/2 or HTTP/3 (reduce round trips)
5. Database read replicas in each region
6. Optimize critical rendering path
7. Lazy loading for non-critical content
```

### Question 2: "How would you increase throughput for a messaging system?"

**Strong Answer:**
```
1. Horizontal scaling (more message brokers)
2. Partitioning (parallel processing)
3. Batching messages (reduce overhead)
4. Async writes (don't wait for disk)
5. Compression (reduce network bandwidth)
6. Connection pooling (reduce connection overhead)
7. In-memory processing (avoid disk I/O)
```

### Question 3: "Your API has good average latency but bad p99. What's wrong?"

**Strong Answer:**
```
Likely causes:
1. Garbage collection pauses (in JVM-based systems)
2. Database connection pool exhaustion
3. Lock contention
4. Noisy neighbors (shared infrastructure)
5. Cold cache scenarios
6. Retry storms
7. Tail latency amplification in microservices

How to investigate:
1. Add distributed tracing
2. Profile code paths
3. Monitor GC metrics
4. Check database slow query logs
5. Review connection pool sizes
```

---

## Practical Example: Designing for Latency & Throughput

### Scenario: Design a URL shortener

**Requirements:**
- 100 million URLs created per month
- 10 billion redirects per month
- Low latency for redirects (< 100ms)

**Calculations:**

```
Write throughput:
  100M / (30 * 24 * 3600) ≈ 40 URLs/second

Read throughput:
  10B / (30 * 24 * 3600) ≈ 4,000 redirects/second

Read:Write ratio = 100:1 (read-heavy!)
```

**Design for Low Latency:**

```
┌─────────────────────────────────────────────────────────────────┐
│               URL SHORTENER - LATENCY OPTIMIZED                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User clicks short URL                                          │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                                │
│  │     CDN     │ ← Cache popular redirects at edge              │
│  └──────┬──────┘   (~10ms if cached)                            │
│         │ miss                                                  │
│         ▼                                                       │
│  ┌─────────────┐                                                │
│  │   L7 Load   │                                                │
│  │  Balancer   │                                                │
│  └──────┬──────┘                                                │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                                │
│  │   Server    │                                                │
│  └──────┬──────┘                                                │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                                │
│  │    Redis    │ ← In-memory cache (~1ms)                       │
│  │   Cache     │                                                │
│  └──────┬──────┘                                                │
│         │ miss                                                  │
│         ▼                                                       │
│  ┌─────────────┐                                                │
│  │  Database   │ ← SSD-backed, indexed (~10ms)                  │
│  └─────────────┘                                                │
│                                                                 │
│  Expected latency:                                              │
│  ├── CDN hit: ~10ms                                             │
│  ├── Cache hit: ~20ms                                           │
│  └── Cache miss: ~50ms                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Design for High Throughput:**

```
┌─────────────────────────────────────────────────────────────────┐
│             URL SHORTENER - THROUGHPUT OPTIMIZED                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Multiple Regions for Geographic Distribution                   │
│                                                                 │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                   GLOBAL LOAD BALANCER                 │    │
│  └────────────┬─────────────────┬─────────────┬───────────┘    │
│               │                 │             │                 │
│        ┌──────▼──────┐   ┌──────▼──────┐  ┌──▼──────────┐      │
│        │  US-WEST    │   │  US-EAST    │  │   EU        │      │
│        │  Cluster    │   │  Cluster    │  │  Cluster    │      │
│        └──────┬──────┘   └──────┬──────┘  └──────┬──────┘      │
│               │                 │                │              │
│        ┌──────▼──────┐   ┌──────▼──────┐  ┌──────▼──────┐      │
│        │ 10 Servers  │   │ 10 Servers  │  │ 10 Servers  │      │
│        │ 4K RPS each │   │ 4K RPS each │  │ 4K RPS each │      │
│        └─────────────┘   └─────────────┘  └─────────────┘      │
│                                                                 │
│  Total capacity: 30 servers × 4,000 RPS = 120,000 RPS          │
│  Required: 4,000 RPS                                            │
│  Headroom: 30x for traffic spikes                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Common Pitfalls

### Pitfall 1: Optimizing the wrong thing
Measure first! Don't optimize latency when throughput is the bottleneck.

### Pitfall 2: Ignoring tail latency
P99 matters more than average at scale.

### Pitfall 3: Adding too many network hops
Each hop adds latency. Microservices can create "latency multiplication."

### Pitfall 4: Not considering geographic latency
Speed of light is a hard limit. Data can't travel faster than ~100ms round-trip intercontinental.

---

## Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│                    REMEMBER THIS                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. LATENCY = How fast (time for single operation)             │
│     THROUGHPUT = How much (operations per unit time)           │
│                                                                │
│  2. You often can't maximize both—understand the trade-off     │
│                                                                │
│  3. P99 latency matters more than average at scale             │
│                                                                │
│  4. Little's Law: L = λ × W                                    │
│                                                                │
│  5. Know your latency numbers (memory vs disk vs network)      │
│                                                                │
│  6. Caching is the #1 way to reduce latency                    │
│                                                                │
│  7. Horizontal scaling is the #1 way to increase throughput    │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

Now that you understand latency and throughput, let's learn about availability and consistency—two properties that are often in tension.

**Next:** [Availability vs Consistency →](availability_consistency.md)
