# Multi-Region Design

## Why Multi-Region?

Multi-region architecture places your system in multiple geographic locations to achieve:

```
┌─────────────────────────────────────────────────────────────┐
│              WHY MULTI-REGION?                              │
│                                                             │
│  1. LOWER LATENCY                                           │
│     Users connect to nearest region                        │
│     US user → US datacenter (20ms)                        │
│     vs US user → EU datacenter (150ms)                    │
│                                                             │
│  2. DISASTER RECOVERY                                       │
│     Entire region can fail (rare but happens)             │
│     - Natural disasters                                    │
│     - Cloud provider outages                               │
│     - Political/regulatory issues                          │
│                                                             │
│  3. DATA SOVEREIGNTY                                        │
│     Some data must stay in specific countries             │
│     - GDPR (EU data in EU)                                │
│     - Data residency laws                                  │
│                                                             │
│  4. HIGHER AVAILABILITY                                     │
│     Single region: 99.99% max                             │
│     Multi-region: 99.999%+ possible                       │
└─────────────────────────────────────────────────────────────┘
```

## Multi-Region Patterns

### Pattern 1: Active-Passive (Primary-Backup)

```
┌─────────────────────────────────────────────────────────────┐
│            ACTIVE-PASSIVE MULTI-REGION                      │
│                                                             │
│       PRIMARY REGION (US-East)     BACKUP REGION (US-West) │
│      ┌─────────────────────┐      ┌─────────────────────┐  │
│      │                     │      │                     │  │
│      │  ┌───────────────┐  │      │  ┌───────────────┐  │  │
│      │  │  App Servers  │◀─┼──────┼──│  App Servers  │  │  │
│      │  └───────┬───────┘  │      │  │   (Standby)   │  │  │
│      │          │          │      │  └───────────────┘  │  │
│      │  ┌───────▼───────┐  │      │                     │  │
│ All  │  │   Database    │──┼──────┼─▶┌───────────────┐  │  │
│Traffic  │   (Primary)   │  │ Async│  │   Database    │  │  │
│  ──────▶└───────────────┘  │ Repl │  │   (Replica)   │  │  │
│      │                     │      │  └───────────────┘  │  │
│      └─────────────────────┘      └─────────────────────┘  │
│                                                             │
│  HOW IT WORKS:                                              │
│  - All traffic goes to primary region                      │
│  - Data replicated async to backup                         │
│  - On failure, DNS switches to backup                      │
│                                                             │
│  PROS:                        CONS:                         │
│  ├── Simple                   ├── Backup region unused     │
│  ├── No data conflicts        ├── Failover takes minutes   │
│  └── Lower cost               └── Possible data loss (lag) │
│                                                             │
│  USE WHEN:                                                  │
│  - Cost is a concern                                       │
│  - Can tolerate minutes of downtime                        │
│  - Writes must be consistent                               │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 2: Active-Active (Both Serve Traffic)

```
┌─────────────────────────────────────────────────────────────┐
│             ACTIVE-ACTIVE MULTI-REGION                      │
│                                                             │
│                      GLOBAL DNS                             │
│                   (GeoDNS/Latency)                         │
│                          │                                  │
│            ┌─────────────┴─────────────┐                   │
│            │                           │                   │
│            ▼                           ▼                   │
│   REGION: US-East              REGION: EU-West             │
│  ┌─────────────────┐          ┌─────────────────┐         │
│  │                 │          │                 │         │
│  │ ┌─────────────┐ │          │ ┌─────────────┐ │         │
│  │ │ App Servers │ │          │ │ App Servers │ │         │
│  │ └──────┬──────┘ │          │ └──────┬──────┘ │         │
│  │        │        │          │        │        │         │
│  │ ┌──────▼──────┐ │◀────────▶│ ┌──────▼──────┐ │         │
│  │ │  Database   │ │  Bi-dir  │ │  Database   │ │         │
│  │ │  (Primary)  │ │  Repl    │ │  (Primary)  │ │         │
│  │ └─────────────┘ │          │ └─────────────┘ │         │
│  │                 │          │                 │         │
│  └─────────────────┘          └─────────────────┘         │
│                                                             │
│  HOW IT WORKS:                                              │
│  - Users routed to nearest region                          │
│  - Both regions handle reads AND writes                    │
│  - Data synced bidirectionally                             │
│                                                             │
│  CHALLENGE: Write conflicts!                               │
│  User A updates profile in US                             │
│  User A updates profile in EU (before sync)               │
│  Which write wins?                                         │
│                                                             │
│  SOLUTIONS:                                                 │
│  - Last-write-wins (simple but lossy)                     │
│  - Conflict resolution logic                               │
│  - CRDTs (conflict-free replicated data types)           │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 3: Follow-the-Sun / Regional Primary

```
┌─────────────────────────────────────────────────────────────┐
│               REGIONAL PRIMARY                              │
│                                                             │
│  Each user has a "home region" where their data lives      │
│                                                             │
│       US Users           EU Users           APAC Users     │
│          │                  │                   │          │
│          ▼                  ▼                   ▼          │
│     ┌─────────┐        ┌─────────┐        ┌─────────┐     │
│     │  US DB  │        │  EU DB  │        │ APAC DB │     │
│     │ Primary │        │ Primary │        │ Primary │     │
│     │for US   │        │for EU   │        │for APAC │     │
│     └─────────┘        └─────────┘        └─────────┘     │
│          │                  │                   │          │
│          └──────────────────┴───────────────────┘          │
│                 Cross-region reads only                    │
│                 (for traveling users)                      │
│                                                             │
│  USER ASSIGNMENT:                                           │
│  - Based on signup location                                │
│  - Or based on phone number country code                  │
│  - Or user choice                                          │
│                                                             │
│  WRITES: Always go to home region                         │
│  READS: Local region first, home region if needed         │
│                                                             │
│  PROS:                                                      │
│  - No write conflicts                                      │
│  - Data sovereignty compliance                            │
│  - Clear data ownership                                    │
│                                                             │
│  CONS:                                                      │
│  - Latency for traveling users                            │
│  - Complex user routing                                    │
│  - Region migration needed if user moves                  │
└─────────────────────────────────────────────────────────────┘
```

## Global Traffic Management

### GeoDNS

```
┌─────────────────────────────────────────────────────────────┐
│                      GeoDNS                                 │
│                                                             │
│  Routes users to nearest datacenter based on IP location   │
│                                                             │
│      US User                              EU User          │
│         │                                    │             │
│         │ DNS: api.example.com              │             │
│         ▼                                    ▼             │
│    ┌──────────────────────────────────────────────┐       │
│    │              GEODNS SERVICE                  │       │
│    │  (Route53, Cloudflare, NS1)                 │       │
│    │                                              │       │
│    │  US IP → 54.23.12.1 (US datacenter)        │       │
│    │  EU IP → 35.12.45.6 (EU datacenter)        │       │
│    └──────────────────────────────────────────────┘       │
│              │                    │                        │
│              ▼                    ▼                        │
│    ┌─────────────────┐  ┌─────────────────┐              │
│    │  US Datacenter  │  │  EU Datacenter  │              │
│    │   54.23.12.1    │  │   35.12.45.6    │              │
│    └─────────────────┘  └─────────────────┘              │
│                                                             │
│  LIMITATIONS:                                               │
│  - VPN users get wrong location                           │
│  - IP geolocation not 100% accurate                       │
│  - DNS caching can cause stickiness                       │
└─────────────────────────────────────────────────────────────┘
```

### Global Load Balancer (Anycast)

```
┌─────────────────────────────────────────────────────────────┐
│                 GLOBAL LOAD BALANCER                        │
│                                                             │
│  Single IP announced from multiple locations               │
│  Network routing sends to nearest                          │
│                                                             │
│      User in Tokyo        User in London                   │
│           │                     │                          │
│           │  1.2.3.4           │  1.2.3.4                 │
│           │  (same IP!)        │  (same IP!)              │
│           ▼                     ▼                          │
│    ┌───────────┐          ┌───────────┐                   │
│    │   GLB     │          │   GLB     │                   │
│    │  Tokyo    │          │  London   │                   │
│    └─────┬─────┘          └─────┬─────┘                   │
│          │                      │                          │
│          ▼                      ▼                          │
│    ┌───────────┐          ┌───────────┐                   │
│    │   APAC    │          │    EU     │                   │
│    │  Region   │          │  Region   │                   │
│    └───────────┘          └───────────┘                   │
│                                                             │
│  SERVICES: AWS Global Accelerator, Cloudflare, Akamai     │
│                                                             │
│  BENEFITS OVER GeoDNS:                                      │
│  - No DNS propagation delay                                │
│  - Works with VPNs                                         │
│  - Automatic failover                                      │
└─────────────────────────────────────────────────────────────┘
```

## Data Replication Strategies

### Synchronous vs Asynchronous

```
┌─────────────────────────────────────────────────────────────┐
│          CROSS-REGION REPLICATION                           │
│                                                             │
│  SYNCHRONOUS:                                               │
│  ┌─────────┐                      ┌─────────┐              │
│  │ Region A│───Write─────────────▶│ Region B│              │
│  │   DB    │◀──Acknowledge────────│   DB    │              │
│  └─────────┘                      └─────────┘              │
│                                                             │
│  - Both regions confirm before commit                      │
│  - Zero data loss                                          │
│  - HIGH LATENCY: +100-200ms per write                     │
│  - Used for: Financial transactions                       │
│                                                             │
│  ASYNCHRONOUS:                                              │
│  ┌─────────┐                      ┌─────────┐              │
│  │ Region A│──Write──┐            │ Region B│              │
│  │   DB    │         └──Later────▶│   DB    │              │
│  └─────────┘                      └─────────┘              │
│       │                                                     │
│  Return immediately                                        │
│                                                             │
│  - Primary confirms, replicates in background              │
│  - Possible data loss (replication lag)                   │
│  - LOW LATENCY: Normal write speed                        │
│  - Used for: Most applications                            │
│                                                             │
│  REPLICATION LAG:                                           │
│  Same region: 1-10 ms                                      │
│  Cross-region: 50-200 ms typically                        │
│  Under load: Can be seconds                               │
└─────────────────────────────────────────────────────────────┘
```

### Conflict Resolution

```
┌─────────────────────────────────────────────────────────────┐
│              CONFLICT RESOLUTION                            │
│                                                             │
│  SCENARIO: Same row updated in two regions simultaneously │
│                                                             │
│  Region US:  UPDATE users SET name='John' WHERE id=1     │
│  Region EU:  UPDATE users SET name='Johnny' WHERE id=1   │
│                                                             │
│  STRATEGIES:                                                │
│                                                             │
│  1. LAST WRITE WINS (LWW)                                  │
│     - Timestamp determines winner                          │
│     - Simple but may lose data                            │
│     - Need synchronized clocks                            │
│                                                             │
│  2. APPLICATION-LEVEL RESOLUTION                           │
│     - Application logic merges changes                    │
│     - Example: Shopping cart union                         │
│     - Complex but flexible                                 │
│                                                             │
│  3. CRDTs (Conflict-free Replicated Data Types)           │
│     - Data structures that auto-merge                     │
│     - Counters, sets, registers                           │
│     - Mathematically guaranteed no conflicts             │
│                                                             │
│  4. AVOID CONFLICTS                                         │
│     - Route writes to single region                       │
│     - Per-user home region                                │
│     - Partition data by region                            │
│                                                             │
│  MOST COMMON: Avoid conflicts with smart routing          │
└─────────────────────────────────────────────────────────────┘
```

## Multi-Region Architecture Example

### Global Social Media Platform

```
┌─────────────────────────────────────────────────────────────┐
│         MULTI-REGION SOCIAL PLATFORM                        │
│                                                             │
│                    ┌─────────────┐                         │
│                    │   Global    │                         │
│                    │    CDN      │ Static assets           │
│                    └──────┬──────┘                         │
│                           │                                 │
│                    ┌──────▼──────┐                         │
│                    │   Global    │                         │
│                    │     LB      │ Route by location       │
│                    └──────┬──────┘                         │
│         ┌─────────────────┼─────────────────┐             │
│         │                 │                 │             │
│         ▼                 ▼                 ▼             │
│    ┌─────────┐       ┌─────────┐       ┌─────────┐       │
│    │   US    │       │   EU    │       │  APAC   │       │
│    │ Region  │       │ Region  │       │ Region  │       │
│    └────┬────┘       └────┬────┘       └────┬────┘       │
│         │                 │                 │             │
│    ┌────┴────┐       ┌────┴────┐       ┌────┴────┐       │
│    │         │       │         │       │         │       │
│   App      Cache    App      Cache    App      Cache     │
│ Servers    Redis   Servers   Redis  Servers    Redis     │
│    │         │       │         │       │         │       │
│    └────┬────┘       └────┬────┘       └────┬────┘       │
│         │                 │                 │             │
│    ┌────▼────┐       ┌────▼────┐       ┌────▼────┐       │
│    │  MySQL  │       │  MySQL  │       │  MySQL  │       │
│    │(Primary │◀─────▶│(Primary │◀─────▶│(Primary │       │
│    │ for US) │       │ for EU) │       │for APAC)│       │
│    └─────────┘       └─────────┘       └─────────┘       │
│                                                             │
│  ROUTING LOGIC:                                             │
│  1. CDN: Serve static files from edge                     │
│  2. GLB: Route user to nearest region                     │
│  3. App: Check if data is local or needs cross-region    │
│  4. DB: Read local, write to user's home region          │
└─────────────────────────────────────────────────────────────┘
```

### Data Locality Example

```
┌─────────────────────────────────────────────────────────────┐
│             DATA LOCALITY STRATEGY                          │
│                                                             │
│  GLOBAL DATA (replicated everywhere):                      │
│  - System configuration                                    │
│  - Product catalog                                         │
│  - Reference data                                          │
│                                                             │
│  REGIONAL DATA (in user's home region):                   │
│  - User profiles                                           │
│  - User posts                                              │
│  - User messages                                           │
│  - Purchase history                                        │
│                                                             │
│  REQUEST FLOW:                                              │
│                                                             │
│  EU user viewing US user's profile:                        │
│  1. Request hits EU region (nearest)                      │
│  2. EU app checks: Is target user in EU?                  │
│  3. No → Fetch from US region                             │
│  4. Cache result in EU for future reads                   │
│                                                             │
│  EU user posting:                                          │
│  1. Request hits EU region (nearest)                      │
│  2. Is EU user's home region? Yes                        │
│  3. Write locally, success                                │
│                                                             │
│  EU user traveling, posting from US:                      │
│  1. Request hits US region (nearest)                      │
│  2. Is EU user's home region? No                         │
│  3. Route write to EU region                              │
│  4. Slightly higher latency but correct                   │
└─────────────────────────────────────────────────────────────┘
```

## Multi-Region Challenges

### Challenge 1: Cross-Region Latency

```
┌─────────────────────────────────────────────────────────────┐
│          TYPICAL CROSS-REGION LATENCIES                     │
│                                                             │
│  ROUTE                    LATENCY (round-trip)             │
│  ──────────────────────────────────────────────            │
│  Same datacenter          < 1 ms                           │
│  Same region (diff AZ)    1-2 ms                           │
│  US East ↔ US West       60-80 ms                          │
│  US ↔ Europe             80-120 ms                         │
│  US ↔ Asia               150-250 ms                        │
│  Europe ↔ Asia           200-300 ms                        │
│                                                             │
│  MITIGATIONS:                                               │
│  ├── Aggressive caching of remote data                    │
│  ├── Async operations where possible                      │
│  ├── Batch cross-region requests                          │
│  └── Accept eventual consistency                          │
└─────────────────────────────────────────────────────────────┘
```

### Challenge 2: Clock Synchronization

```
┌─────────────────────────────────────────────────────────────┐
│              CLOCK ISSUES                                   │
│                                                             │
│  Problem: Different servers have different times           │
│                                                             │
│  Server A (US):  10:00:00.000                             │
│  Server B (EU):  10:00:00.050  (50ms ahead)               │
│                                                             │
│  If using timestamps for ordering:                        │
│  - US write at 10:00:00.100                               │
│  - EU write at 10:00:00.080 (actually happened after!)   │
│  - LWW picks EU write incorrectly                        │
│                                                             │
│  SOLUTIONS:                                                 │
│  1. NTP synchronization (±10ms typical)                   │
│  2. Hybrid logical clocks                                  │
│  3. True Time (Google Spanner) - atomic clocks           │
│  4. Vector clocks (detect conflicts)                      │
│  5. Avoid timestamp-based ordering                        │
└─────────────────────────────────────────────────────────────┘
```

### Challenge 3: Split Brain

```
┌─────────────────────────────────────────────────────────────┐
│                SPLIT BRAIN                                  │
│                                                             │
│  Scenario: Network partition between regions              │
│                                                             │
│        US Region              EU Region                    │
│       ┌─────────┐            ┌─────────┐                  │
│       │   DB    │    X       │   DB    │                  │
│       │(thinks  │───────────│(thinks  │                  │
│       │it's     │  Network  │it's     │                  │
│       │primary) │  Broken   │primary) │                  │
│       └─────────┘            └─────────┘                  │
│                                                             │
│  Both think they're primary = data divergence!            │
│                                                             │
│  PREVENTION:                                                │
│  1. Quorum-based decisions (need majority)                │
│  2. External witness/arbiter                              │
│  3. Prefer availability: Accept divergence, merge later   │
│  4. Prefer consistency: One region goes read-only        │
└─────────────────────────────────────────────────────────────┘
```

## Multi-Region Checklist

```
┌─────────────────────────────────────────────────────────────┐
│           MULTI-REGION CHECKLIST                            │
│                                                             │
│  INFRASTRUCTURE                                             │
│  □ At least 2 regions (different failure domains)         │
│  □ Global load balancing configured                        │
│  □ CDN for static content                                  │
│  □ Independent deployments per region                      │
│                                                             │
│  DATA                                                       │
│  □ Replication strategy chosen (sync/async)               │
│  □ Conflict resolution strategy defined                   │
│  □ User home region assignment logic                      │
│  □ Cross-region read strategy                             │
│                                                             │
│  ROUTING                                                    │
│  □ User routing logic implemented                         │
│  □ Failover routing tested                                 │
│  □ Sticky sessions handled                                 │
│                                                             │
│  OPERATIONS                                                 │
│  □ Per-region monitoring                                   │
│  □ Cross-region latency alerts                            │
│  □ Replication lag monitoring                             │
│  □ Failover runbooks                                       │
│  □ Region evacuation procedure                            │
│                                                             │
│  TESTING                                                    │
│  □ Region failover tested                                  │
│  □ Network partition scenarios tested                     │
│  □ Data consistency after failover verified               │
└─────────────────────────────────────────────────────────────┘
```

## Interview Tips

```
1. START SIMPLE
   "For initial scale, single-region with multi-AZ is enough.
    We'd add multi-region when we need global latency or
    true DR capability."

2. EXPLAIN DATA STRATEGY
   "Users would have a home region. Writes always go to home
    region, reads can be served locally with async replication."

3. ACKNOWLEDGE COMPLEXITY
   "Multi-region adds significant complexity. We'd need to
    handle: data conflicts, cross-region latency, deployment
    coordination, and split-brain scenarios."

4. KNOW THE TRADEOFFS
   "Active-active gives us better latency but we need to
    handle write conflicts. Active-passive is simpler but
    means higher latency for non-primary region users."
```

---

**Next:** [10_Read_Write_Patterns.md](./10_Read_Write_Patterns.md) - Optimize for your workload type
