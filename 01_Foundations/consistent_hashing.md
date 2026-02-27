# Consistent Hashing

## The Simple Explanation

Consistent hashing is a technique for distributing data across multiple servers in a way that minimizes reorganization when servers are added or removed.

```
┌─────────────────────────────────────────────────────────────────┐
│                THE PROBLEM WITH REGULAR HASHING                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Regular approach: server = hash(key) % num_servers            │
│                                                                 │
│   With 3 servers:                                               │
│   hash("user_1") % 3 = 0 → Server A                            │
│   hash("user_2") % 3 = 1 → Server B                            │
│   hash("user_3") % 3 = 2 → Server C                            │
│   hash("user_4") % 3 = 0 → Server A                            │
│                                                                 │
│   Now add a 4th server:                                         │
│   hash("user_1") % 4 = 1 → Server B  (was A!)                  │
│   hash("user_2") % 4 = 2 → Server C  (was B!)                  │
│   hash("user_3") % 4 = 3 → Server D  (was C!)                  │
│   hash("user_4") % 4 = 0 → Server A  (unchanged)               │
│                                                                 │
│   PROBLEM: Almost ALL keys moved! (~75% in this case)           │
│            Massive cache invalidation and data migration        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Why This Matters

```
┌─────────────────────────────────────────────────────────────────┐
│                   REAL WORLD IMPACT                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Scenario: Cache cluster with 10 servers, 1 billion keys       │
│                                                                 │
│   Adding 1 server with regular hashing:                         │
│   • ~909 million keys need to move (10/11 ≈ 90%)                │
│   • Massive network traffic                                     │
│   • Cache miss storm (data not where expected)                  │
│   • Database overwhelmed by cache misses                        │
│   • OUTAGE!                                                     │
│                                                                 │
│   Adding 1 server with consistent hashing:                      │
│   • ~91 million keys need to move (1/11 ≈ 9%)                   │
│   • Minimal disruption                                          │
│   • System stays healthy                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## How Consistent Hashing Works

### The Ring Concept

Instead of modulo, we use a ring (circular space):

```
┌─────────────────────────────────────────────────────────────────┐
│                   THE HASH RING                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Imagine numbers 0 to 2³² arranged in a circle:                │
│                                                                 │
│                            0                                    │
│                           ╱│╲                                   │
│                         ╱  │  ╲                                 │
│                       ╱    │    ╲                               │
│                     ╱      │      ╲                             │
│               2³²-1        │        2³⁰                         │
│                   │        │        │                           │
│                   │        │        │                           │
│                   │        │        │                           │
│                   │        │        │                           │
│               2³¹+2³⁰      │      2³⁰+2²⁹                       │
│                     ╲      │      ╱                             │
│                       ╲    │    ╱                               │
│                         ╲  │  ╱                                 │
│                           ╲│╱                                   │
│                           2³¹                                   │
│                                                                 │
│   Both SERVERS and KEYS are hashed onto this ring.              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Placing Servers on the Ring

```
┌─────────────────────────────────────────────────────────────────┐
│               SERVERS ON THE RING                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   hash("Server_A") = position on ring                           │
│   hash("Server_B") = position on ring                           │
│   hash("Server_C") = position on ring                           │
│                                                                 │
│                           0°                                    │
│                          ╱│╲                                    │
│                        ╱  │  ╲                                  │
│                      ╱    │    ╲                                │
│                    ╱      │      ╲                              │
│                  ╱        │        ╲                            │
│              270° ────────┼──────── 90°                         │
│                │          │          │                          │
│                │    [A]   │          │                          │
│                │   /      │          │                          │
│               [C]         │          │                          │
│                  ╲        │        ╱                            │
│                    ╲      │      ╱                              │
│                      ╲    │    ╱                                │
│                        ╲  │  ╱                                  │
│                          ╲│╱                                    │
│                          180°                                   │
│                          [B]                                    │
│                                                                 │
│   Servers A, B, C positioned based on their hash values         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Finding the Server for a Key

**Rule: Walk clockwise from key's position until you hit a server.**

```
┌─────────────────────────────────────────────────────────────────┐
│              KEY ASSIGNMENT                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                           0°                                    │
│                      key_1 ●                                    │
│                        ╱   │                                    │
│                      ╱     │                                    │
│                    ╱  ┌────┼─────── Walk clockwise              │
│                  ╱    │    │          until server              │
│                ╱      ▼    │                                    │
│            270° ────[A]────┼──────── 90°                        │
│                │           │          │                         │
│          key_3 ●           │          │                         │
│                │╲          │          │                         │
│                │ ╲         │       ● key_2                      │
│               [C]  ╲       │        ╲│                          │
│                     ╲      │         ╲│                         │
│                      ╲     │          ▼                         │
│                       ╲    │         [B]                        │
│                        ╲   │        180°                        │
│                         ╲  │                                    │
│                                                                 │
│   key_1 → walks clockwise → hits Server A                       │
│   key_2 → walks clockwise → hits Server B                       │
│   key_3 → walks clockwise → hits Server C                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Adding a New Server

```
┌─────────────────────────────────────────────────────────────────┐
│              ADDING SERVER D                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   BEFORE (3 servers):              AFTER (4 servers):           │
│                                                                 │
│           0°                               0°                   │
│           │                                │                    │
│       ● key_1                          ● key_1                  │
│        ╲  │                             ╲  │                    │
│         ╲ │                              ╲ │                    │
│          ╲│                               ╲│                    │
│    270°──[A]── 90°                  270°──[A]── 90°             │
│      │                                │   │                     │
│      │                                │  [D]  ◄── New server!  │
│     [C]                              [C]  │                     │
│        ╲                                ╲ ● key_2               │
│         ╲                                ╲│                     │
│          ╲                                ╲│                    │
│          [B]                              [B]                   │
│          180°                             180°                  │
│                                                                 │
│   WHAT MOVED:                                                   │
│   • key_1: Still → Server A (no change)                         │
│   • key_2: Was → Server B, Now → Server D                       │
│   • key_3: Still → Server C (no change)                         │
│                                                                 │
│   Only keys BETWEEN the new server and its predecessor move!    │
│   Expected: ~1/4 of keys (25%), not 75%!                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Removing a Server

```
┌─────────────────────────────────────────────────────────────────┐
│              REMOVING SERVER B                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   BEFORE (3 servers):              AFTER (2 servers):           │
│                                                                 │
│           0°                               0°                   │
│           │                                │                    │
│       ● key_1                          ● key_1                  │
│        ╲  │                             ╲  │                    │
│         ╲ │                              ╲ │                    │
│          ╲│                               ╲│                    │
│    270°──[A]── 90°                  270°──[A]── 90°             │
│      │                                │                         │
│     [C]                              [C]──────┐                  │
│        ╲  ● key_2                       ╲     │                 │
│         ╲  │                             ╲    │                 │
│          ╲ │                              ╲   │                 │
│          [B] ◄── Removed!                 ● key_2 (now → C)    │
│          180°                             180°                  │
│                                                                 │
│   Only keys that were on Server B need to move!                 │
│   They go to the next server clockwise (Server C).              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Problem: Uneven Distribution

With just a few servers, distribution can be very uneven:

```
┌─────────────────────────────────────────────────────────────────┐
│              UNEVEN DISTRIBUTION PROBLEM                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                           0°                                    │
│                           │                                     │
│                           │                                     │
│                          [A]                                    │
│                          /│                                     │
│                         / │                                     │
│              ──────────/──┼──────────                           │
│                      [B]  │                                     │
│                       │   │                                     │
│                       │   │                                     │
│                       │   │                                     │
│                       │   │                                     │
│                       │   │                                     │
│                       │   │                                     │
│                       │   │                                     │
│                       │  [C]                                    │
│                           180°                                  │
│                                                                 │
│   Server distribution:                                          │
│   • Server A: handles 5% of keys                                │
│   • Server B: handles 10% of keys                               │
│   • Server C: handles 85% of keys  ← OVERLOADED!                │
│                                                                 │
│   This happens because server positions are random!             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Virtual Nodes: The Solution

Instead of one position per server, each server gets multiple positions (virtual nodes):

```
┌─────────────────────────────────────────────────────────────────┐
│                    VIRTUAL NODES                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Each physical server gets multiple positions on the ring:     │
│                                                                 │
│   Server A → hash("A-1"), hash("A-2"), hash("A-3"), ...         │
│   Server B → hash("B-1"), hash("B-2"), hash("B-3"), ...         │
│   Server C → hash("C-1"), hash("C-2"), hash("C-3"), ...         │
│                                                                 │
│                           0°                                    │
│                          [A1]                                   │
│                      [C2] │ [B1]                                │
│                        ╲  │  ╱                                  │
│                         ╲ │ ╱                                   │
│                          ╲│╱                                    │
│                  [B3]─────┼─────[A2]                            │
│                          ╱│╲                                    │
│                         ╱ │ ╲                                   │
│                        ╱  │  ╲                                  │
│                      [A3] │ [C1]                                │
│                          [B2]                                   │
│                          180°                                   │
│                                                                 │
│   Now each server "owns" multiple small segments!               │
│   Distribution is much more even.                               │
│                                                                 │
│   Typical: 100-200 virtual nodes per physical server            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Why Virtual Nodes Work

```
┌─────────────────────────────────────────────────────────────────┐
│              VIRTUAL NODE BENEFITS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. EVEN DISTRIBUTION                                          │
│      • More points = statistically more even                    │
│      • Law of large numbers                                     │
│                                                                 │
│   2. FLEXIBLE CAPACITY                                          │
│      • Powerful server: more virtual nodes                      │
│      • Weak server: fewer virtual nodes                         │
│      • Example: Server A (16GB) gets 200 vnodes                 │
│                 Server B (8GB) gets 100 vnodes                  │
│                                                                 │
│   3. SMOOTHER REBALANCING                                       │
│      • Adding server: steal from many servers, not just one     │
│      • Removing server: distribute to many servers              │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  Without virtual nodes:                                 │  │
│   │  Remove Server B → ALL its keys go to Server C          │  │
│   │                    Server C doubles its load!           │  │
│   │                                                         │  │
│   │  With virtual nodes:                                    │  │
│   │  Remove Server B → keys distributed to A, C, D, E...    │  │
│   │                    Each server gets small increase      │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Real-World Usage

### Where Consistent Hashing is Used

```
┌────────────────────────────────────────────────────────────────┐
│              CONSISTENT HASHING IN THE WILD                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  DISTRIBUTED CACHES                                            │
│  └── Memcached clusters                                        │
│  └── Redis Cluster (uses hash slots, similar concept)          │
│                                                                │
│  DISTRIBUTED DATABASES                                         │
│  └── Amazon DynamoDB                                           │
│  └── Apache Cassandra                                          │
│  └── Riak                                                      │
│                                                                │
│  LOAD BALANCERS                                                │
│  └── Consistent hashing for sticky sessions                    │
│  └── Backend server selection                                  │
│                                                                │
│  CONTENT DELIVERY                                              │
│  └── CDN node selection                                        │
│  └── Akamai (pioneered this technique)                         │
│                                                                │
│  MESSAGE QUEUES                                                │
│  └── Kafka partition assignment                                │
│  └── Consumer group coordination                               │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Example: Distributed Cache

```
┌─────────────────────────────────────────────────────────────────┐
│              DISTRIBUTED CACHE EXAMPLE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client wants to cache user_123's data                         │
│                                                                 │
│   1. COMPUTE HASH                                               │
│      hash("user_123") = 847392...                               │
│                                                                 │
│   2. FIND SERVER ON RING                                        │
│      Walk clockwise from 847392... → hits Server B              │
│                                                                 │
│   3. STORE/RETRIEVE                                             │
│      SET on Server B: cache.set("user_123", data)               │
│      GET from Server B: cache.get("user_123")                   │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                                                         │  │
│   │   Client ──► hash(key) ──► find server ──► Cache Node   │  │
│   │                                                         │  │
│   │   Any client can independently find the right server    │  │
│   │   No central coordinator needed!                        │  │
│   │                                                         │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation Sketch

### Basic Implementation (Python)

```python
import hashlib
from bisect import bisect_right

class ConsistentHash:
    def __init__(self, nodes=None, virtual_nodes=100):
        self.virtual_nodes = virtual_nodes
        self.ring = {}  # hash -> node mapping
        self.sorted_keys = []  # sorted hash values

        if nodes:
            for node in nodes:
                self.add_node(node)

    def _hash(self, key):
        """Generate hash value for a key."""
        return int(hashlib.md5(key.encode()).hexdigest(), 16)

    def add_node(self, node):
        """Add a node with virtual nodes."""
        for i in range(self.virtual_nodes):
            virtual_key = f"{node}-{i}"
            hash_val = self._hash(virtual_key)
            self.ring[hash_val] = node
            self.sorted_keys.append(hash_val)

        self.sorted_keys.sort()

    def remove_node(self, node):
        """Remove a node and its virtual nodes."""
        for i in range(self.virtual_nodes):
            virtual_key = f"{node}-{i}"
            hash_val = self._hash(virtual_key)
            del self.ring[hash_val]
            self.sorted_keys.remove(hash_val)

    def get_node(self, key):
        """Find which node a key belongs to."""
        if not self.ring:
            return None

        hash_val = self._hash(key)

        # Find first node clockwise from hash
        idx = bisect_right(self.sorted_keys, hash_val)

        # Wrap around if we've gone past the end
        if idx == len(self.sorted_keys):
            idx = 0

        return self.ring[self.sorted_keys[idx]]


# Usage
ch = ConsistentHash(["server_a", "server_b", "server_c"])

# Find server for keys
print(ch.get_node("user_123"))  # → "server_b"
print(ch.get_node("user_456"))  # → "server_a"

# Add a server - minimal keys move
ch.add_node("server_d")

# Remove a server - only its keys redistribute
ch.remove_node("server_b")
```

---

## Comparison: Regular vs Consistent Hashing

```
┌─────────────────────────────────────────────────────────────────┐
│           REGULAR vs CONSISTENT HASHING                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Metric              │ Regular Hash  │ Consistent Hash         │
│   ────────────────────┼───────────────┼───────────────────────  │
│   Algorithm           │ hash % N      │ Ring + clockwise walk   │
│   Add server          │ ~N/(N+1) move │ ~1/(N+1) move           │
│   Remove server       │ ~N/(N-1) move │ ~1/(N-1) move           │
│   Distribution        │ Even          │ Even (with vnodes)      │
│   Complexity          │ O(1)          │ O(log N) with vnodes    │
│   State needed        │ Just N        │ Ring structure          │
│                                                                 │
│   Example with 10 servers, adding 1:                            │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  Regular:    ~91% of keys move (10/11)                  │  │
│   │  Consistent: ~9% of keys move (1/11)                    │  │
│   │                                                         │  │
│   │  With 1 billion keys:                                   │  │
│   │  Regular:    910 million keys move                      │  │
│   │  Consistent: 90 million keys move                       │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Question 1: "Explain consistent hashing and when you'd use it"

**Strong Answer:**
```
Consistent hashing is a technique for distributing data across
multiple nodes while minimizing reorganization when nodes change.

HOW IT WORKS:
1. Arrange hash space in a ring (0 to 2^32)
2. Hash both servers and keys onto the ring
3. Each key belongs to the first server clockwise from it
4. Adding/removing servers only affects nearby keys

WHY IT MATTERS:
With regular hashing (hash % n), changing n servers moves ~100% of keys.
With consistent hashing, only ~1/n keys move.

WHEN TO USE:
• Distributed caches (Memcached clusters)
• Distributed databases (Cassandra, DynamoDB)
• Load balancing with session affinity
• Any system where:
  - Data is partitioned across nodes
  - Nodes may be added/removed dynamically
  - Minimizing data movement is important

VIRTUAL NODES:
To ensure even distribution, each physical server gets 100+
virtual positions on the ring.
```

### Question 2: "Design a distributed cache using consistent hashing"

**Strong Answer:**
```
ARCHITECTURE:

Client Library:
• Maintains consistent hash ring
• Receives cluster membership updates
• Routes requests to correct server

Cache Servers:
• Simple key-value stores
• No coordination between servers
• Report health to cluster manager

Cluster Manager:
• Tracks which servers are alive
• Broadcasts membership changes
• Handles server addition/removal

FLOW:
1. Client: hash(key) → find server on ring
2. Client: send request to that server
3. Server: handle GET/SET locally

HANDLING FAILURES:
• Health checks detect dead servers
• Remove from ring → keys move to next server
• Clients get updated ring from cluster manager

REPLICATION (optional):
• Store on next N servers clockwise
• Read from any, write to all N
• Handles server failure without data loss

CONFIGURATION:
• 100-200 virtual nodes per server
• Health check every 5 seconds
• Replication factor: 3 for production
```

---

## Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│                    REMEMBER THIS                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. Regular hashing (% N) moves ~100% of keys when N changes   │
│     Consistent hashing moves only ~1/N keys                    │
│                                                                │
│  2. THE RING CONCEPT:                                          │
│     • Hash servers and keys onto a ring                        │
│     • Key belongs to first server clockwise                    │
│     • Changes only affect neighboring segment                  │
│                                                                │
│  3. VIRTUAL NODES solve uneven distribution                    │
│     • Each server gets 100+ positions                          │
│     • Enables weighted distribution                            │
│     • Smoother rebalancing                                     │
│                                                                │
│  4. USED IN: Cassandra, DynamoDB, Memcached, CDNs              │
│                                                                │
│  5. KEY BENEFIT: Scale nodes up/down with minimal disruption   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## Congratulations!

You've completed the **Foundations** section! You now understand:
- What system design is
- How the internet works
- Latency, throughput, availability, consistency
- CAP theorem
- Scaling strategies
- Load balancing
- Caching
- Storage types
- SQL vs NoSQL
- Indexing
- Consistent hashing

**Next Section:** [02_High_Level_Design](../02_High_Level_Design/) - Learn how to approach system design interviews with a structured framework.
