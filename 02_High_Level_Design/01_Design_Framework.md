# Step-by-Step System Design Framework

## The Problem with "Winging It"

Most candidates fail system design interviews not because they lack knowledge, but because they lack **structure**. They jump around randomly, forget important aspects, and run out of time.

This framework ensures you cover everything systematically.

## The RESHADED Framework

A memorable acronym for your design process:

```
R - Requirements
E - Estimation
S - Storage schema
H - High-level design
A - APIs
D - Detailed design
E - Evaluate (tradeoffs)
D - Discuss improvements
```

## Detailed Breakdown

### Phase 1: Requirements (4-5 minutes)

**Goal:** Understand what you're building before you build it.

```
┌─────────────────────────────────────────────────────────────┐
│                    REQUIREMENTS PHASE                       │
│                                                             │
│  FUNCTIONAL (What)              NON-FUNCTIONAL (How well)   │
│  ├── Core features              ├── Scale (users, data)     │
│  ├── User actions               ├── Latency requirements    │
│  ├── Data inputs/outputs        ├── Availability (99.9%?)   │
│  └── Edge cases                 └── Consistency needs       │
│                                                             │
│  ASK: "What's in scope? What's out of scope?"              │
└─────────────────────────────────────────────────────────────┘
```

**Questions to Ask:**
- "Who are the users? How many?"
- "What's the most important feature?"
- "Do we need real-time or near-real-time?"
- "Is consistency or availability more important?"
- "Any geographic requirements?"

**Example for Twitter:**
```
Functional:
✓ Post tweets (text, images)
✓ Follow/unfollow users
✓ View home timeline
✓ Search tweets
✗ Direct messages (out of scope)
✗ Analytics (out of scope)

Non-Functional:
- 500M daily active users
- Read-heavy (100:1 read:write ratio)
- Timeline loads < 200ms
- 99.99% availability
- Eventual consistency acceptable
```

### Phase 2: Capacity Estimation (3-5 minutes)

**Goal:** Understand the scale to make informed architecture decisions.

```
┌─────────────────────────────────────────────────────────────┐
│                 CAPACITY ESTIMATION                         │
│                                                             │
│  START WITH:                                                │
│  └── Daily Active Users (DAU)                              │
│                                                             │
│  DERIVE:                                                    │
│  ├── Queries Per Second (QPS)                              │
│  ├── Storage requirements                                   │
│  ├── Bandwidth needs                                        │
│  └── Memory/Cache sizing                                    │
│                                                             │
│  REMEMBER: Round numbers for simplicity!                    │
└─────────────────────────────────────────────────────────────┘
```

**Quick Math Template:**
```
Users:
- DAU: 100M
- Requests/user/day: 10
- Total requests/day: 1B

QPS:
- 1B requests / 86,400 seconds ≈ 12,000 QPS
- Peak (2x average): 24,000 QPS

Storage (5 years):
- New data/day: 100M users × 1KB = 100GB
- 5 years: 100GB × 365 × 5 = 180TB

Bandwidth:
- 12,000 QPS × 10KB response = 120 MB/s
```

### Phase 3: High-Level Design (10-15 minutes)

**Goal:** Draw the architecture showing all major components.

```
┌─────────────────────────────────────────────────────────────┐
│              HIGH-LEVEL DESIGN TEMPLATE                     │
│                                                             │
│                      ┌─────────┐                           │
│                      │   CDN   │                           │
│                      └────┬────┘                           │
│                           │                                 │
│  ┌──────────┐       ┌────▼────┐       ┌──────────┐        │
│  │  Mobile  │──────▶│   LB    │──────▶│   API    │        │
│  │   App    │       │         │       │ Gateway  │        │
│  └──────────┘       └────┬────┘       └────┬─────┘        │
│                          │                  │              │
│  ┌──────────┐            │            ┌────▼─────┐        │
│  │   Web    │────────────┘            │ Services │        │
│  │  Client  │                         └────┬─────┘        │
│  └──────────┘                              │              │
│                                            │              │
│                    ┌───────────────────────┼───────┐      │
│                    │                       │       │      │
│              ┌─────▼─────┐          ┌─────▼─────┐ │      │
│              │   Cache   │          │  Message  │ │      │
│              │  (Redis)  │          │   Queue   │ │      │
│              └───────────┘          └───────────┘ │      │
│                    │                       │       │      │
│              ┌─────▼─────┐          ┌─────▼─────┐ │      │
│              │  Primary  │          │   Async   │ │      │
│              │    DB     │◀────────▶│  Workers  │ │      │
│              └───────────┘          └───────────┘ │      │
│                    │                              │      │
│              ┌─────▼─────┐                        │      │
│              │  Replica  │                        │      │
│              │    DB     │                        │      │
│              └───────────┘                        │      │
└─────────────────────────────────────────────────────────────┘
```

**Component Selection Checklist:**
- [ ] Load Balancer - Do we have multiple servers?
- [ ] API Gateway - Do we have multiple services?
- [ ] Cache - Do we have read-heavy workload?
- [ ] CDN - Do we serve static content globally?
- [ ] Message Queue - Do we need async processing?
- [ ] Search Engine - Do we need full-text search?
- [ ] Object Storage - Do we store files/media?

### Phase 4: API Design (3-5 minutes)

**Goal:** Define the contract between clients and servers.

```
┌─────────────────────────────────────────────────────────────┐
│                    API DESIGN                               │
│                                                             │
│  REST TEMPLATE:                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ POST /api/v1/tweets                                 │   │
│  │ Body: { "content": "...", "media_ids": [...] }     │   │
│  │ Response: { "tweet_id": "123", "created_at": "..." }│   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  KEY DECISIONS:                                             │
│  - REST vs GraphQL vs gRPC                                  │
│  - Pagination strategy                                      │
│  - Authentication method                                    │
│  - Rate limiting                                            │
│  - Versioning approach                                      │
└─────────────────────────────────────────────────────────────┘
```

### Phase 5: Data Model (3-5 minutes)

**Goal:** Design schemas that support your access patterns.

```
┌─────────────────────────────────────────────────────────────┐
│                   DATA MODEL                                │
│                                                             │
│  THINK ABOUT:                                               │
│  1. What entities exist?                                    │
│  2. How do they relate?                                     │
│  3. What are the access patterns?                          │
│  4. SQL or NoSQL?                                          │
│                                                             │
│  EXAMPLE - Twitter:                                         │
│                                                             │
│  Users                    Tweets                            │
│  ┌────────────────┐      ┌──────────────────┐              │
│  │ user_id (PK)   │      │ tweet_id (PK)    │              │
│  │ username       │      │ user_id (FK)     │              │
│  │ email          │      │ content          │              │
│  │ created_at     │      │ media_urls       │              │
│  └────────────────┘      │ created_at       │              │
│          │               │ like_count       │              │
│          │               └──────────────────┘              │
│          │                                                  │
│          ▼                                                  │
│  Follows                                                    │
│  ┌────────────────┐                                        │
│  │ follower_id    │                                        │
│  │ followee_id    │                                        │
│  │ created_at     │                                        │
│  └────────────────┘                                        │
└─────────────────────────────────────────────────────────────┘
```

### Phase 6: Deep Dive (10-15 minutes)

**Goal:** Address the hardest parts of the system.

The interviewer will typically guide you to specific areas:
- How does the timeline work?
- How do you handle millions of followers?
- What happens when a celebrity tweets?

```
┌─────────────────────────────────────────────────────────────┐
│                   DEEP DIVE AREAS                           │
│                                                             │
│  COMMON DEEP DIVES:                                         │
│                                                             │
│  1. SCALING                                                 │
│     ├── Horizontal scaling strategy                         │
│     ├── Sharding approach                                   │
│     └── Caching strategy                                    │
│                                                             │
│  2. RELIABILITY                                             │
│     ├── Failure handling                                    │
│     ├── Data replication                                    │
│     └── Disaster recovery                                   │
│                                                             │
│  3. SPECIAL CASES                                           │
│     ├── Hot spots (celebrity users)                         │
│     ├── Viral content                                       │
│     └── Peak traffic                                        │
│                                                             │
│  4. DATA FLOW                                               │
│     ├── Write path                                          │
│     ├── Read path                                           │
│     └── Consistency guarantees                              │
└─────────────────────────────────────────────────────────────┘
```

### Phase 7: Tradeoffs & Wrap-up (2-3 minutes)

**Goal:** Show mature engineering thinking.

```
┌─────────────────────────────────────────────────────────────┐
│                 TRADEOFFS TO DISCUSS                        │
│                                                             │
│  CONSISTENCY vs AVAILABILITY                                │
│  "We chose eventual consistency because availability is     │
│   more important for a social media feed."                  │
│                                                             │
│  COST vs PERFORMANCE                                        │
│  "Caching adds complexity but reduces DB load by 90%."     │
│                                                             │
│  SIMPLICITY vs SCALABILITY                                  │
│  "We could start with a monolith and extract services       │
│   as we identify bottlenecks."                              │
│                                                             │
│  LATENCY vs THROUGHPUT                                      │
│  "Batching increases throughput but adds latency."          │
└─────────────────────────────────────────────────────────────┘
```

## Time Management by Interview Length

### 30-Minute Interview
```
Requirements:     3 min
Estimation:       2 min
High-level:       8 min
API/Data Model:   5 min
Deep Dive:        8 min
Wrap-up:          2 min
Buffer:           2 min
```

### 45-Minute Interview
```
Requirements:     5 min
Estimation:       4 min
High-level:      10 min
API/Data Model:   6 min
Deep Dive:       15 min
Wrap-up:          3 min
Buffer:           2 min
```

### 60-Minute Interview
```
Requirements:     5 min
Estimation:       5 min
High-level:      12 min
API/Data Model:   8 min
Deep Dive:       22 min
Wrap-up:          5 min
Buffer:           3 min
```

## Common Mistakes to Avoid

### 1. Starting Without Requirements
```
❌ "Let me draw the architecture..."
✅ "Before I start, let me clarify the requirements..."
```

### 2. Over-Engineering Early
```
❌ "We need Kubernetes with service mesh and..."
✅ "Let's start simple and scale as needed..."
```

### 3. Ignoring Numbers
```
❌ "We'll use sharding because it's good practice"
✅ "With 10TB of data, we need sharding across 50 nodes"
```

### 4. Not Justifying Decisions
```
❌ "We'll use Redis"
✅ "We'll use Redis because our read:write ratio is 100:1
    and we need sub-millisecond latency for hot data"
```

### 5. Getting Lost in Details
```
❌ Spending 10 minutes on authentication implementation
✅ "We'll use JWT tokens, moving on to the core problem..."
```

## The Confident Candidate Template

Use these phrases to sound confident:

**Starting:**
> "Before diving in, I'd like to clarify a few requirements..."

**Estimating:**
> "Let me do some quick back-of-envelope calculations..."

**Designing:**
> "I'll walk you through my high-level architecture, then we can dive deeper into any area..."

**Making Decisions:**
> "I'm choosing X over Y because [specific reason]..."

**Acknowledging Uncertainty:**
> "There are tradeoffs here. We could also do Z if [condition]..."

**Wrapping Up:**
> "To summarize, the key tradeoffs are... and potential improvements include..."

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────┐
│              SYSTEM DESIGN CHEAT SHEET                      │
│                                                             │
│  1. REQUIREMENTS                                            │
│     □ Functional requirements (3-5 core features)           │
│     □ Non-functional (scale, latency, availability)         │
│     □ Scope boundaries                                      │
│                                                             │
│  2. ESTIMATION                                              │
│     □ DAU → QPS → Peak QPS                                 │
│     □ Storage (daily × retention)                           │
│     □ Bandwidth (QPS × response size)                       │
│                                                             │
│  3. DESIGN                                                  │
│     □ Architecture diagram                                  │
│     □ Component justification                               │
│     □ Data flow (read/write paths)                         │
│                                                             │
│  4. APIS                                                    │
│     □ Core endpoints                                        │
│     □ Request/response format                               │
│     □ Pagination strategy                                   │
│                                                             │
│  5. DATA MODEL                                              │
│     □ Entity relationships                                  │
│     □ Access patterns                                       │
│     □ Indexing strategy                                     │
│                                                             │
│  6. DEEP DIVE                                               │
│     □ Scaling strategy                                      │
│     □ Failure handling                                      │
│     □ Bottleneck solutions                                  │
│                                                             │
│  7. TRADEOFFS                                               │
│     □ What we sacrificed                                    │
│     □ Alternative approaches                                │
│     □ Future improvements                                   │
└─────────────────────────────────────────────────────────────┘
```

---

**Next:** [02_Requirements_Gathering.md](./02_Requirements_Gathering.md) - Master the art of asking the right questions
