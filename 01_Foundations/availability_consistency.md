# Availability vs Consistency

## The Simple Explanation

Imagine you have a bank account with $100. You make a withdrawal at an ATM while your partner checks the balance on their phone at the exact same moment.

**Two things we want:**
1. **Consistency:** Both of you see the same balance
2. **Availability:** Both transactions work without waiting

The challenge? **You often can't have both perfectly at the same time.**

```
┌─────────────────────────────────────────────────────────────────┐
│                    THE FUNDAMENTAL TENSION                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   CONSISTENCY                    AVAILABILITY                   │
│   ───────────                    ────────────                   │
│   All users see the              The system is                  │
│   same data at the               always responsive              │
│   same time                      (never says "please wait")     │
│                                                                 │
│              ◄────── TRADE-OFF ──────►                          │
│                                                                 │
│   To guarantee everyone          To guarantee instant           │
│   sees the same data,            responses, we might            │
│   we might need to               serve slightly stale           │
│   make some users wait           data                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Availability: The System Responds

### What is Availability?

Availability measures what percentage of time a system is operational and accessible.

```
                    Uptime
Availability = ─────────────────────
               Uptime + Downtime
```

### The Nines

Availability is often expressed in "nines":

```
┌────────────────────────────────────────────────────────────────┐
│                   THE NINES OF AVAILABILITY                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Availability    Downtime/Year    Downtime/Month  Downtime/Day │
│  ────────────    ─────────────    ──────────────  ────────────│
│  90%     (1 nine)    36.5 days        3 days       2.4 hours  │
│  99%     (2 nines)   3.65 days       7.3 hours    14.4 minutes│
│  99.9%   (3 nines)   8.76 hours      43.8 minutes  1.44 min   │
│  99.99%  (4 nines)   52.6 minutes    4.38 minutes  8.6 seconds│
│  99.999% (5 nines)   5.26 minutes    26.3 seconds  0.86 sec   │
│                                                                │
└────────────────────────────────────────────────────────────────┘

Industry Standards:
├── Typical web app: 99.9% (3 nines)
├── Cloud providers (AWS/GCP): 99.95% - 99.99%
├── Payment systems: 99.99%+ (4+ nines)
└── Life-critical systems: 99.999%+ (5+ nines)
```

### How to Achieve High Availability

```
┌─────────────────────────────────────────────────────────────────┐
│               HIGH AVAILABILITY STRATEGIES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. REDUNDANCY                                                  │
│     ├── Multiple servers (no single point of failure)          │
│     ├── Multiple data centers                                  │
│     └── Multiple cloud providers (for critical systems)        │
│                                                                 │
│  2. FAILOVER                                                    │
│     ├── Automatic detection of failures                        │
│     ├── Traffic rerouting to healthy nodes                     │
│     └── Quick recovery procedures                              │
│                                                                 │
│  3. LOAD BALANCING                                              │
│     ├── Distribute traffic across servers                      │
│     ├── Health checks remove unhealthy servers                 │
│     └── Prevents overload on any single server                 │
│                                                                 │
│  4. REPLICATION                                                 │
│     ├── Data copied to multiple locations                      │
│     ├── If one copy fails, others are available                │
│     └── Enables geographic distribution                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Availability Architecture

```
                    SINGLE POINT OF FAILURE (BAD)
                    ─────────────────────────────

                         ┌──────────┐
              Users ────►│  Server  │────► Database
                         └──────────┘
                              │
                        If this dies,
                        EVERYTHING dies


                    HIGH AVAILABILITY (GOOD)
                    ─────────────────────────

                         ┌──────────────┐
              Users ────►│Load Balancer │
                         │  (Primary)   │
                         └──────┬───────┘
                                │
                    ┌───────────┼───────────┐
                    │           │           │
                    ▼           ▼           ▼
              ┌──────────┐ ┌──────────┐ ┌──────────┐
              │ Server 1 │ │ Server 2 │ │ Server 3 │
              └────┬─────┘ └────┬─────┘ └────┬─────┘
                   │            │            │
                   └────────────┼────────────┘
                                │
                    ┌───────────┼───────────┐
                    │           │           │
                    ▼           ▼           ▼
              ┌──────────┐ ┌──────────┐ ┌──────────┐
              │   DB     │ │   DB     │ │   DB     │
              │ Primary  │ │ Replica  │ │ Replica  │
              └──────────┘ └──────────┘ └──────────┘

              Any single component can fail
              and the system keeps running!
```

---

## Consistency: Everyone Sees the Same Data

### What is Consistency?

Consistency ensures that all nodes in a distributed system have the same data at any given time.

```
┌─────────────────────────────────────────────────────────────────┐
│                   CONSISTENCY EXAMPLE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   CONSISTENT STATE:                                             │
│                                                                 │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐                 │
│   │  Node A  │    │  Node B  │    │  Node C  │                 │
│   │          │    │          │    │          │                 │
│   │ Balance: │    │ Balance: │    │ Balance: │                 │
│   │   $100   │    │   $100   │    │   $100   │                 │
│   └──────────┘    └──────────┘    └──────────┘                 │
│                                                                 │
│   All nodes agree. Any query returns $100. ✓                   │
│                                                                 │
│                                                                 │
│   INCONSISTENT STATE:                                           │
│                                                                 │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐                 │
│   │  Node A  │    │  Node B  │    │  Node C  │                 │
│   │          │    │          │    │          │                 │
│   │ Balance: │    │ Balance: │    │ Balance: │                 │
│   │   $100   │    │   $80    │    │   $100   │                 │
│   └──────────┘    └──────────┘    └──────────┘                 │
│                                                                 │
│   Node B has different data! Query result depends on           │
│   which node you ask. ✗                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Levels of Consistency

```
┌─────────────────────────────────────────────────────────────────┐
│                  CONSISTENCY SPECTRUM                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   STRONGEST                                           WEAKEST   │
│       │                                                   │     │
│       ▼                                                   ▼     │
│   ┌────────────┐ ┌──────────┐ ┌────────────┐ ┌──────────────┐  │
│   │  STRONG/   │ │ CAUSAL   │ │  READ YOUR │ │  EVENTUAL    │  │
│   │ LINEARIZABLE│ │CONSISTENCY│ │   WRITES  │ │ CONSISTENCY  │  │
│   └────────────┘ └──────────┘ └────────────┘ └──────────────┘  │
│                                                                 │
│   Every read     Related ops  You see your   Eventually        │
│   sees the most  are ordered  own writes     all nodes         │
│   recent write   correctly    immediately    will agree        │
│                                                                 │
│   Highest        Moderate     Good UX for    Highest           │
│   latency        latency      single user    availability      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Strong Consistency

**Definition:** After a write completes, all subsequent reads will return that value.

```
Timeline:

Writer:     ────Write($100→$80)─────────────────►
                      │
                      │ Acknowledged
                      ▼
Reader 1:   ─────────────────Read────────────────►
                              │
                              └──► Returns $80 ✓

Reader 2:   ─────────────────────────Read────────►
                                      │
                                      └──► Returns $80 ✓

NO reader can see $100 after the write completes.
```

**Use cases:**
- Banking systems
- Inventory management
- Booking systems (seats, hotels)

### Eventual Consistency

**Definition:** If no new updates are made, eventually all nodes will converge to the same value.

```
Timeline:

Writer:     ────Write($100→$80)─────────────────────────►
                      │
                      │ Acknowledged (before full replication)
                      ▼
Reader 1:   ─────────────────Read──────────────────────►
                              │
                              └──► Returns $100 (stale!)

Reader 2:   ───────────────────────────────Read────────►
                                            │
                                            └──► Returns $80 ✓

After some time, all reads return $80.
```

**Use cases:**
- Social media likes/views
- Product reviews
- DNS
- Shopping cart (non-critical)

---

## The Trade-off in Action

### Why Can't We Have Both?

In a distributed system, network issues WILL happen. When they do:

```
┌─────────────────────────────────────────────────────────────────┐
│                 NETWORK PARTITION SCENARIO                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│              Normal Operation                                   │
│              ─────────────────                                  │
│                                                                 │
│         ┌──────────┐     ┌──────────┐                          │
│         │  Node A  │◄───►│  Node B  │                          │
│         │   $100   │     │   $100   │                          │
│         └──────────┘     └──────────┘                          │
│                                                                 │
│                                                                 │
│              Network Partition (Connection Lost)                │
│              ───────────────────────────────────                │
│                                                                 │
│         ┌──────────┐  ✗  ┌──────────┐                          │
│         │  Node A  │     │  Node B  │                          │
│         │   $100   │     │   $100   │                          │
│         └──────────┘     └──────────┘                          │
│                                                                 │
│         User tries to withdraw $20 at Node A                    │
│                                                                 │
│         CHOICE 1: CONSISTENCY (CP)                              │
│         ──────────────────────────                              │
│         "Sorry, I can't process this until I                    │
│          confirm with Node B. Please wait."                     │
│         → System is UNAVAILABLE but CONSISTENT                  │
│                                                                 │
│         CHOICE 2: AVAILABILITY (AP)                             │
│         ─────────────────────────────                           │
│         "Done! Your new balance is $80."                        │
│         But Node B still shows $100...                          │
│         → System is AVAILABLE but INCONSISTENT                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Real-World Examples

| System | Choice | Why |
|--------|--------|-----|
| **Banks** | Consistency | Can't have wrong balances |
| **Twitter** | Availability | Missing a like is okay |
| **Inventory** | Consistency | Can't oversell |
| **DNS** | Availability | Old IP better than no IP |
| **E-commerce cart** | Availability | Cart can sync later |
| **Flight booking** | Consistency | Can't double-book seats |

---

## Practical Patterns

### Pattern 1: Read Your Own Writes

Even with eventual consistency, users should see their own updates immediately.

```
┌─────────────────────────────────────────────────────────────────┐
│               READ YOUR OWN WRITES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   User posts a comment:                                         │
│                                                                 │
│   Step 1: Write goes to primary database                        │
│   Step 2: User's subsequent reads go to primary (not replica)   │
│                                                                 │
│         ┌──────────┐                                            │
│   User ─►│  Write   │──────►┌──────────┐                        │
│         └──────────┘       │  Primary  │──────► Replicas        │
│         ┌──────────┐       │    DB     │       (async)          │
│   User ─►│  Read    │──────►└──────────┘                        │
│         └──────────┘                                            │
│                      │                                          │
│                      └── User reads from primary,               │
│                          sees their own write immediately       │
│                                                                 │
│   Other users might read from replicas (eventual consistency)   │
│   but that's usually acceptable.                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Pattern 2: Monotonic Reads

Once you've seen a value, you won't see an older value.

```
┌─────────────────────────────────────────────────────────────────┐
│                   MONOTONIC READS                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   WITHOUT MONOTONIC READS (Confusing!):                         │
│                                                                 │
│   Read 1 (from Replica A): Comment has 10 likes                 │
│   Read 2 (from Replica B): Comment has 8 likes  ← Went down?!   │
│   Read 3 (from Replica A): Comment has 12 likes                 │
│                                                                 │
│   WITH MONOTONIC READS (Better UX):                             │
│                                                                 │
│   Read 1: Comment has 10 likes                                  │
│   Read 2: Comment has 10 likes  ← Sticky session to same replica│
│   Read 3: Comment has 12 likes                                  │
│                                                                 │
│   Implementation: Route user to same replica (sticky sessions)  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Pattern 3: Write Quorums

Balance consistency and availability with quorum-based systems.

```
┌─────────────────────────────────────────────────────────────────┐
│                    QUORUM CONSENSUS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Given N replicas:                                             │
│   ├── W = Number of nodes that must acknowledge a write         │
│   └── R = Number of nodes that must respond to a read           │
│                                                                 │
│   RULE: W + R > N guarantees seeing latest write                │
│                                                                 │
│   Example with N=3 replicas:                                    │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  Setting      │ W │ R │ Behavior                        │  │
│   ├───────────────┼───┼───┼─────────────────────────────────┤  │
│   │ Strong read   │ 2 │ 2 │ W+R=4 > 3 ✓ Always consistent   │  │
│   │ Fast read     │ 2 │ 1 │ W+R=3 = 3 ✗ May see stale       │  │
│   │ Fast write    │ 1 │ 2 │ W+R=3 = 3 ✗ May see stale       │  │
│   │ Balanced      │ 2 │ 2 │ W+R=4 > 3 ✓ Good balance        │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
│   Visual:                                                       │
│                                                                 │
│   Write (W=2):                  Read (R=2):                     │
│   ┌───┐ ┌───┐ ┌───┐            ┌───┐ ┌───┐ ┌───┐               │
│   │ ✓ │ │ ✓ │ │   │            │ ✓ │ │   │ │ ✓ │               │
│   │ A │ │ B │ │ C │            │ A │ │ B │ │ C │               │
│   └───┘ └───┘ └───┘            └───┘ └───┘ └───┘               │
│     │     │                      │           │                  │
│     └──┬──┘                      └─────┬─────┘                  │
│        │                               │                        │
│   At least one                   At least one                   │
│   overlap guaranteed!            has the latest                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Deep Dive

### Question: "How would you design a banking system?"

**Strong Answer:**

```
REQUIREMENTS:
├── Strong consistency (can't have wrong balances)
├── High availability (bank can't be down)
└── ACID transactions

APPROACH: Prioritize Consistency (CP system)

Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌────────────┐                                                 │
│  │ API Gateway│                                                 │
│  └─────┬──────┘                                                 │
│        │                                                        │
│        ▼                                                        │
│  ┌────────────────────┐                                         │
│  │ Transaction Service│                                         │
│  └─────────┬──────────┘                                         │
│            │                                                    │
│            ▼                                                    │
│  ┌─────────────────────────────────────────┐                   │
│  │         Primary Database                 │                   │
│  │  ┌─────────────────────────────────┐    │                   │
│  │  │     Synchronous Replication     │    │                   │
│  │  └─────────────────────────────────┘    │                   │
│  │              │                          │                   │
│  │     ┌────────┼────────┐                 │                   │
│  │     ▼        ▼        ▼                 │                   │
│  │  ┌─────┐  ┌─────┐  ┌─────┐             │                   │
│  │  │Sync │  │Sync │  │Sync │             │                   │
│  │  │Rep 1│  │Rep 2│  │Rep 3│             │                   │
│  │  └─────┘  └─────┘  └─────┘             │                   │
│  └─────────────────────────────────────────┘                   │
│                                                                 │
│  Key decisions:                                                 │
│  ├── Synchronous replication (wait for ack from replicas)       │
│  ├── Two-phase commit for cross-account transfers               │
│  ├── Pessimistic locking on accounts                            │
│  └── Accept higher latency for guaranteed consistency           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Question: "How would you design a social media like counter?"

**Strong Answer:**

```
REQUIREMENTS:
├── High availability (likes should always work)
├── High throughput (millions of likes per second)
└── Eventual consistency is acceptable

APPROACH: Prioritize Availability (AP system)

Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌────────────┐                                                 │
│  │   Client   │─────► Like request                              │
│  └─────┬──────┘                                                 │
│        │                                                        │
│        ▼                                                        │
│  ┌────────────────┐                                             │
│  │  Load Balancer │                                             │
│  └────────┬───────┘                                             │
│           │                                                     │
│     ┌─────┴─────┐                                               │
│     ▼           ▼                                               │
│ ┌────────┐  ┌────────┐                                          │
│ │Server 1│  │Server 2│                                          │
│ └───┬────┘  └───┬────┘                                          │
│     │           │                                               │
│     └─────┬─────┘                                               │
│           ▼                                                     │
│   ┌─────────────────┐                                           │
│   │  Message Queue  │ ◄── Async processing                      │
│   │    (Kafka)      │                                           │
│   └────────┬────────┘                                           │
│            │                                                    │
│            ▼                                                    │
│   ┌─────────────────┐                                           │
│   │ Counter Service │                                           │
│   │  (Aggregator)   │                                           │
│   └────────┬────────┘                                           │
│            │                                                    │
│            ▼                                                    │
│   ┌─────────────────┐                                           │
│   │     Redis       │ ◄── Increment counter                     │
│   │  (Eventually    │                                           │
│   │   consistent)   │                                           │
│   └─────────────────┘                                           │
│                                                                 │
│  Key decisions:                                                 │
│  ├── Async writes (immediate response to user)                  │
│  ├── Counter aggregation (batch updates)                        │
│  ├── Eventual consistency (likes might lag a few seconds)       │
│  └── Display cached counts (refreshed periodically)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Common Interview Traps

### Trap 1: "Always choose consistency for critical data"

**Wrong:** Even banks use eventual consistency for some things (like viewing transaction history).

**Right:** Match consistency requirements to the specific operation.

### Trap 2: "Eventual consistency means data loss"

**Wrong:** Data is never lost; it just takes time to propagate.

**Right:** All writes are durable; reads might see older values temporarily.

### Trap 3: "Strong consistency is always slower"

**Wrong:** For single-node systems or local reads, strong consistency can be fast.

**Right:** The latency cost comes from coordination across distributed nodes.

---

## Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│                    REMEMBER THIS                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. AVAILABILITY = System responds (measured in "nines")       │
│     CONSISTENCY = All users see same data                      │
│                                                                │
│  2. You CANNOT have perfect consistency AND availability       │
│     during network partitions (this leads to CAP theorem)      │
│                                                                │
│  3. Most systems use EVENTUAL CONSISTENCY for scalability      │
│     with patterns to improve UX (read-your-writes, etc.)       │
│                                                                │
│  4. Match consistency level to business requirements:          │
│     ├── Money, inventory → Strong consistency                  │
│     └── Likes, views, feeds → Eventual consistency             │
│                                                                │
│  5. Quorums (W + R > N) allow tunable consistency              │
│                                                                │
│  6. Always clarify consistency requirements in interviews!     │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

You've learned about the trade-off between availability and consistency. Now let's formalize this with the famous CAP theorem.

**Next:** [CAP Theorem →](cap_theorem.md)
