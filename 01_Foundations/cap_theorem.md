# CAP Theorem

## The Simple Explanation

The CAP theorem says that in a distributed system, when a network failure happens, you can only guarantee TWO of these three properties:

```
┌─────────────────────────────────────────────────────────────────┐
│                      CAP THEOREM                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                         C                                       │
│                    CONSISTENCY                                  │
│                    /         \                                  │
│                   /           \                                 │
│                  /             \                                │
│                 /   You can     \                               │
│                /    only pick    \                              │
│               /        TWO        \                             │
│              /                     \                            │
│             A ─────────────────────── P                         │
│       AVAILABILITY            PARTITION                         │
│                               TOLERANCE                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Definitions

| Property | Meaning | Simple Analogy |
|----------|---------|----------------|
| **C**onsistency | All nodes see the same data | Everyone has the same version of the document |
| **A**vailability | Every request gets a response | The restaurant is always open |
| **P**artition Tolerance | System works despite network failures | The show goes on even if some performers are stuck in traffic |

---

## Understanding Each Property

### Consistency (C)

Every read receives the most recent write or an error.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONSISTENT SYSTEM                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Time ──────────────────────────────────────────────────────►  │
│                                                                 │
│   Writer:      Write(X=5)                                       │
│                    │                                            │
│                    ▼                                            │
│   Node A:     [X=1] ───► [X=5] ──────────────────────────►     │
│                              │                                  │
│                    (Sync)    │                                  │
│                              ▼                                  │
│   Node B:     [X=1] ───────► [X=5] ───────────────────────►    │
│                                                                 │
│   Reader:                      Read(X)                          │
│                                  │                              │
│                                  ▼                              │
│                            Returns X=5 ✓                        │
│                                                                 │
│   After a write completes, ALL reads return the new value.     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Availability (A)

Every request receives a non-error response (though it might not be the latest data).

```
┌─────────────────────────────────────────────────────────────────┐
│                    AVAILABLE SYSTEM                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   User Request                                                  │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────────┐                                               │
│   │   System    │                                               │
│   └─────────────┘                                               │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────────┐                                               │
│   │  Response   │  ← ALWAYS responds (never errors/timeouts)    │
│   │   (data)    │                                               │
│   └─────────────┘                                               │
│                                                                 │
│   Note: The data might be stale, but you WILL get a response.  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Partition Tolerance (P)

The system continues to operate despite network partitions.

```
┌─────────────────────────────────────────────────────────────────┐
│                  NETWORK PARTITION                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   BEFORE PARTITION:                                             │
│                                                                 │
│   ┌──────────┐    Network OK    ┌──────────┐                   │
│   │  Node A  │◄────────────────►│  Node B  │                   │
│   │  (X=5)   │                  │  (X=5)   │                   │
│   └──────────┘                  └──────────┘                   │
│                                                                 │
│   AFTER PARTITION:                                              │
│                                                                 │
│   ┌──────────┐    XXXX BROKEN   ┌──────────┐                   │
│   │  Node A  │       XXXX       │  Node B  │                   │
│   │  (X=5)   │                  │  (X=5)   │                   │
│   └──────────┘                  └──────────┘                   │
│        │                              │                        │
│        ▼                              ▼                        │
│   Still handles              Still handles                     │
│   requests!                  requests!                         │
│                                                                 │
│   A partition-tolerant system keeps running on both sides.     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Key Insight: Partitions Are Inevitable

Here's the critical understanding:

```
┌─────────────────────────────────────────────────────────────────┐
│                    THE REAL CHOICE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   In distributed systems, network partitions WILL happen.       │
│                                                                 │
│   Therefore: P (Partition Tolerance) is NOT optional.           │
│                                                                 │
│   So the REAL choice is:                                        │
│                                                                 │
│              During a partition, choose:                        │
│                                                                 │
│              CP                    or                AP         │
│         (Consistency)                         (Availability)    │
│                                                                 │
│   "Refuse to respond        vs.    "Respond with possibly      │
│    until I'm sure the              stale data rather than      │
│    data is correct"                making users wait"          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## CP vs AP Systems

### CP Systems (Consistency + Partition Tolerance)

When a partition happens, CP systems sacrifice availability to maintain consistency.

```
┌─────────────────────────────────────────────────────────────────┐
│                       CP SYSTEM                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   During Partition:                                             │
│                                                                 │
│   ┌──────────┐      PARTITION     ┌──────────┐                 │
│   │  Node A  │         ✗          │  Node B  │                 │
│   │  (X=5)   │                    │  (X=5)   │                 │
│   └────┬─────┘                    └────┬─────┘                 │
│        │                               │                        │
│   User tries                      User tries                    │
│   to write X=10                   to read X                     │
│        │                               │                        │
│        ▼                               ▼                        │
│   ┌─────────────┐                ┌─────────────┐               │
│   │   ERROR!    │                │   ERROR!    │               │
│   │ "Cannot     │                │ "Cannot     │               │
│   │  confirm    │                │  confirm    │               │
│   │  write"     │                │  latest     │               │
│   └─────────────┘                │  value"     │               │
│                                  └─────────────┘               │
│                                                                 │
│   System becomes UNAVAILABLE to guarantee CONSISTENCY.          │
│                                                                 │
│   Examples: HBase, MongoDB (with certain configs), Redis Cluster│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**When to use CP:**
- Banking transactions
- Inventory management (prevent overselling)
- Booking systems (prevent double-booking)
- Any financial or critical data operation

### AP Systems (Availability + Partition Tolerance)

When a partition happens, AP systems sacrifice consistency to maintain availability.

```
┌─────────────────────────────────────────────────────────────────┐
│                       AP SYSTEM                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   During Partition:                                             │
│                                                                 │
│   ┌──────────┐      PARTITION     ┌──────────┐                 │
│   │  Node A  │         ✗          │  Node B  │                 │
│   │  (X=5)   │                    │  (X=5)   │                 │
│   └────┬─────┘                    └────┬─────┘                 │
│        │                               │                        │
│   User writes                     User reads                    │
│   X=10                            X                             │
│        │                               │                        │
│        ▼                               ▼                        │
│   ┌──────────┐                    ┌──────────┐                 │
│   │  Node A  │                    │  Returns │                 │
│   │  (X=10)  │                    │   X=5    │  ← Stale!       │
│   └──────────┘                    │  (works) │                 │
│                                   └──────────┘                 │
│                                                                 │
│   After partition heals, nodes sync and resolve conflicts.      │
│                                                                 │
│   Examples: Cassandra, DynamoDB, CouchDB, Riak                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**When to use AP:**
- Social media (likes, comments)
- Product catalogs
- Shopping carts
- DNS
- Content delivery

---

## System Classification

```
┌─────────────────────────────────────────────────────────────────┐
│                 CAP CLASSIFICATION EXAMPLES                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                         CONSISTENCY                             │
│                              ▲                                  │
│                              │                                  │
│        CP Systems            │                                  │
│        ───────────           │                                  │
│        • MongoDB (default)   │                                  │
│        • HBase               │                                  │
│        • Redis Cluster       │                                  │
│        • Zookeeper           │                                  │
│        • etcd                │                                  │
│                              │                                  │
│   ◄──────────────────────────┼──────────────────────────────►   │
│   AVAILABILITY               │         PARTITION TOLERANCE      │
│                              │                                  │
│        AP Systems            │         CA Systems               │
│        ──────────            │         ──────────               │
│        • Cassandra           │         • Traditional RDBMS      │
│        • DynamoDB            │           (single node)          │
│        • CouchDB             │         • PostgreSQL (single)    │
│        • Riak                │         • MySQL (single)         │
│        • Voldemort           │                                  │
│                              │         Note: CA doesn't exist   │
│                              │         in truly distributed     │
│                              ▼         systems!                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Why CA Doesn't Exist in Distributed Systems

```
┌─────────────────────────────────────────────────────────────────┐
│                   CA = MYTH FOR DISTRIBUTED                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   If you want CA (no partition tolerance):                      │
│   You're saying: "We assume the network NEVER fails"            │
│                                                                 │
│   In reality:                                                   │
│   • Network cables get cut                                      │
│   • Routers fail                                                │
│   • Data centers have outages                                   │
│   • Cloud regions go down                                       │
│                                                                 │
│   Therefore:                                                    │
│   ┌────────────────────────────────────────────────────────┐   │
│   │  Any system that spans multiple machines MUST handle   │   │
│   │  partitions. CA is only possible on a SINGLE machine.  │   │
│   └────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Single-node PostgreSQL = CA (but not distributed)             │
│   Multi-node anything = Must be CP or AP                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## PACELC: A More Complete Picture

CAP only talks about behavior during partitions. PACELC extends this:

```
┌─────────────────────────────────────────────────────────────────┐
│                        PACELC THEOREM                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   if (Partition) {                                              │
│       choose: Availability OR Consistency                       │
│   } else {  // Normal operation                                 │
│       choose: Latency OR Consistency                            │
│   }                                                             │
│                                                                 │
│   P  A  C  E  L  C                                              │
│   │  │  │  │  │  │                                              │
│   │  │  │  │  │  └── Consistency (when no partition)            │
│   │  │  │  │  └───── Latency (when no partition)                │
│   │  │  │  └──────── Else (normal operation)                    │
│   │  │  └─────────── Consistency (during partition)             │
│   │  └────────────── Availability (during partition)            │
│   └───────────────── Partition                                  │
│                                                                 │
│   Examples:                                                     │
│   ┌────────────────────────────────────────────────────────┐   │
│   │  System     │ Partition Choice │ Normal Operation       │   │
│   ├─────────────┼──────────────────┼───────────────────────┤   │
│   │  DynamoDB   │      PA          │       EL              │   │
│   │  (PA/EL)    │  (Availability)  │   (Low Latency)       │   │
│   ├─────────────┼──────────────────┼───────────────────────┤   │
│   │  MongoDB    │      PC          │       EC              │   │
│   │  (PC/EC)    │  (Consistency)   │   (Consistency)       │   │
│   ├─────────────┼──────────────────┼───────────────────────┤   │
│   │  Cassandra  │      PA          │       EL              │   │
│   │  (PA/EL)    │  (Availability)  │   (Low Latency)       │   │
│   └────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practical Application: Making the Choice

### Decision Framework

```
┌─────────────────────────────────────────────────────────────────┐
│              HOW TO CHOOSE: CP vs AP                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Ask yourself:                                                 │
│                                                                 │
│   ┌────────────────────────────────────────────────────────┐   │
│   │  "What's worse for my users?"                          │   │
│   │                                                        │   │
│   │  A) System says "please wait" or "error"              │   │
│   │     (temporary unavailability)                         │   │
│   │                                                        │   │
│   │  B) System gives wrong/stale data                     │   │
│   │     (temporary inconsistency)                          │   │
│   └────────────────────────────────────────────────────────┘   │
│                                                                 │
│   If A is worse → Choose AP (stay available)                    │
│   If B is worse → Choose CP (stay consistent)                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Real-World Decision Examples

```
SCENARIO 1: Bank Balance Check
─────────────────────────────
Q: What's worse - showing "please wait" or showing wrong balance?
A: Wrong balance is worse (could cause overdraft, confusion)
→ Choose CP

SCENARIO 2: Instagram Like Count
────────────────────────────────
Q: What's worse - like button not working, or showing 999 instead of 1000?
A: Button not working is worse (frustrating UX)
→ Choose AP

SCENARIO 3: Flight Seat Selection
─────────────────────────────────
Q: What's worse - "please wait" or double-booking a seat?
A: Double-booking is worse (legal/operational nightmare)
→ Choose CP

SCENARIO 4: Shopping Cart
─────────────────────────
Q: What's worse - can't add items, or cart shows old state briefly?
A: Can't add is worse (lost sales)
→ Choose AP (can reconcile later)

SCENARIO 5: Medical Records
───────────────────────────
Q: What's worse - "system down" or showing outdated allergies?
A: Outdated allergies could be fatal
→ Choose CP
```

---

## Interview Questions

### Question 1: "Explain CAP theorem to me"

**Strong Answer:**
```
"CAP theorem states that in a distributed system, when a network
partition occurs, you must choose between consistency and availability.

- Consistency means all nodes see the same data
- Availability means the system always responds
- Partition tolerance means the system handles network failures

Since network failures are inevitable in distributed systems,
partition tolerance is required. So the real choice is between
CP (sacrifice availability for consistency) and AP (sacrifice
consistency for availability).

For example, a banking system would choose CP because showing
wrong balances is unacceptable. A social media like counter would
choose AP because users prefer the button to work, even if the
count is momentarily stale."
```

### Question 2: "Is MongoDB CP or AP?"

**Strong Answer:**
```
"MongoDB is typically classified as CP, but it's configurable.

By default, MongoDB uses a single primary for writes and requires
acknowledgment from a majority of nodes. During a partition where
the primary is isolated:
- The minority partition cannot elect a new primary
- Writes are unavailable until the partition heals
- This is CP behavior

However, you can configure MongoDB to:
- Allow reads from secondaries (potentially stale data)
- Use lower write concerns (w:1 instead of w:majority)

This makes it more AP-like, but with consistency trade-offs.

The key insight is that CAP isn't binary—systems can be tuned
along a spectrum based on configuration and use case."
```

### Question 3: "Design a system that needs both high consistency AND high availability"

**Strong Answer:**
```
"You can't have perfect consistency AND availability during partitions,
but you can design for high levels of both through:

1. **Minimize partitions**
   - Multiple network paths
   - Same-region deployments
   - High-quality infrastructure

2. **Different consistency levels per operation**
   - Critical operations (transfers): Strong consistency
   - Non-critical operations (balance view): Eventually consistent

3. **Optimistic locking with conflict resolution**
   - Allow concurrent operations
   - Detect and resolve conflicts

4. **CRDT (Conflict-free Replicated Data Types)**
   - Data structures that automatically merge
   - Example: Counters that can be incremented anywhere

5. **Compensation transactions**
   - Allow inconsistency temporarily
   - Detect and fix after the fact (like double-booking resolution)

The key is accepting that trade-offs exist and designing specifically
for your business requirements rather than trying to achieve the
impossible."
```

---

## Common Misconceptions

### Misconception 1: "CAP means pick any two"

**Reality:** During normal operation, you can have all three. CAP only applies during network partitions. And since partitions happen, you're really choosing between CP and AP.

### Misconception 2: "AP systems lose data"

**Reality:** AP systems don't lose data. They might serve stale reads temporarily, but all writes are eventually persisted and propagated.

### Misconception 3: "Strong consistency is always better"

**Reality:** Strong consistency comes with latency and availability costs. Many applications work perfectly well with eventual consistency.

### Misconception 4: "CAP applies to single-node systems"

**Reality:** CAP only applies to distributed systems. A single PostgreSQL instance doesn't have network partitions between nodes.

---

## Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│                    REMEMBER THIS                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. CAP = Consistency, Availability, Partition Tolerance       │
│                                                                │
│  2. Partition tolerance is REQUIRED (networks fail)            │
│     So the real choice is: CP or AP                            │
│                                                                │
│  3. CP = System might be unavailable during partition          │
│     AP = System might serve stale data during partition        │
│                                                                │
│  4. Choose based on: "What's worse for my users?"              │
│     Wrong data → CP                                            │
│     System down → AP                                           │
│                                                                │
│  5. CAP is a spectrum, not binary                              │
│     Systems can be tuned via configuration                     │
│                                                                │
│  6. PACELC extends CAP to normal operation (Latency vs         │
│     Consistency when there's no partition)                     │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

Now that you understand CAP theorem, let's learn about how to actually handle growth—vertical vs horizontal scaling.

**Next:** [Vertical vs Horizontal Scaling →](scaling.md)
