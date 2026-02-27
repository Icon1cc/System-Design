# HLD Example: Twitter System Design

## The Interview Scenario

**Interviewer:** "Design Twitter."

This is a classic system design question. Let's walk through a complete, interview-ready solution.

---

## Step 1: Requirements Gathering (5 minutes)

### Clarifying Questions

```
YOU: "Before I start designing, I'd like to clarify the requirements.
     First, what features should I focus on?"

INTERVIEWER: "Core Twitter functionality - tweets, timeline, follow."

YOU: "Should I include search, direct messages, notifications?"

INTERVIEWER: "Include basic search. DMs are out of scope."

YOU: "What scale should I design for?"

INTERVIEWER: "Let's say 500 million daily active users."

YOU: "What are the latency requirements?"

INTERVIEWER: "Timeline should load under 200ms."

YOU: "Is eventual consistency acceptable for the timeline?"

INTERVIEWER: "Yes, a few seconds delay is fine."
```

### Requirements Summary

```
┌─────────────────────────────────────────────────────────────┐
│                TWITTER REQUIREMENTS                         │
│                                                             │
│  FUNCTIONAL REQUIREMENTS:                                   │
│  ✓ Post tweets (text, up to 280 chars, optional media)    │
│  ✓ Follow/unfollow users                                   │
│  ✓ Home timeline (tweets from followed users)             │
│  ✓ User timeline (user's own tweets)                      │
│  ✓ Like tweets                                             │
│  ✓ Basic search                                            │
│  ✗ Direct messages (out of scope)                         │
│  ✗ Trending topics (out of scope)                         │
│                                                             │
│  NON-FUNCTIONAL REQUIREMENTS:                               │
│  - 500M DAU                                                 │
│  - Timeline latency < 200ms                                │
│  - High availability (99.99%)                              │
│  - Eventual consistency acceptable                         │
│  - Global user base                                        │
│                                                             │
│  ASSUMPTIONS:                                               │
│  - Read-heavy system (100:1 ratio)                        │
│  - Average user follows 200 users                         │
│  - 10% users tweet daily, average 2 tweets                │
│  - Celebrity users exist (millions of followers)          │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 2: Capacity Estimation (5 minutes)

### Traffic Estimates

```
┌─────────────────────────────────────────────────────────────┐
│              TRAFFIC ESTIMATION                             │
│                                                             │
│  USERS:                                                     │
│  - DAU: 500M                                               │
│  - Total users: ~1.5B                                      │
│                                                             │
│  TWEETS:                                                    │
│  - 10% tweet daily: 50M users                             │
│  - Average 2 tweets/user: 100M tweets/day                 │
│  - Tweets/second: 100M / 86,400 ≈ 1,200 tweets/sec       │
│  - Peak (2x): 2,400 tweets/sec                            │
│                                                             │
│  TIMELINE VIEWS:                                            │
│  - Average user views timeline 20x/day                    │
│  - Total: 500M × 20 = 10B timeline views/day              │
│  - Timeline QPS: 10B / 86,400 ≈ 115,000 QPS              │
│  - Peak (2x): 230,000 QPS                                 │
│                                                             │
│  READ:WRITE RATIO:                                          │
│  - 115,000 reads / 1,200 writes ≈ 100:1                  │
│  - Confirmed: This is a read-heavy system                 │
└─────────────────────────────────────────────────────────────┘
```

### Storage Estimates

```
┌─────────────────────────────────────────────────────────────┐
│              STORAGE ESTIMATION                             │
│                                                             │
│  TWEET SIZE:                                                │
│  - tweet_id: 8 bytes                                       │
│  - user_id: 8 bytes                                        │
│  - content: 280 bytes (280 chars)                         │
│  - created_at: 8 bytes                                     │
│  - metadata: ~200 bytes                                    │
│  - Total: ~500 bytes per tweet                            │
│                                                             │
│  DAILY TWEET STORAGE:                                       │
│  - 100M tweets × 500 bytes = 50 GB/day                    │
│                                                             │
│  5-YEAR STORAGE (tweets only):                             │
│  - 50 GB × 365 × 5 = 91 TB                                │
│  - With 3x replication: 273 TB                            │
│                                                             │
│  MEDIA STORAGE:                                             │
│  - 20% tweets have images: 20M images/day                 │
│  - Average image: 500 KB                                   │
│  - Daily: 20M × 500 KB = 10 TB/day                        │
│  - 5 years: 10 TB × 365 × 5 = 18 PB                       │
│                                                             │
│  FOLLOW RELATIONSHIPS:                                      │
│  - 1.5B users × 200 avg follows = 300B edges             │
│  - Each edge: 16 bytes (two user_ids)                    │
│  - Total: 4.8 TB                                           │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 3: High-Level Design (15 minutes)

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       TWITTER HIGH-LEVEL ARCHITECTURE                        │
│                                                                              │
│    ┌─────────────────────────────────────────────────────────────────────┐  │
│    │                              CDN                                     │  │
│    │                    (Static assets, images)                          │  │
│    └────────────────────────────────┬────────────────────────────────────┘  │
│                                     │                                        │
│    ┌────────────────────────────────▼────────────────────────────────────┐  │
│    │                         LOAD BALANCER                                │  │
│    │                      (Geographic routing)                           │  │
│    └────────────────────────────────┬────────────────────────────────────┘  │
│                                     │                                        │
│    ┌────────────────────────────────▼────────────────────────────────────┐  │
│    │                          API GATEWAY                                 │  │
│    │              (Auth, rate limiting, routing)                         │  │
│    └────────────┬───────────────────┬───────────────────┬────────────────┘  │
│                 │                   │                   │                    │
│         ┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐          │
│         │    User       │   │    Tweet      │   │   Timeline    │          │
│         │   Service     │   │   Service     │   │   Service     │          │
│         └───────┬───────┘   └───────┬───────┘   └───────┬───────┘          │
│                 │                   │                   │                    │
│    ┌────────────┼───────────────────┼───────────────────┼────────────────┐  │
│    │            │                   │                   │                │  │
│    │    ┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐       │  │
│    │    │   User DB     │   │  Tweet DB     │   │ Timeline Cache│       │  │
│    │    │  (MySQL)      │   │  (Sharded)    │   │   (Redis)     │       │  │
│    │    └───────────────┘   └───────────────┘   └───────────────┘       │  │
│    │                                                                     │  │
│    │    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐       │  │
│    │    │  Follow DB    │   │  Search       │   │ Media Storage │       │  │
│    │    │  (Graph/SQL)  │   │(Elasticsearch)│   │    (S3)       │       │  │
│    │    └───────────────┘   └───────────────┘   └───────────────┘       │  │
│    │                                                                     │  │
│    └─────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│    ┌─────────────────────────────────────────────────────────────────────┐  │
│    │                        MESSAGE QUEUE (Kafka)                         │  │
│    │              (Async fan-out, notifications, analytics)              │  │
│    └─────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Core Components

```
┌─────────────────────────────────────────────────────────────┐
│                 COMPONENT BREAKDOWN                          │
│                                                             │
│  1. API GATEWAY                                              │
│     - Authentication (JWT tokens)                           │
│     - Rate limiting (per user)                             │
│     - Request routing                                       │
│     - SSL termination                                       │
│                                                             │
│  2. USER SERVICE                                             │
│     - User registration/login                               │
│     - Profile management                                    │
│     - Follow/unfollow operations                           │
│     - Stores: User DB (MySQL), Follow Graph                │
│                                                             │
│  3. TWEET SERVICE                                            │
│     - Create/delete tweets                                  │
│     - Store tweet content                                   │
│     - Handle media uploads                                  │
│     - Stores: Tweet DB (sharded), S3 for media            │
│                                                             │
│  4. TIMELINE SERVICE                                         │
│     - Generate home timeline                               │
│     - Serve cached timelines                               │
│     - Handle fan-out                                        │
│     - Stores: Timeline Cache (Redis)                       │
│                                                             │
│  5. SEARCH SERVICE                                           │
│     - Index tweets for search                              │
│     - Handle search queries                                │
│     - Stores: Elasticsearch                                │
│                                                             │
│  6. NOTIFICATION SERVICE                                     │
│     - Push notifications                                    │
│     - Mention detection                                    │
│     - Async via Kafka                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 4: API Design

```
┌─────────────────────────────────────────────────────────────┐
│                    TWITTER API                              │
│                                                             │
│  TWEETS:                                                    │
│  ────────────────────────────────────────────              │
│  POST /api/v1/tweets                                       │
│  Body: {"content": "Hello!", "media_ids": ["abc"]}        │
│  Response: {"tweet_id": "123", "created_at": "..."}       │
│                                                             │
│  GET /api/v1/tweets/{tweet_id}                            │
│  Response: {"tweet_id": "123", "content": "...", ...}     │
│                                                             │
│  DELETE /api/v1/tweets/{tweet_id}                         │
│  Response: 204 No Content                                  │
│                                                             │
│  TIMELINE:                                                  │
│  ────────────────────────────────────────────              │
│  GET /api/v1/timeline/home?cursor=X&limit=20              │
│  Response: {                                               │
│    "tweets": [...],                                        │
│    "next_cursor": "eyJ..."                                │
│  }                                                          │
│                                                             │
│  GET /api/v1/timeline/user/{user_id}?cursor=X&limit=20    │
│                                                             │
│  FOLLOW:                                                    │
│  ────────────────────────────────────────────              │
│  POST /api/v1/users/{user_id}/follow                      │
│  DELETE /api/v1/users/{user_id}/follow                    │
│                                                             │
│  GET /api/v1/users/{user_id}/followers?cursor=X           │
│  GET /api/v1/users/{user_id}/following?cursor=X           │
│                                                             │
│  SEARCH:                                                    │
│  ────────────────────────────────────────────              │
│  GET /api/v1/search?q=keyword&cursor=X&limit=20           │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 5: Data Model

### Database Schema

```
┌─────────────────────────────────────────────────────────────┐
│                    DATA MODEL                               │
│                                                             │
│  USERS TABLE (MySQL - not sharded)                         │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ user_id        BIGINT       PRIMARY KEY              │ │
│  │ username       VARCHAR(30)  UNIQUE                   │ │
│  │ email          VARCHAR(255) UNIQUE                   │ │
│  │ display_name   VARCHAR(50)                           │ │
│  │ bio            TEXT                                  │ │
│  │ avatar_url     VARCHAR(500)                          │ │
│  │ follower_count INT          DEFAULT 0               │ │
│  │ following_count INT         DEFAULT 0               │ │
│  │ created_at     TIMESTAMP                            │ │
│  │                                                       │ │
│  │ INDEX (username)                                     │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  TWEETS TABLE (Sharded by user_id)                         │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ tweet_id       BIGINT       PRIMARY KEY              │ │
│  │ user_id        BIGINT       (shard key)              │ │
│  │ content        VARCHAR(280)                          │ │
│  │ media_urls     JSON                                  │ │
│  │ like_count     INT          DEFAULT 0               │ │
│  │ retweet_count  INT          DEFAULT 0               │ │
│  │ created_at     TIMESTAMP                            │ │
│  │                                                       │ │
│  │ INDEX (user_id, created_at DESC)                    │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  FOLLOWS TABLE (Sharded by follower_id)                   │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ follower_id    BIGINT                                │ │
│  │ followee_id    BIGINT                                │ │
│  │ created_at     TIMESTAMP                            │ │
│  │                                                       │ │
│  │ PRIMARY KEY (follower_id, followee_id)              │ │
│  │ INDEX (followee_id) -- for "my followers" query     │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  TIMELINE CACHE (Redis)                                    │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ Key: timeline:{user_id}                             │ │
│  │ Value: Sorted Set of tweet_ids, scored by timestamp │ │
│  │ TTL: 24 hours                                        │ │
│  │ Max size: 800 tweets per user                       │ │
│  └──────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 6: Deep Dive - Timeline Generation

This is the most complex part of Twitter. Let's explore it in detail.

### The Core Problem

```
┌─────────────────────────────────────────────────────────────┐
│              THE TIMELINE PROBLEM                           │
│                                                             │
│  User A follows 500 users                                  │
│  User A requests home timeline                             │
│                                                             │
│  NAIVE APPROACH (Fan-out on Read):                         │
│  1. Get list of 500 followed users                        │
│  2. For each user, fetch their recent tweets              │
│  3. Merge all tweets, sort by time                        │
│  4. Return top 20                                          │
│                                                             │
│  PROBLEM:                                                   │
│  - 500+ database queries per timeline request!            │
│  - At 115K QPS = 57.5 million DB queries/second          │
│  - Latency: 500ms+ (too slow)                             │
│                                                             │
│  WE NEED A BETTER SOLUTION                                 │
└─────────────────────────────────────────────────────────────┘
```

### Solution: Fan-Out on Write (Push Model)

```
┌─────────────────────────────────────────────────────────────┐
│              FAN-OUT ON WRITE                               │
│                                                             │
│  When User A posts a tweet:                                │
│                                                             │
│  1. Store tweet in Tweet DB                                │
│                                                             │
│  2. Get User A's follower list (e.g., 1000 followers)     │
│                                                             │
│  3. For EACH follower, add tweet_id to their timeline cache│
│                                                             │
│     ┌─────────────────────────────────────────────────┐   │
│     │ User A posts tweet_123                          │   │
│     │           │                                      │   │
│     │           ▼                                      │   │
│     │    Get followers: [B, C, D, E, ...]             │   │
│     │           │                                      │   │
│     │    ┌──────┴──────┬──────────┬──────────┐       │   │
│     │    ▼             ▼          ▼          ▼       │   │
│     │ timeline:B   timeline:C  timeline:D  ...       │   │
│     │ ADD tweet_123 to each                          │   │
│     └─────────────────────────────────────────────────┘   │
│                                                             │
│  When User B requests timeline:                            │
│  1. Read from timeline:B cache (1 Redis read!)            │
│  2. Return cached tweet_ids                               │
│  3. Hydrate with tweet content                            │
│                                                             │
│  RESULT:                                                    │
│  - Reads are O(1) instead of O(N)                         │
│  - Timeline latency < 50ms                                │
│  - Trade-off: Writes are more expensive                   │
└─────────────────────────────────────────────────────────────┘
```

### The Celebrity Problem

```
┌─────────────────────────────────────────────────────────────┐
│              THE CELEBRITY PROBLEM                          │
│                                                             │
│  Celebrity user has 50 million followers                   │
│  When they tweet:                                          │
│                                                             │
│  Fan-out on write means:                                   │
│  - 50 million Redis writes!                                │
│  - Takes minutes to complete                               │
│  - Huge spike in write load                                │
│                                                             │
│  SOLUTION: Hybrid Approach                                  │
│  ─────────────────────────────────────────                 │
│  1. Define "celebrity" threshold (e.g., > 10K followers)  │
│                                                             │
│  2. For NORMAL users: Fan-out on write (push)             │
│     - Their tweets go to all followers' caches            │
│                                                             │
│  3. For CELEBRITY users: Fan-out on read (pull)           │
│     - Their tweets stored separately                       │
│     - Merged at read time                                  │
│                                                             │
│  TIMELINE READ (Hybrid):                                    │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ 1. Get pre-computed timeline from cache (fast)      │  │
│  │                                                       │  │
│  │ 2. Get list of celebrities I follow                 │  │
│  │                                                       │  │
│  │ 3. Fetch recent tweets from each celebrity          │  │
│  │    (small number - maybe 5-10 celebrities)          │  │
│  │                                                       │  │
│  │ 4. Merge and sort                                    │  │
│  │                                                       │  │
│  │ 5. Return combined timeline                          │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  WHY THIS WORKS:                                            │
│  - Most users follow few celebrities                       │
│  - Celebrity tweet fetch is bounded (not 500 queries)     │
│  - Merging is in-memory, very fast                        │
└─────────────────────────────────────────────────────────────┘
```

### Timeline Architecture Detail

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    TIMELINE WRITE PATH                                       │
│                                                                              │
│   User posts          Tweet            Fan-out                Timeline       │
│    tweet              Service          Service                Cache          │
│      │                   │                 │                    │            │
│      │  POST /tweets     │                 │                    │            │
│      │──────────────────▶│                 │                    │            │
│      │                   │                 │                    │            │
│      │                   │ Store tweet     │                    │            │
│      │                   │────────────────▶│                    │            │
│      │                   │        (Tweet DB)                   │            │
│      │                   │                 │                    │            │
│      │                   │ Publish event   │                    │            │
│      │                   │────────────────▶│                    │            │
│      │                   │      (Kafka)    │                    │            │
│      │                   │                 │                    │            │
│      │  201 Created      │                 │ Get followers     │            │
│      │◀──────────────────│                 │───────────────────│            │
│      │                   │                 │ (Follow DB)       │            │
│      │                   │                 │                    │            │
│      │                   │                 │ Is celebrity?     │            │
│      │                   │                 │ NO: Fan-out       │            │
│      │                   │                 │                    │            │
│      │                   │                 │ Add to each       │            │
│      │                   │                 │ follower's cache  │            │
│      │                   │                 │───────────────────▶│            │
│      │                   │                 │                    │            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                    TIMELINE READ PATH                                        │
│                                                                              │
│   User              Timeline           Timeline          Tweet               │
│                     Service            Cache             Service             │
│      │                 │                  │                 │                │
│      │ GET /timeline   │                  │                 │                │
│      │────────────────▶│                  │                 │                │
│      │                 │                  │                 │                │
│      │                 │ Get cached IDs   │                 │                │
│      │                 │─────────────────▶│                 │                │
│      │                 │                  │                 │                │
│      │                 │ [id1, id2, ...]  │                 │                │
│      │                 │◀─────────────────│                 │                │
│      │                 │                  │                 │                │
│      │                 │ Get celebrity tweets (if any)     │                │
│      │                 │─────────────────────────────────────▶│             │
│      │                 │                                      │             │
│      │                 │ Merge + hydrate tweet content       │             │
│      │                 │◀─────────────────────────────────────│             │
│      │                 │                  │                 │                │
│      │ Timeline data   │                  │                 │                │
│      │◀────────────────│                  │                 │                │
│      │                 │                  │                 │                │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Step 7: Scaling Strategy

### Database Sharding

```
┌─────────────────────────────────────────────────────────────┐
│              SHARDING STRATEGY                              │
│                                                             │
│  TWEETS: Shard by user_id                                  │
│  ─────────────────────────────────────────                 │
│  - All tweets from same user on same shard                │
│  - User timeline query hits single shard                  │
│  - 100 shards for 100M tweets                             │
│                                                             │
│  shard_id = hash(user_id) % 100                           │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                   │
│  │ Shard 0 │  │ Shard 1 │  │Shard 99 │                   │
│  │Users    │  │Users    │  │Users    │                   │
│  │0-99     │  │100-199  │  │9900-9999│                   │
│  └─────────┘  └─────────┘  └─────────┘                   │
│                                                             │
│  FOLLOWS: Shard by follower_id                            │
│  ─────────────────────────────────────────                 │
│  - "Who do I follow?" = single shard lookup               │
│  - "Who follows me?" = scatter-gather (less common)       │
│                                                             │
│  USER DATA: Not sharded (fits on one machine)             │
│  - 1.5B users × 1KB = 1.5TB                              │
│  - Use read replicas for scale                            │
└─────────────────────────────────────────────────────────────┘
```

### Caching Strategy

```
┌─────────────────────────────────────────────────────────────┐
│              CACHING LAYERS                                 │
│                                                             │
│  LAYER 1: CDN                                               │
│  - Profile images                                          │
│  - Static assets                                           │
│                                                             │
│  LAYER 2: API Response Cache                               │
│  - User profiles (short TTL: 60s)                         │
│  - Popular tweets                                          │
│                                                             │
│  LAYER 3: Timeline Cache (Redis Cluster)                  │
│  - Pre-computed timelines                                  │
│  - Key: timeline:{user_id}                                │
│  - Value: Sorted set of tweet_ids                         │
│  - TTL: 24 hours                                           │
│  - Size: ~100GB for active users                          │
│                                                             │
│  LAYER 4: Tweet Cache                                       │
│  - Recently accessed tweets                               │
│  - Hydration cache                                         │
│                                                             │
│  CACHE HIT TARGETS:                                         │
│  - Timeline cache: 95%+ hit rate                          │
│  - Tweet cache: 80%+ hit rate                             │
│  - User cache: 90%+ hit rate                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 8: Handling Edge Cases

### New User / Cold Start

```
User signs up and follows 200 users
Their timeline cache is EMPTY

SOLUTION:
1. On first timeline request, detect empty cache
2. Fan-out on READ (one-time)
3. Fetch recent tweets from followed users
4. Populate cache
5. Future reads use cache
```

### User Unfollows Someone

```
User A unfollows User B
User B's tweets should not appear in A's timeline

SOLUTION:
1. Remove follow relationship from DB
2. Async: Remove B's tweets from A's timeline cache
3. Or: Let them expire naturally (lazy cleanup)
```

### Tweet Deletion

```
User deletes a tweet
Should disappear from all timelines

SOLUTION:
1. Mark tweet as deleted in DB
2. Async: Publish delete event to Kafka
3. Fan-out service removes from all caches
4. Eventual consistency: May take minutes
5. UI: Filter deleted tweets at render time
```

---

## Step 9: Final Architecture Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COMPLETE TWITTER ARCHITECTURE                             │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                              CLIENT LAYER                                ││
│  │   Mobile Apps    │    Web App    │    Third-party (API)                ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                     │                                        │
│  ┌─────────────────────────────────▼────────────────────────────────────┐  │
│  │                         EDGE LAYER                                     │  │
│  │  CDN (Images/Static)  │  Load Balancer  │  DDoS Protection          │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                     │                                        │
│  ┌─────────────────────────────────▼────────────────────────────────────┐  │
│  │                    GATEWAY LAYER                                       │  │
│  │  API Gateway: Auth │ Rate Limit │ Routing │ SSL                      │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                     │                                        │
│  ┌──────────────┬──────────────────┼───────────────────┬────────────────┐  │
│  │              │                  │                   │                │  │
│  ▼              ▼                  ▼                   ▼                │  │
│ ┌────────┐ ┌─────────┐       ┌──────────┐       ┌──────────┐          │  │
│ │ User   │ │ Tweet   │       │ Timeline │       │ Search   │          │  │
│ │Service │ │ Service │       │ Service  │       │ Service  │          │  │
│ └───┬────┘ └────┬────┘       └────┬─────┘       └────┬─────┘          │  │
│     │           │                 │                  │                 │  │
│  ┌──┴──────────┬┴─────────────────┴──────────────────┴───────────────┐ │  │
│  │             │                DATA LAYER                            │ │  │
│  │  ┌─────────┴────┐  ┌────────────┐  ┌─────────────┐  ┌──────────┐ │ │  │
│  │  │   User DB    │  │ Tweet DB   │  │  Follow DB  │  │  Search  │ │ │  │
│  │  │   (MySQL)    │  │ (Sharded)  │  │  (Sharded)  │  │ (Elastic)│ │ │  │
│  │  │  + Replicas  │  │  100 nodes │  │   50 nodes  │  │  Cluster │ │ │  │
│  │  └──────────────┘  └────────────┘  └─────────────┘  └──────────┘ │ │  │
│  │                                                                    │ │  │
│  │  ┌──────────────────────────────────────────────────────────────┐│ │  │
│  │  │               CACHE LAYER (Redis Cluster)                     ││ │  │
│  │  │  Timeline Cache │ Tweet Cache │ User Cache │ Session Cache   ││ │  │
│  │  └──────────────────────────────────────────────────────────────┘│ │  │
│  └───────────────────────────────────────────────────────────────────┘ │  │
│                                                                          │  │
│  ┌───────────────────────────────────────────────────────────────────┐  │  │
│  │                  ASYNC LAYER (Kafka)                               │  │  │
│  │   Fan-out Events │ Notification Events │ Analytics Events        │  │  │
│  └───────────────────────────────────────────────────────────────────┘  │  │
│                                                                          │  │
│  ┌───────────────────────────────────────────────────────────────────┐  │  │
│  │               STORAGE LAYER                                        │  │  │
│  │   S3 (Media) │ Glacier (Archive) │ Analytics DW                  │  │  │
│  └───────────────────────────────────────────────────────────────────┘  │  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Step 10: Tradeoffs & Discussion Points

```
┌─────────────────────────────────────────────────────────────┐
│              KEY TRADEOFFS                                  │
│                                                             │
│  1. FAN-OUT STRATEGY                                        │
│     Push (write) vs Pull (read) vs Hybrid                 │
│     We chose: Hybrid for scalability                       │
│                                                             │
│  2. CONSISTENCY MODEL                                       │
│     Strong vs Eventual                                      │
│     We chose: Eventual (few seconds delay OK)             │
│                                                             │
│  3. STORAGE                                                 │
│     SQL vs NoSQL                                           │
│     We chose: SQL for users, sharded for scale            │
│                                                             │
│  4. CACHE STRATEGY                                          │
│     What to cache, TTL policies                           │
│     We chose: Aggressive caching, 24h TTL                 │
│                                                             │
│  POTENTIAL FOLLOW-UP QUESTIONS:                            │
│  ─────────────────────────────────────────                 │
│  • "How would you add trending topics?"                   │
│  • "How would you handle spam detection?"                 │
│  • "How would you implement real-time notifications?"     │
│  • "How would you add direct messages?"                   │
│  • "How would you handle a celebrity with 100M followers?"│
└─────────────────────────────────────────────────────────────┘
```

---

## Interview Evaluation Criteria

```
┌─────────────────────────────────────────────────────────────┐
│         WHAT INTERVIEWERS LOOK FOR                          │
│                                                             │
│  ✓ REQUIREMENTS GATHERING                                   │
│    - Asked clarifying questions                            │
│    - Scoped appropriately                                  │
│                                                             │
│  ✓ CAPACITY ESTIMATION                                      │
│    - Reasonable numbers                                    │
│    - Identified read-heavy nature                          │
│                                                             │
│  ✓ HIGH-LEVEL DESIGN                                        │
│    - Covered all components                                │
│    - Logical data flow                                     │
│                                                             │
│  ✓ DEEP DIVE                                                │
│    - Timeline is the hard problem                          │
│    - Fan-out strategy explained                            │
│    - Celebrity problem addressed                           │
│                                                             │
│  ✓ SCALABILITY                                              │
│    - Sharding strategy                                     │
│    - Caching strategy                                      │
│                                                             │
│  ✓ TRADEOFFS                                                │
│    - Acknowledged limitations                              │
│    - Discussed alternatives                                │
└─────────────────────────────────────────────────────────────┘
```

---

**Next:** [13_HLD_Example_Uber.md](./13_HLD_Example_Uber.md) - Complete Uber system design
