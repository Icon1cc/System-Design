# Requirements Gathering - The Foundation of Good Design

## Why Requirements Matter

**80% of system design failures can be traced back to unclear requirements.**

The first 5 minutes of your interview are the most important. Get requirements wrong, and you'll design the wrong system entirely.

```
┌─────────────────────────────────────────────────────────────┐
│                    THE ICEBERG PROBLEM                      │
│                                                             │
│                        "Design Twitter"                     │
│                             │                               │
│                             ▼                               │
│                    ┌─────────────────┐                     │
│    What they say:  │   Tweet, Like   │   ← 10% visible    │
│                    └────────┬────────┘                     │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │
│                    ┌────────┴────────┐                     │
│                    │  Scale: 500M    │                     │
│                    │  Latency: <200ms│                     │
│    What matters:   │  Celebrity prob │   ← 90% hidden     │
│                    │  Read:Write 100:1│                    │
│                    │  Global users   │                     │
│                    │  Media storage  │                     │
│                    └─────────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

## Functional vs Non-Functional Requirements

### Functional Requirements (FR)
**What the system should DO**

These describe the features and behaviors:
- User can post a tweet
- User can follow other users
- User can see a timeline
- User can search for tweets

### Non-Functional Requirements (NFR)
**How WELL the system should perform**

These describe quality attributes:
- Handle 100,000 requests per second
- 99.99% availability
- Response time under 200ms
- Data retained for 10 years

```
┌─────────────────────────────────────────────────────────────┐
│           FUNCTIONAL vs NON-FUNCTIONAL                      │
│                                                             │
│  FUNCTIONAL (Features)          NON-FUNCTIONAL (Quality)    │
│  ├── Create account             ├── Performance             │
│  ├── Post content               │   ├── Latency            │
│  ├── Search                     │   ├── Throughput         │
│  ├── Notifications              │   └── Response time      │
│  ├── Follow users               ├── Scalability            │
│  └── View feed                  │   ├── User growth        │
│                                 │   └── Data growth        │
│                                 ├── Reliability            │
│                                 │   ├── Availability       │
│                                 │   └── Durability         │
│                                 ├── Security               │
│                                 └── Cost                   │
└─────────────────────────────────────────────────────────────┘
```

## The Requirements Questioning Framework

### Category 1: Users & Scale

**Must-ask questions:**

```
1. "Who are the users?"
   - Consumers? Businesses? Internal?
   - Global or regional?

2. "How many users?"
   - Daily Active Users (DAU)
   - Monthly Active Users (MAU)
   - Growth expectations

3. "How do they use it?"
   - Peak hours
   - Usage patterns
   - Mobile vs web
```

**Example Dialog:**
```
You: "How many daily active users should we design for?"
Interviewer: "Let's say 100 million DAU."
You: "And what's the expected growth? Should I design for 10x?"
Interviewer: "Yes, design for future scale."
You: "Got it - 100M DAU now, planning for 1B."
```

### Category 2: Core Features

**Prioritization is key:**

```
You: "What are the MUST-HAVE features for MVP?"
You: "What's OUT of scope for this discussion?"
You: "What's the single most important user flow?"
```

**Use the MoSCoW Method:**
```
┌─────────────────────────────────────────────────────────────┐
│                    MoSCoW PRIORITIZATION                    │
│                                                             │
│  MUST HAVE (Core)           SHOULD HAVE (Important)         │
│  ├── Post tweets            ├── Edit tweets                 │
│  ├── Follow users           ├── Bookmarks                   │
│  └── View timeline          └── Lists                       │
│                                                             │
│  COULD HAVE (Nice)          WON'T HAVE (Out of scope)       │
│  ├── Polls                  ├── Spaces (audio)              │
│  └── Thread analytics       └── Monetization                │
└─────────────────────────────────────────────────────────────┘
```

### Category 3: Performance Requirements

**Key metrics to clarify:**

```
LATENCY
├── "What's acceptable response time?"
├── "P50 vs P99 requirements?"
└── "Real-time or near-real-time?"

AVAILABILITY
├── "What's the availability target?"
├── "99.9%? 99.99%? 99.999%?"
└── "Can we have scheduled maintenance?"

CONSISTENCY
├── "Is eventual consistency acceptable?"
├── "What requires strong consistency?"
└── "How stale can data be?"
```

**Availability Numbers to Remember:**
```
┌────────────────────────────────────────────────────────────┐
│                 AVAILABILITY TARGETS                        │
│                                                             │
│  Availability    Downtime/Year    Downtime/Month           │
│  ─────────────────────────────────────────────             │
│  99%             3.65 days        7.3 hours                │
│  99.9%           8.76 hours       43.8 minutes             │
│  99.99%          52.6 minutes     4.4 minutes              │
│  99.999%         5.26 minutes     26 seconds               │
│                                                             │
│  Most systems: 99.9% - 99.99%                              │
│  Payment systems: 99.99%+                                   │
│  Internal tools: 99% often acceptable                      │
└────────────────────────────────────────────────────────────┘
```

### Category 4: Data Requirements

```
STORAGE
├── "How long do we retain data?"
├── "What's the data format?"
└── "Do we need to support media?"

ACCESS PATTERNS
├── "Read-heavy or write-heavy?"
├── "What's the read:write ratio?"
└── "What queries are most common?"

CONSISTENCY
├── "Can users see stale data?"
├── "What operations need ACID?"
└── "Global ordering required?"
```

### Category 5: Constraints & Special Cases

```
TECHNICAL CONSTRAINTS
├── "Any existing systems to integrate?"
├── "Technology preferences/restrictions?"
└── "Budget or resource constraints?"

SPECIAL SCENARIOS
├── "Celebrity/viral content handling?"
├── "Seasonal traffic spikes?"
└── "Disaster recovery requirements?"

COMPLIANCE
├── "Data privacy requirements?"
├── "Geographic restrictions?"
└── "Audit requirements?"
```

## The Complete Requirements Template

Use this checklist in every interview:

```
┌─────────────────────────────────────────────────────────────┐
│              REQUIREMENTS CHECKLIST                         │
│                                                             │
│  USERS & SCALE                                              │
│  □ Who are the users?                                       │
│  □ DAU/MAU numbers                                          │
│  □ Geographic distribution                                  │
│  □ Growth expectations                                      │
│                                                             │
│  FUNCTIONAL REQUIREMENTS                                    │
│  □ Core features (3-5)                                      │
│  □ Out of scope features                                    │
│  □ User journeys                                            │
│                                                             │
│  NON-FUNCTIONAL REQUIREMENTS                                │
│  □ Latency targets                                          │
│  □ Availability target                                      │
│  □ Consistency requirements                                 │
│  □ Data retention period                                    │
│                                                             │
│  ACCESS PATTERNS                                            │
│  □ Read:write ratio                                         │
│  □ Peak vs average traffic                                  │
│  □ Hot spots expected?                                      │
│                                                             │
│  CONSTRAINTS                                                │
│  □ Budget/cost constraints                                  │
│  □ Technology constraints                                   │
│  □ Compliance requirements                                  │
└─────────────────────────────────────────────────────────────┘
```

## Example: Complete Requirements for Twitter

Let's see how a strong candidate gathers requirements:

```
DIALOG:

Candidate: "Before I start designing, I'd like to understand the
           requirements better. First, who are our users?"

Interviewer: "General consumers, global audience."

Candidate: "How many daily active users?"

Interviewer: "Let's say 500 million DAU."

Candidate: "Got it. And what are the core features I should focus on?"

Interviewer: "Tweets, following users, and the home timeline."

Candidate: "Should I include direct messages, search, or notifications?"

Interviewer: "Let's keep DMs out of scope. Include basic search
             and notifications."

Candidate: "What's our latency target for timeline loading?"

Interviewer: "Under 200ms for P99."

Candidate: "For availability, should I target 99.99%?"

Interviewer: "Yes, that's our target."

Candidate: "Is eventual consistency acceptable for the timeline?"

Interviewer: "Yes, users don't need to see tweets instantly.
             A few seconds delay is fine."

Candidate: "Understood. What about celebrity users with millions
           of followers? Should I optimize for that?"

Interviewer: "Yes, that's an important case to handle."

Candidate: "Let me summarize..."
```

**Resulting Requirements Document:**

```
┌─────────────────────────────────────────────────────────────┐
│           TWITTER DESIGN - REQUIREMENTS                     │
│                                                             │
│  FUNCTIONAL REQUIREMENTS                                    │
│  ├── Post tweets (text + media up to 4 images)             │
│  ├── Follow/unfollow users                                  │
│  ├── View home timeline (posts from followed users)        │
│  ├── Like and retweet                                       │
│  ├── Basic search for tweets and users                     │
│  └── Push notifications for mentions and likes             │
│                                                             │
│  OUT OF SCOPE                                               │
│  ├── Direct messages                                        │
│  ├── Spaces (audio rooms)                                   │
│  ├── Analytics                                              │
│  └── Ads platform                                           │
│                                                             │
│  NON-FUNCTIONAL REQUIREMENTS                                │
│  ├── Scale: 500M DAU, design for 1B                        │
│  ├── Latency: Timeline < 200ms P99                         │
│  ├── Availability: 99.99% uptime                           │
│  ├── Consistency: Eventual (few seconds acceptable)        │
│  └── Data retention: 10 years                              │
│                                                             │
│  SPECIAL CONSIDERATIONS                                     │
│  ├── Celebrity users (1M+ followers)                       │
│  ├── Viral tweets                                           │
│  ├── Read-heavy: 100:1 read to write ratio                │
│  └── Global user base                                       │
└─────────────────────────────────────────────────────────────┘
```

## Smart Assumptions

Sometimes interviewers won't give you all the details. Make **reasonable assumptions** and state them clearly:

```
✅ GOOD ASSUMPTIONS:
"I'll assume a read-heavy workload with 100:1 ratio, which is
typical for social media. Let me know if that's different."

"I'll assume we're optimizing for availability over consistency,
since users expect the timeline to always load."

"I'll design for 5 years of data retention unless you'd like
something different."

❌ BAD ASSUMPTIONS:
Not stating assumptions at all
Assuming critical requirements without asking
Making assumptions that dramatically change the problem
```

## Requirements Anti-Patterns

### Anti-Pattern 1: The Feature Creep
```
❌ Listing 15 features
✅ Focus on 3-5 core features

Say: "I'll focus on the core features. We can discuss
     extensions at the end if time permits."
```

### Anti-Pattern 2: The Vague Requirements
```
❌ "It should be fast"
✅ "Timeline loads in under 200ms P99"

❌ "Lots of users"
✅ "500M DAU with 2x peak during events"
```

### Anti-Pattern 3: The Non-Questioner
```
❌ Making all assumptions silently
✅ "Can I clarify - is search a core requirement?"
```

### Anti-Pattern 4: The Over-Clarifier
```
❌ Spending 10 minutes on requirements
✅ 3-5 minutes, then move on

Say: "I think I have enough to start. I'll make some
     assumptions and call them out as I go."
```

## Interview Phrases for Requirements

**Opening:**
> "Before I dive into the design, I'd like to spend a few minutes understanding the requirements."

**Clarifying:**
> "Just to make sure I understand correctly, are we optimizing for..."

**Scoping:**
> "For the scope of this discussion, I'll focus on X, Y, and Z. We can discuss others if time permits."

**Assumptions:**
> "I'll assume [X]. Please correct me if that's not right."

**Transitioning:**
> "I feel I have a good understanding of the requirements. Let me summarize before moving to the design..."

## Practice Exercise

**Prompt:** "Design a food delivery app like DoorDash"

**Write down your questions in these categories:**

```
USERS & SCALE:
1. _________________________________
2. _________________________________
3. _________________________________

FUNCTIONAL:
1. _________________________________
2. _________________________________
3. _________________________________

NON-FUNCTIONAL:
1. _________________________________
2. _________________________________
3. _________________________________

SPECIAL CASES:
1. _________________________________
2. _________________________________
```

**Sample Answers:**

```
USERS & SCALE:
1. How many DAU (customers + restaurants + drivers)?
2. Which cities/regions are we targeting?
3. What's the peak order volume (meal times)?

FUNCTIONAL:
1. Core flow: browse → order → track → deliver?
2. Should I include restaurant management features?
3. Is real-time driver tracking required?

NON-FUNCTIONAL:
1. Order placement latency target?
2. What if the system goes down during an order?
3. How accurate should delivery ETAs be?

SPECIAL CASES:
1. How do we handle peak hours (dinner rush)?
2. What about popular restaurants with high order volume?
3. Driver unavailability in certain areas?
```

---

**Next:** [03_Capacity_Estimation.md](./03_Capacity_Estimation.md) - Turn requirements into numbers
