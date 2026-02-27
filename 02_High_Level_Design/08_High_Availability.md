# Designing for High Availability

## What is High Availability?

High Availability (HA) means your system **continues working even when things fail**. It's measured as uptime percentage.

```
┌─────────────────────────────────────────────────────────────┐
│                 AVAILABILITY TARGETS                        │
│                                                             │
│  "Nines"    Uptime %    Downtime/Year    Downtime/Month    │
│  ──────────────────────────────────────────────────────    │
│  Two 9s     99%         3.65 days        7.3 hours         │
│  Three 9s   99.9%       8.76 hours       43.8 minutes      │
│  Four 9s    99.99%      52.6 minutes     4.4 minutes       │
│  Five 9s    99.999%     5.26 minutes     26 seconds        │
│                                                             │
│  TYPICAL TARGETS:                                           │
│  ├── Internal tools: 99% (acceptable for non-critical)    │
│  ├── Consumer apps: 99.9% - 99.99%                        │
│  ├── Financial systems: 99.99%+                           │
│  └── Life-critical: 99.999%+                              │
│                                                             │
│  Each additional "9" is 10x harder and more expensive!    │
└─────────────────────────────────────────────────────────────┘
```

## The Enemy: Single Points of Failure (SPOF)

A SPOF is any component whose failure brings down the entire system.

```
┌─────────────────────────────────────────────────────────────┐
│            SINGLE POINTS OF FAILURE                         │
│                                                             │
│  BAD DESIGN (Full of SPOFs):                               │
│                                                             │
│  Users ──▶ [Single Server] ──▶ [Single DB]                │
│                    ↑                 ↑                      │
│                  SPOF              SPOF                     │
│                                                             │
│  If server dies: System down                               │
│  If DB dies: System down                                   │
│  If network fails: System down                             │
│                                                             │
│  GOAL: Eliminate ALL single points of failure              │
└─────────────────────────────────────────────────────────────┘
```

### Identifying SPOFs

```
Common SPOFs to check:
├── Load balancer (single LB?)
├── Database (single primary?)
├── Cache (single Redis instance?)
├── DNS (single provider?)
├── Network (single ISP/region?)
├── Storage (single disk/array?)
├── Authentication service
└── Configuration server
```

## HA Architecture Patterns

### Pattern 1: Active-Passive (Failover)

```
┌─────────────────────────────────────────────────────────────┐
│               ACTIVE-PASSIVE                                │
│                                                             │
│         Normal Operation          After Failover            │
│                                                             │
│    ┌──────────┐                  ┌──────────┐              │
│    │  ACTIVE  │◀── Traffic      │  ACTIVE  │              │
│    │ (Primary)│                  │ (Primary)│ (dead)       │
│    └────┬─────┘                  └──────────┘              │
│         │                              │                    │
│    Heartbeat                      ┌────▼─────┐             │
│         │                         │  PASSIVE │◀── Traffic  │
│    ┌────▼─────┐                   │(now Active)│            │
│    │  PASSIVE │ (standby)        └──────────┘              │
│    │ (Standby)│                                             │
│    └──────────┘                                             │
│                                                             │
│  HOW IT WORKS:                                              │
│  1. Passive monitors Active via heartbeat                  │
│  2. If heartbeat fails, Passive takes over                │
│  3. DNS/LB redirects traffic to new Active                │
│                                                             │
│  PROS: Simple, less resource usage                         │
│  CONS: Failover takes time (seconds to minutes)           │
│  USE: Databases, single-primary systems                   │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 2: Active-Active

```
┌─────────────────────────────────────────────────────────────┐
│                ACTIVE-ACTIVE                                │
│                                                             │
│         Normal Operation          After Failure             │
│                                                             │
│              Traffic                    Traffic             │
│                │                           │                │
│         ┌──────┴──────┐             ┌──────┴──────┐        │
│         │             │             │             │        │
│    ┌────▼────┐   ┌────▼────┐   ┌────▼────┐   ┌────────┐   │
│    │ ACTIVE  │   │ ACTIVE  │   │ ACTIVE  │   │ ACTIVE │   │
│    │ Node 1  │   │ Node 2  │   │ Node 1  │   │ Node 2 │   │
│    └─────────┘   └─────────┘   └─────────┘   │ (dead) │   │
│                                               └────────┘   │
│                                                             │
│  HOW IT WORKS:                                              │
│  1. All nodes actively handle traffic                      │
│  2. Load balancer distributes requests                     │
│  3. If one dies, others absorb its traffic                │
│                                                             │
│  PROS: No failover delay, better resource utilization     │
│  CONS: More complex, data sync challenges                 │
│  USE: Stateless services, read replicas                   │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 3: Database HA with Replication

```
┌─────────────────────────────────────────────────────────────┐
│           DATABASE HIGH AVAILABILITY                        │
│                                                             │
│  SYNCHRONOUS REPLICATION:                                   │
│  ┌─────────┐    Sync Write    ┌─────────┐                 │
│  │ Primary │─────────────────▶│ Standby │                 │
│  │   DB    │◀─────────────────│   DB    │                 │
│  └─────────┘    Acknowledge    └─────────┘                 │
│                                                             │
│  - Write confirmed after both have it                      │
│  - No data loss on failover                                │
│  - Higher latency                                          │
│                                                             │
│  ASYNCHRONOUS REPLICATION:                                  │
│  ┌─────────┐    Async Write   ┌─────────┐                 │
│  │ Primary │─────────────────▶│ Replica │                 │
│  │   DB    │                  │   DB    │                 │
│  └─────────┘                  └─────────┘                 │
│                                                             │
│  - Write confirmed after primary only                      │
│  - Possible data loss on failover (lag)                   │
│  - Lower latency                                           │
│                                                             │
│  MULTI-REPLICA:                                             │
│  ┌─────────┐                                               │
│  │ Primary │──────┬──────────┬──────────┐                 │
│  │   DB    │      │          │          │                 │
│  └─────────┘      ▼          ▼          ▼                 │
│              ┌────────┐ ┌────────┐ ┌────────┐             │
│              │Replica1│ │Replica2│ │Replica3│             │
│              └────────┘ └────────┘ └────────┘             │
│                                                             │
│  Quorum: Majority must acknowledge (2 of 3)               │
└─────────────────────────────────────────────────────────────┘
```

## Redundancy at Every Layer

```
┌─────────────────────────────────────────────────────────────┐
│           REDUNDANCY AT EVERY LAYER                         │
│                                                             │
│  DNS LEVEL:                                                 │
│  Multiple DNS providers (Route53 + Cloudflare)             │
│                                                             │
│  CDN LEVEL:                                                 │
│  Multiple edge locations, automatic failover               │
│                                                             │
│  LOAD BALANCER:                                             │
│  ┌────────┐     ┌────────┐                                │
│  │  LB 1  │     │  LB 2  │  (Active-Passive or Active-Active) │
│  └────────┘     └────────┘                                │
│                                                             │
│  APPLICATION:                                               │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                      │
│  │App1│ │App2│ │App3│ │App4│ │App5│  (Multiple instances) │
│  └────┘ └────┘ └────┘ └────┘ └────┘                      │
│                                                             │
│  CACHE:                                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                   │
│  │ Redis 1  │ │ Redis 2  │ │ Redis 3  │  (Redis Cluster)  │
│  └──────────┘ └──────────┘ └──────────┘                   │
│                                                             │
│  DATABASE:                                                  │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐              │
│  │ Primary │────▶│ Standby │     │ Replica │              │
│  └─────────┘     └─────────┘     └─────────┘              │
│                                                             │
│  STORAGE:                                                   │
│  ┌─────────────────────────────────────────┐              │
│  │  S3 (11 nines durability, auto-replicated)│              │
│  └─────────────────────────────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

## Health Checks and Monitoring

### Types of Health Checks

```
┌─────────────────────────────────────────────────────────────┐
│               HEALTH CHECK TYPES                            │
│                                                             │
│  1. LIVENESS CHECK                                          │
│     "Is the service running?"                              │
│     GET /health → 200 OK                                   │
│     Failure: Restart the service                           │
│                                                             │
│  2. READINESS CHECK                                         │
│     "Can the service handle requests?"                     │
│     GET /ready → 200 OK (DB connected, cache warm)        │
│     Failure: Remove from load balancer                     │
│                                                             │
│  3. DEEP HEALTH CHECK                                       │
│     "Are all dependencies healthy?"                        │
│     GET /health/deep → Check DB, cache, external APIs     │
│     Use for debugging, not load balancer                   │
│                                                             │
│  EXAMPLE IMPLEMENTATION:                                    │
│  GET /health                                               │
│  {                                                          │
│    "status": "healthy",                                    │
│    "version": "1.2.3",                                     │
│    "uptime": "5d 12h 30m",                                │
│    "checks": {                                             │
│      "database": "ok",                                     │
│      "cache": "ok",                                        │
│      "disk": "ok"                                          │
│    }                                                        │
│  }                                                          │
└─────────────────────────────────────────────────────────────┘
```

### Monitoring and Alerting

```
┌─────────────────────────────────────────────────────────────┐
│            MONITORING PYRAMID                               │
│                                                             │
│                    ▲                                        │
│                   /│\                                       │
│                  / │ \      BUSINESS METRICS               │
│                 /  │  \     - Revenue, conversions         │
│                /   │   \    - User signups                 │
│               /    │    \                                   │
│              /     │     \  APPLICATION METRICS            │
│             /      │      \ - Request rate, latency        │
│            /       │       \- Error rate, queue depth      │
│           /        │        \                              │
│          /         │         \ INFRASTRUCTURE METRICS     │
│         /          │          \- CPU, memory, disk        │
│        /           │           \- Network, connections    │
│       ─────────────┴────────────                          │
│                                                             │
│  ALERT ON:                                                  │
│  ├── Error rate > 1%                                       │
│  ├── Latency P99 > 500ms                                  │
│  ├── CPU > 80% sustained                                  │
│  ├── Disk > 85% full                                      │
│  ├── Memory > 90%                                         │
│  └── Replication lag > 10s                                │
│                                                             │
│  DON'T ALERT ON:                                           │
│  ├── Every 404 error                                       │
│  ├── Brief CPU spikes                                      │
│  └── Expected maintenance events                          │
└─────────────────────────────────────────────────────────────┘
```

## Failure Handling Patterns

### Circuit Breaker

```
┌─────────────────────────────────────────────────────────────┐
│                CIRCUIT BREAKER                              │
│                                                             │
│  Prevents cascading failures by "breaking" when errors     │
│  exceed threshold.                                          │
│                                                             │
│  STATES:                                                    │
│                                                             │
│  ┌─────────┐ Errors exceed  ┌─────────┐                   │
│  │ CLOSED  │───threshold───▶│  OPEN   │                   │
│  │(Normal) │                │(Failing)│                   │
│  └─────────┘                └────┬────┘                   │
│       ▲                          │                         │
│       │                     Timeout                        │
│       │                          │                         │
│       │                    ┌─────▼─────┐                  │
│       │                    │HALF-OPEN  │                  │
│       │                    │(Testing)  │                  │
│       │                    └─────┬─────┘                  │
│       │                          │                         │
│       └────── Success ───────────┘                        │
│                                                             │
│  CLOSED: All requests pass through                        │
│  OPEN: All requests fail immediately (no call to service) │
│  HALF-OPEN: Limited requests test if service recovered    │
│                                                             │
│  EXAMPLE:                                                   │
│  - 50% errors in last 30 seconds → OPEN                   │
│  - Wait 60 seconds → HALF-OPEN                            │
│  - 3 successful requests → CLOSED                         │
└─────────────────────────────────────────────────────────────┘
```

### Retry with Exponential Backoff

```
┌─────────────────────────────────────────────────────────────┐
│           EXPONENTIAL BACKOFF                               │
│                                                             │
│  Instead of retrying immediately, wait longer each time:   │
│                                                             │
│  Attempt 1: Fail → Wait 1 second                          │
│  Attempt 2: Fail → Wait 2 seconds                         │
│  Attempt 3: Fail → Wait 4 seconds                         │
│  Attempt 4: Fail → Wait 8 seconds                         │
│  Attempt 5: Fail → Give up                                 │
│                                                             │
│  WITH JITTER (randomization):                              │
│  wait = min(cap, base * 2^attempt) + random(0, 1000ms)    │
│                                                             │
│  WHY JITTER?                                                │
│  Without: All retries hit at same time (thundering herd)  │
│  With: Retries spread out, reducing load spikes           │
│                                                             │
│  PSEUDOCODE:                                                │
│  for attempt in range(max_retries):                        │
│      try:                                                   │
│          return make_request()                             │
│      except:                                                │
│          if attempt == max_retries - 1:                    │
│              raise                                          │
│          wait = min(30, (2 ** attempt)) + random(0, 1)    │
│          sleep(wait)                                        │
└─────────────────────────────────────────────────────────────┘
```

### Bulkhead Pattern

```
┌─────────────────────────────────────────────────────────────┐
│               BULKHEAD PATTERN                              │
│                                                             │
│  Like ship bulkheads: Isolate failures to prevent          │
│  the entire system from sinking.                           │
│                                                             │
│  WITHOUT BULKHEADS:                                         │
│  ┌──────────────────────────────────────────┐             │
│  │              SINGLE THREAD POOL          │             │
│  │  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐    │             │
│  │  │ A  │ │ B  │ │ B  │ │ B  │ │ B  │    │             │
│  └──────────────────────────────────────────┘             │
│  If Service B is slow, it blocks threads for A too!       │
│                                                             │
│  WITH BULKHEADS:                                            │
│  ┌─────────────────┐  ┌─────────────────┐                 │
│  │  POOL A (20)    │  │  POOL B (30)    │                 │
│  │  ┌────┐ ┌────┐  │  │  ┌────┐ ┌────┐  │                 │
│  │  │ A  │ │ A  │  │  │  │ B  │ │ B  │  │                 │
│  └─────────────────┘  └─────────────────┘                 │
│  Service B problems don't affect Service A!               │
│                                                             │
│  IMPLEMENTATIONS:                                           │
│  ├── Separate thread pools per service                    │
│  ├── Separate connection pools per dependency             │
│  ├── Rate limiting per tenant                             │
│  └── Separate processes/containers                        │
└─────────────────────────────────────────────────────────────┘
```

### Timeout Configuration

```
┌─────────────────────────────────────────────────────────────┐
│               TIMEOUT STRATEGY                              │
│                                                             │
│  Rule: Every external call needs a timeout!                │
│                                                             │
│  TYPICAL TIMEOUTS:                                          │
│  ├── Database query: 5-30 seconds                         │
│  ├── Cache (Redis): 100-500 ms                            │
│  ├── Internal service: 1-5 seconds                        │
│  ├── External API: 5-30 seconds                           │
│  └── User-facing request: 10-30 seconds total             │
│                                                             │
│  TIMEOUT BUDGET:                                            │
│  User request has 10 second budget:                        │
│  ├── Auth service: 500ms                                   │
│  ├── Product service: 2s                                   │
│  ├── Recommendation service: 2s                           │
│  ├── Database: 3s                                          │
│  └── Buffer: 2.5s                                          │
│                                                             │
│  COMMON MISTAKES:                                           │
│  ├── No timeout (request hangs forever)                   │
│  ├── Timeout too long (resources exhausted)              │
│  ├── Timeout too short (false failures)                  │
│  └── Not accounting for retries                          │
└─────────────────────────────────────────────────────────────┘
```

## Disaster Recovery

### RPO and RTO

```
┌─────────────────────────────────────────────────────────────┐
│                RPO vs RTO                                   │
│                                                             │
│  RPO (Recovery Point Objective):                           │
│  "How much data can we afford to lose?"                    │
│                                                             │
│  RTO (Recovery Time Objective):                            │
│  "How long can we be down?"                                │
│                                                             │
│  Timeline:                                                  │
│                                                             │
│  ◀────── RPO ──────▶                    ◀──── RTO ────▶   │
│  │                  │                    │              │   │
│  ▼                  ▼                    ▼              ▼   │
│  ┌──────────────────┬────────────────────┬──────────────┐  │
│  │   Last Backup    │     DISASTER       │   Recovery   │  │
│  │                  │     OCCURS         │   Complete   │  │
│  └──────────────────┴────────────────────┴──────────────┘  │
│                                                             │
│  TYPICAL TARGETS:                                           │
│  ├── Financial: RPO = 0, RTO = minutes                    │
│  ├── E-commerce: RPO = 1 hour, RTO = 1 hour              │
│  ├── Internal tools: RPO = 24 hours, RTO = 4 hours       │
│                                                             │
│  Lower RPO/RTO = More expensive                           │
└─────────────────────────────────────────────────────────────┘
```

### Backup Strategies

```
┌─────────────────────────────────────────────────────────────┐
│              BACKUP STRATEGIES                              │
│                                                             │
│  1. FULL BACKUP                                             │
│     - Complete copy of all data                            │
│     - Slow, storage-intensive                              │
│     - Weekly typical                                        │
│                                                             │
│  2. INCREMENTAL BACKUP                                      │
│     - Only changes since last backup                       │
│     - Fast, less storage                                    │
│     - Daily or hourly                                       │
│                                                             │
│  3. POINT-IN-TIME RECOVERY (PITR)                          │
│     - Transaction logs enable any-point recovery          │
│     - Very low RPO (seconds)                               │
│     - More complex                                          │
│                                                             │
│  BACKUP SCHEDULE EXAMPLE:                                   │
│  ├── Full backup: Weekly (Sunday 2 AM)                    │
│  ├── Incremental: Daily                                    │
│  ├── Transaction logs: Every 5 minutes                    │
│  └── Retention: 30 days                                    │
│                                                             │
│  THE 3-2-1 RULE:                                           │
│  - 3 copies of data                                        │
│  - 2 different storage types                               │
│  - 1 offsite (different region/provider)                  │
└─────────────────────────────────────────────────────────────┘
```

## High Availability Checklist

```
┌─────────────────────────────────────────────────────────────┐
│           HIGH AVAILABILITY CHECKLIST                       │
│                                                             │
│  INFRASTRUCTURE                                             │
│  □ No single points of failure                             │
│  □ Multiple availability zones                             │
│  □ Load balancer redundancy                                │
│  □ Database replication configured                         │
│  □ Auto-scaling enabled                                     │
│                                                             │
│  APPLICATION                                                │
│  □ Stateless services (or state externalized)             │
│  □ Health check endpoints                                   │
│  □ Graceful shutdown handling                              │
│  □ Circuit breakers for dependencies                       │
│  □ Retry logic with backoff                                │
│  □ Timeouts on all external calls                          │
│                                                             │
│  DATA                                                       │
│  □ Regular backups tested                                  │
│  □ Point-in-time recovery enabled                          │
│  □ Cross-region backup copies                              │
│  □ Backup restoration tested                               │
│                                                             │
│  MONITORING                                                 │
│  □ Health dashboards                                        │
│  □ Alerting configured                                      │
│  □ On-call rotation                                         │
│  □ Runbooks for common failures                            │
│                                                             │
│  TESTING                                                    │
│  □ Chaos engineering (kill instances)                      │
│  □ Failover drills                                          │
│  □ Disaster recovery tests                                 │
│  □ Load testing                                             │
└─────────────────────────────────────────────────────────────┘
```

## Interview Tips for HA

```
1. IDENTIFY SPOFs FIRST
   "Let me identify single points of failure in this design..."

2. QUANTIFY AVAILABILITY
   "This design achieves 99.9% availability because..."

3. EXPLAIN FAILOVER
   "If the primary database fails, the standby takes over
    in under 30 seconds using automatic failover..."

4. MENTION TRADEOFFS
   "Synchronous replication gives us zero data loss but
    adds 5-10ms latency to writes..."

5. DISCUSS TESTING
   "We'd validate this with chaos engineering - randomly
    killing instances to ensure failover works..."
```

---

**Next:** [09_Multi_Region_Design.md](./09_Multi_Region_Design.md) - Go global with your HA design
