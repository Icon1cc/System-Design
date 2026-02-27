# Capacity Estimation - Back-of-Envelope Calculations

## Why Estimation Matters

Capacity estimation transforms vague requirements into concrete numbers that drive architectural decisions.

```
┌─────────────────────────────────────────────────────────────┐
│                  WHY ESTIMATE?                              │
│                                                             │
│  WITHOUT NUMBERS:              WITH NUMBERS:                │
│  "We need caching"             "We need 50GB cache to       │
│                                 hold 80% of hot data"       │
│                                                             │
│  "We need sharding"            "10TB needs 100 shards       │
│                                 at 100GB each"              │
│                                                             │
│  "It needs to be fast"         "12K QPS needs 24 servers    │
│                                 at 500 QPS each"            │
│                                                             │
│  Vague → Precise → Actionable                              │
└─────────────────────────────────────────────────────────────┘
```

## The Numbers You Must Know

Memorize these - they're used constantly in interviews:

### Time Conversions
```
┌─────────────────────────────────────────────────────────────┐
│                   TIME CONVERSIONS                          │
│                                                             │
│  1 day    = 86,400 seconds  ≈ 100,000 seconds (for math)   │
│  1 month  = 2.5 million seconds                             │
│  1 year   = 31.5 million seconds ≈ 30 million              │
│                                                             │
│  Quick trick: seconds in a day ≈ 10^5                      │
└─────────────────────────────────────────────────────────────┘
```

### Data Size Conversions
```
┌─────────────────────────────────────────────────────────────┐
│                   DATA SIZES                                │
│                                                             │
│  1 Byte (B)        = 8 bits                                │
│  1 Kilobyte (KB)   = 1,000 bytes ≈ 10^3 bytes              │
│  1 Megabyte (MB)   = 1,000 KB ≈ 10^6 bytes                 │
│  1 Gigabyte (GB)   = 1,000 MB ≈ 10^9 bytes                 │
│  1 Terabyte (TB)   = 1,000 GB ≈ 10^12 bytes                │
│  1 Petabyte (PB)   = 1,000 TB ≈ 10^15 bytes                │
│                                                             │
│  Character         ≈ 1 byte (ASCII) or 2-4 bytes (UTF-8)   │
│  Integer           ≈ 4 bytes (32-bit) or 8 bytes (64-bit)  │
│  UUID              ≈ 16 bytes                               │
│  Timestamp         ≈ 8 bytes                                │
└─────────────────────────────────────────────────────────────┘
```

### Typical Object Sizes
```
┌─────────────────────────────────────────────────────────────┐
│                 TYPICAL SIZES                               │
│                                                             │
│  TYPE                    SIZE                               │
│  ─────────────────────────────────────                     │
│  Tweet (text only)       250 bytes (280 chars)              │
│  Tweet (with metadata)   1 KB                               │
│  User profile            1-2 KB                             │
│  Image (compressed)      200 KB - 2 MB                      │
│  Image (thumbnail)       10-50 KB                           │
│  Video (1 min, SD)       10-50 MB                           │
│  Video (1 min, HD)       100-500 MB                         │
│  Log entry               100-500 bytes                      │
│  Database row            100 bytes - 1 KB                   │
└─────────────────────────────────────────────────────────────┘
```

### Latency Numbers
```
┌─────────────────────────────────────────────────────────────┐
│               LATENCY NUMBERS                               │
│                                                             │
│  OPERATION                        TIME                      │
│  ───────────────────────────────────────                   │
│  L1 cache reference               0.5 ns                    │
│  L2 cache reference               7 ns                      │
│  RAM reference                    100 ns                    │
│  SSD random read                  150 μs                    │
│  HDD seek                         10 ms                     │
│  Same datacenter round trip       0.5 ms                    │
│  Cross-region round trip          50-150 ms                 │
│  Redis GET                        0.5 ms                    │
│  MySQL simple query               1-10 ms                   │
│  HTTP request (same DC)           10-50 ms                  │
│                                                             │
│  Key insight: Network > Disk > Memory > Cache              │
└─────────────────────────────────────────────────────────────┘
```

## The Estimation Framework

### Step 1: Start with Users

```
┌─────────────────────────────────────────────────────────────┐
│            TRAFFIC ESTIMATION FLOW                          │
│                                                             │
│                    DAU                                      │
│                     │                                       │
│                     ▼                                       │
│            Actions per User                                 │
│                     │                                       │
│                     ▼                                       │
│           Total Daily Actions                               │
│                     │                                       │
│                     ▼                                       │
│               ÷ 86,400                                      │
│                     │                                       │
│                     ▼                                       │
│            Average QPS                                      │
│                     │                                       │
│                     ▼                                       │
│             × 2-3 (Peak)                                    │
│                     │                                       │
│                     ▼                                       │
│              Peak QPS                                       │
└─────────────────────────────────────────────────────────────┘
```

### Step 2: Calculate QPS

**Formula:**
```
Average QPS = (DAU × actions_per_user) / seconds_per_day

Peak QPS = Average QPS × peak_factor (usually 2-3x)
```

**Example - Twitter:**
```
Given:
- DAU: 500 million
- Tweets per user per day: 0.1 (most users read, few write)
- Timeline views per user per day: 20

Writes (Tweets):
- Daily tweets: 500M × 0.1 = 50M tweets/day
- Write QPS: 50M / 100,000 = 500 QPS
- Peak write QPS: 500 × 3 = 1,500 QPS

Reads (Timeline):
- Daily views: 500M × 20 = 10B views/day
- Read QPS: 10B / 100,000 = 100,000 QPS
- Peak read QPS: 100,000 × 3 = 300,000 QPS

Read:Write ratio = 100,000 / 500 = 200:1
```

### Step 3: Calculate Storage

**Formula:**
```
Daily storage = Daily new objects × Size per object

Total storage = Daily storage × Retention days × Replication factor
```

**Example - Twitter:**
```
Tweets:
- Daily tweets: 50M
- Size per tweet: 1 KB (text + metadata)
- Daily tweet storage: 50M × 1 KB = 50 GB/day

Media:
- 20% of tweets have images
- Daily images: 10M
- Size per image: 500 KB average
- Daily media storage: 10M × 500 KB = 5 TB/day

5-year total (with 3x replication):
- Tweets: 50 GB × 365 × 5 × 3 = 274 TB
- Media: 5 TB × 365 × 5 × 3 = 27 PB
```

### Step 4: Calculate Bandwidth

**Formula:**
```
Incoming bandwidth = Write QPS × Avg request size
Outgoing bandwidth = Read QPS × Avg response size
```

**Example - Twitter:**
```
Incoming (writes):
- Write QPS: 500
- Avg tweet size: 1 KB
- Incoming: 500 × 1 KB = 500 KB/s = 0.5 MB/s

Outgoing (reads):
- Read QPS: 100,000
- Avg timeline response: 50 KB (50 tweets × 1 KB)
- Outgoing: 100,000 × 50 KB = 5 GB/s

Note: Read bandwidth dominates (as expected for read-heavy)
```

### Step 5: Calculate Cache Size

**Formula:**
```
Cache size = Hot data percentage × Total data size

OR

Cache size = QPS × Avg response size × Cache duration
```

**The 80/20 Rule:**
```
┌─────────────────────────────────────────────────────────────┐
│                  80/20 CACHE RULE                           │
│                                                             │
│  80% of requests hit 20% of data                           │
│                                                             │
│  If total data = 100 GB                                     │
│  Cache 20% = 20 GB                                          │
│  This serves 80% of requests                               │
│                                                             │
│  More aggressive: Cache 10% for 70% hit rate               │
│  More conservative: Cache 30% for 90% hit rate             │
└─────────────────────────────────────────────────────────────┘
```

**Example - Twitter:**
```
Method 1: 80/20 Rule
- Total tweet data: 100 TB
- Cache 20%: 20 TB distributed cache

Method 2: Working Set
- Timeline cache per user: 50 tweets × 1 KB = 50 KB
- Active users at peak: 50M
- Cache needed: 50M × 50 KB = 2.5 TB

We'd likely use a combination:
- User timeline cache: 2.5 TB
- Hot tweets cache: 500 GB
- Total: ~3 TB
```

## Complete Worked Example: URL Shortener

Let's walk through a complete estimation:

```
┌─────────────────────────────────────────────────────────────┐
│           URL SHORTENER - CAPACITY ESTIMATION               │
│                                                             │
│  REQUIREMENTS:                                              │
│  - 500M new URLs per month                                  │
│  - 100:1 read to write ratio                               │
│  - URLs stored for 5 years                                  │
│  - Short URL length: 7 characters                          │
└─────────────────────────────────────────────────────────────┘
```

### Traffic Estimates

```
WRITES (URL shortening):
├── New URLs/month: 500M
├── New URLs/second: 500M / (30 days × 86,400)
│                  = 500M / 2.6M
│                  ≈ 200 URLs/second
└── Peak writes: 200 × 2 = 400 URLs/second

READS (URL redirects):
├── Read:write = 100:1
├── Average reads: 200 × 100 = 20,000/second
└── Peak reads: 20,000 × 2 = 40,000/second

SUMMARY:
┌────────────────┬─────────┬──────────┐
│ Operation      │ Average │ Peak     │
├────────────────┼─────────┼──────────┤
│ Write (create) │ 200/s   │ 400/s    │
│ Read (redirect)│ 20K/s   │ 40K/s    │
└────────────────┴─────────┴──────────┘
```

### Storage Estimates

```
URL RECORD SIZE:
├── Short URL (7 chars): 7 bytes
├── Long URL (average): 200 bytes
├── Created timestamp: 8 bytes
├── Expiration: 8 bytes
├── User ID: 8 bytes
└── Total per URL: ~250 bytes (round to 500 for safety)

TOTAL STORAGE:
├── URLs per month: 500M
├── URLs per year: 6B
├── URLs in 5 years: 30B
├── Storage: 30B × 500 bytes = 15 TB
└── With replication (3x): 45 TB

UNIQUE SHORT URLS NEEDED:
├── 30B URLs over 5 years
├── Base62 encoding (a-z, A-Z, 0-9): 62 characters
├── 7-character URL: 62^7 = 3.5 trillion combinations
└── 30B << 3.5T ✓ (no collision concerns)
```

### Bandwidth Estimates

```
INCOMING (writes):
├── Write QPS: 200
├── Request size: ~250 bytes (mostly the long URL)
└── Incoming bandwidth: 200 × 250 = 50 KB/s

OUTGOING (reads):
├── Read QPS: 20,000
├── Response: redirect (minimal, ~100 bytes)
└── Outgoing bandwidth: 20K × 100 = 2 MB/s

TOTAL BANDWIDTH: ~2.5 MB/s (very manageable)
```

### Cache Estimates

```
CACHE STRATEGY:
├── Most URLs are accessed within 24 hours of creation
├── Then access drops dramatically
├── Cache hot URLs (20% of daily URLs)

CACHE SIZE:
├── Daily new URLs: 500M / 30 = ~17M
├── Hot URLs (20%): 3.4M URLs
├── Size per URL: 500 bytes
├── Daily cache: 3.4M × 500 = 1.7 GB
└── With buffer: ~5 GB cache

CACHE HIT RATIO TARGET: 80%+
```

### Summary Table

```
┌─────────────────────────────────────────────────────────────┐
│          URL SHORTENER - ESTIMATION SUMMARY                 │
│                                                             │
│  METRIC                    VALUE                            │
│  ─────────────────────────────────────                     │
│  Write QPS (avg/peak)      200 / 400                        │
│  Read QPS (avg/peak)       20K / 40K                        │
│  Storage (5 years)         15 TB (45 TB with replication)  │
│  Bandwidth                 ~2.5 MB/s                        │
│  Cache size                5 GB                             │
│  Unique URLs supported     3.5 trillion                     │
│                                                             │
│  ARCHITECTURE IMPLICATIONS:                                 │
│  ├── Single database can handle write QPS                  │
│  ├── Need multiple read replicas for read QPS              │
│  ├── Redis cluster for caching                             │
│  └── CDN optional (responses are tiny)                     │
└─────────────────────────────────────────────────────────────┘
```

## Complete Worked Example: WhatsApp

```
┌─────────────────────────────────────────────────────────────┐
│            WHATSAPP - CAPACITY ESTIMATION                   │
│                                                             │
│  REQUIREMENTS:                                              │
│  - 2B total users                                           │
│  - 500M DAU                                                 │
│  - 50 messages sent per user per day                       │
│  - Support text, images, videos                            │
│  - Message delivery < 500ms                                 │
│  - Messages stored for 30 days                             │
└─────────────────────────────────────────────────────────────┘
```

### Traffic Estimates

```
MESSAGES:
├── DAU: 500M
├── Messages per user: 50/day
├── Total messages/day: 25B messages
├── Messages/second: 25B / 86,400 ≈ 290,000/second
└── Peak (2x): 580,000/second

MESSAGE TYPES (assumed distribution):
├── Text: 70% = 175K/s
├── Images: 25% = 70K/s
└── Videos: 5% = 15K/s

CONNECTION HANDLING:
├── DAU: 500M
├── Concurrent users (20% of DAU): 100M
└── WebSocket connections: 100M persistent connections
```

### Storage Estimates

```
MESSAGE SIZES:
├── Text message: 100 bytes
├── Image: 200 KB (compressed)
├── Video: 5 MB (1 min, compressed)

DAILY STORAGE:
├── Text: 25B × 0.7 × 100 bytes = 1.75 TB
├── Images: 25B × 0.25 × 200 KB = 1.25 PB
├── Videos: 25B × 0.05 × 5 MB = 6.25 PB
└── Total daily: ~7.5 PB

30-DAY RETENTION:
├── Total: 7.5 PB × 30 = 225 PB
└── With 3x replication: 675 PB (0.67 EB)

This is MASSIVE storage!
```

### Bandwidth Estimates

```
OUTGOING (message delivery):
├── Message QPS: 290,000
├── Avg message size: ~200 KB (weighted by type)
└── Bandwidth: 290K × 200 KB = 58 GB/s = 464 Gbps

This requires significant network infrastructure!

PER DATA CENTER (10 data centers):
├── 46 Gbps per DC
└── Requires multiple 10G+ links
```

### Server Estimates

```
CONNECTION SERVERS (WebSocket):
├── Connections to handle: 100M concurrent
├── Connections per server: 100K (high-end server)
└── Servers needed: 100M / 100K = 1,000 servers

MESSAGE PROCESSING SERVERS:
├── Peak QPS: 580,000
├── QPS per server: 10,000 (with processing)
└── Servers needed: 580K / 10K = 58 servers

DATABASE SERVERS:
├── Write throughput needed: 58 GB/s
├── Per Cassandra node: 50 MB/s
└── Nodes needed: 1,160 nodes (distributed globally)
```

### Summary Table

```
┌─────────────────────────────────────────────────────────────┐
│           WHATSAPP - ESTIMATION SUMMARY                     │
│                                                             │
│  METRIC                    VALUE                            │
│  ─────────────────────────────────────                     │
│  Messages/second (peak)    580,000                          │
│  Concurrent connections    100 million                      │
│  Daily storage             7.5 PB                           │
│  Total storage (30 days)   675 PB                           │
│  Bandwidth                 464 Gbps                         │
│                                                             │
│  INFRASTRUCTURE:                                            │
│  ├── Connection servers    1,000+                          │
│  ├── Processing servers    60+                             │
│  ├── Database nodes        1,200+                          │
│  └── Data centers          10+ globally                    │
│                                                             │
│  KEY CHALLENGES:                                            │
│  ├── Massive storage for media                             │
│  ├── Real-time message delivery                            │
│  ├── 100M concurrent WebSocket connections                 │
│  └── Global message routing                                │
└─────────────────────────────────────────────────────────────┘
```

## Common Estimation Patterns

### Pattern 1: Social Media Read-Heavy

```
Typical ratios:
- Read:Write = 100:1 to 1000:1
- 90% reads from cache
- Fan-out on read vs write decision

Twitter example:
- 100K read QPS
- 500 write QPS
- 200:1 ratio
```

### Pattern 2: Messaging Real-Time

```
Typical characteristics:
- Many concurrent connections
- Small, frequent messages
- Delivery guarantee needed

WhatsApp example:
- 100M connections
- 290K messages/second
- Sub-second delivery
```

### Pattern 3: Storage-Heavy Media

```
Typical characteristics:
- Large objects (images, videos)
- CDN-heavy
- Async processing

Instagram example:
- 500M images uploaded/day
- 2 PB daily storage
- CDN serves 99% of reads
```

## Quick Reference Formulas

```
┌─────────────────────────────────────────────────────────────┐
│               QUICK ESTIMATION FORMULAS                     │
│                                                             │
│  QPS:                                                       │
│  Average QPS = (DAU × actions/user) / 86,400               │
│  Peak QPS = Average QPS × 2 to 3                           │
│                                                             │
│  STORAGE:                                                   │
│  Daily = objects/day × size                                 │
│  Total = Daily × days × replication_factor                 │
│                                                             │
│  BANDWIDTH:                                                 │
│  = QPS × object_size                                        │
│                                                             │
│  SERVERS:                                                   │
│  = Peak QPS / QPS_per_server                               │
│                                                             │
│  CACHE:                                                     │
│  = Total data × 0.2 (80/20 rule)                          │
│  OR = Working set size                                      │
│                                                             │
│  SHARDS:                                                    │
│  = Total data / data_per_shard                             │
└─────────────────────────────────────────────────────────────┘
```

## Estimation Mistakes to Avoid

### Mistake 1: False Precision
```
❌ "We need exactly 847,291 QPS"
✅ "We need roughly 850K QPS, let's plan for 1M"

Always round to nice numbers!
```

### Mistake 2: Forgetting Replication
```
❌ "We need 10TB storage"
✅ "We need 10TB, so 30TB with 3x replication"
```

### Mistake 3: Ignoring Peak vs Average
```
❌ "Average is 10K QPS, so 10 servers"
✅ "Peak is 30K QPS, so 30 servers + 20% buffer"
```

### Mistake 4: Wrong Order of Magnitude
```
❌ Confusing MB and GB
❌ Confusing thousands and millions

Double-check your powers of 10!
```

## Practice Problems

Try estimating these yourself:

```
1. YouTube
   - 2B monthly users
   - Average 5 videos watched per day
   - Average video: 10 minutes, 720p

2. Uber
   - 20M rides per day
   - Average ride: 15 minutes
   - Location updates: every 4 seconds

3. Gmail
   - 1.8B users
   - 50 emails sent per user per day
   - Average email: 50KB with attachments
```

---

**Next:** [04_API_Design.md](./04_API_Design.md) - Design clean, scalable APIs
