# What is System Design?

## The Simple Explanation

Imagine you're building a house. You don't just start hammering nails randomly—you need a blueprint. You need to decide:
- How many rooms?
- Where do the pipes go?
- How will electricity flow?
- Can the foundation support a second floor later?

**System Design is the blueprint for software systems.**

When building software that serves millions of users, you need to answer questions like:
- Where will we store all this data?
- How do we make sure the system doesn't crash when too many users visit?
- What happens if one of our servers catches fire?
- How do we keep user data secure?

```
┌────────────────────────────────────────────────────────────┐
│                    SYSTEM DESIGN                           │
│                                                            │
│   The art of designing large-scale software systems        │
│   that are reliable, scalable, and maintainable.          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Why Does System Design Matter?

### Real-World Scenario

Imagine you built a simple website for your restaurant:
- Day 1: 10 customers visit → Works fine!
- Day 30: 100 customers visit → Still okay!
- Day 100: Your restaurant goes viral on TikTok
- Day 101: 1,000,000 people try to visit your site
- Result: 💥 CRASH

```
Traffic Growth Without System Design:

Users      │                               💥 CRASH!
           │                              /
1M         │                            /
           │                          /
100K       │                        /
           │                      /
10K        │                    /
           │                  /
1K         │       ________/
           │      /
100        │_____/
           │
           └──────────────────────────────────────► Time
               Day 1                     Day 100
```

**With proper system design, your system can handle this growth gracefully.**

---

## The Two Types of System Design

### High-Level Design (HLD)

Think of HLD as the city planning view—you see buildings, roads, and how they connect, but not the interior of each building.

**Questions HLD answers:**
- What are the main components?
- How do they communicate?
- Where is data stored?
- How does it scale?

**Example:** "Design Twitter"
```
┌─────────────────────────────────────────────────────────────────┐
│                    TWITTER HLD (Simplified)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────┐     ┌──────────────┐     ┌─────────────────┐    │
│   │  Users   │────►│ Load Balancer│────►│  Web Servers    │    │
│   └──────────┘     └──────────────┘     └────────┬────────┘    │
│                                                  │              │
│                    ┌─────────────────────────────┼─────────┐   │
│                    │                             │         │   │
│                    ▼                             ▼         ▼   │
│              ┌──────────┐              ┌──────────┐  ┌───────┐ │
│              │  Cache   │              │ Database │  │ Queue │ │
│              └──────────┘              └──────────┘  └───────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Low-Level Design (LLD)

Think of LLD as the architectural floor plan—you see every room, door, and electrical outlet.

**Questions LLD answers:**
- What classes do we need?
- What methods should each class have?
- How do objects interact?
- What design patterns apply?

**Example:** "Design a Parking Lot System"
```
┌─────────────────────────────────────────────────────────────────┐
│                    PARKING LOT LLD (Simplified)                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌────────────────┐                                            │
│   │  ParkingLot    │                                            │
│   ├────────────────┤       ┌────────────────┐                   │
│   │ - floors[]     │──────►│    Floor       │                   │
│   │ - capacity     │       ├────────────────┤                   │
│   ├────────────────┤       │ - spots[]      │                   │
│   │ + parkVehicle()│       │ - floorNumber  │                   │
│   │ + exitVehicle()│       └───────┬────────┘                   │
│   └────────────────┘               │                            │
│                                    ▼                            │
│                          ┌────────────────┐                     │
│                          │  ParkingSpot   │                     │
│                          ├────────────────┤                     │
│                          │ - spotId       │                     │
│                          │ - spotType     │                     │
│                          │ - isOccupied   │                     │
│                          └────────────────┘                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Concepts You'll Learn

### The Building Blocks

```
┌─────────────────────────────────────────────────────────────────┐
│                  SYSTEM DESIGN BUILDING BLOCKS                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STORAGE              COMPUTE              NETWORKING           │
│  ───────              ───────              ──────────           │
│  • Databases          • Servers            • Load Balancers     │
│  • Caches             • Containers         • CDNs               │
│  • File Systems       • Serverless         • DNS                │
│  • Object Storage                          • API Gateways       │
│                                                                 │
│  COMMUNICATION        RELIABILITY          SECURITY             │
│  ─────────────        ───────────          ────────             │
│  • REST APIs          • Replication        • Authentication     │
│  • Message Queues     • Redundancy         • Authorization      │
│  • WebSockets         • Failover           • Encryption         │
│  • gRPC               • Backups            • Rate Limiting      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### The "ilities" - What Makes a Good System

| Property | What It Means | Example |
|----------|---------------|---------|
| **Scalability** | Can handle more users/data | Netflix handles 200M users |
| **Reliability** | Works correctly, even when things fail | Banks don't lose your money |
| **Availability** | System is up and accessible | Google Search is always on |
| **Maintainability** | Easy to update and fix | Engineers can deploy daily |
| **Security** | Protected from attacks | Your password stays safe |

---

## System Design vs. Just Coding

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│   SMALL PROJECT (Just Coding)                                  │
│   ──────────────────────────                                   │
│                                                                │
│   You → Your Computer → Your Code → Works!                     │
│                                                                │
│   • One database                                               │
│   • One server                                                 │
│   • Simple logic                                               │
│   • You control everything                                     │
│                                                                │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   LARGE SYSTEM (System Design Required)                        │
│   ─────────────────────────────────────                        │
│                                                                │
│   Millions of Users → Your System → Complex Infrastructure     │
│                                                                │
│   • Multiple databases                                         │
│   • Hundreds of servers                                        │
│   • Complex distributed logic                                  │
│   • Multiple teams maintain different parts                    │
│   • Things WILL fail                                           │
│   • Must handle partial failures gracefully                    │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## The System Design Interview

### What Companies Want to Know

1. **Can you think at scale?**
   - Not "how to build for 100 users" but "how to build for 100 million users"

2. **Do you understand trade-offs?**
   - There's rarely a "perfect" solution
   - Every choice has pros and cons

3. **Can you communicate technical ideas?**
   - Can you explain your design clearly?
   - Can you respond to feedback?

4. **Do you have practical experience?**
   - Have you dealt with real-world constraints?
   - Do you know what can go wrong?

### A Typical Question

> "Design a URL shortener like bit.ly"

**What they're really asking:**
- How will you store billions of URLs?
- How will you generate unique short codes?
- How will you handle millions of requests per second?
- What happens if your database crashes?
- How will you ensure links work globally with low latency?

---

## Common Misconceptions

### Misconception 1: "I need to know everything"
**Truth:** You need to know core concepts deeply. Nobody knows everything.

### Misconception 2: "There's one right answer"
**Truth:** There are many valid designs. What matters is your reasoning.

### Misconception 3: "I should memorize designs"
**Truth:** Understanding beats memorization. Interviewers can tell the difference.

### Misconception 4: "System design is just for senior engineers"
**Truth:** Even junior engineers benefit from thinking about scale and reliability.

---

## Your First System Design: A Mental Model

Let's design a simple service: **A Todo List App**

### Step 1: Requirements
- Users can create, read, update, delete todos
- Each user has their own list
- Must work on mobile and web

### Step 2: Simple Design (v1)

```
┌─────────────────────────────────────────────────────────────────┐
│                     TODO APP v1                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    ┌────────┐         ┌────────────┐         ┌──────────┐      │
│    │ User   │ ──────► │   Server   │ ──────► │ Database │      │
│    │(Phone) │ ◄────── │  (1 box)   │ ◄────── │ (1 box)  │      │
│    └────────┘         └────────────┘         └──────────┘      │
│                                                                 │
│    This works for ~100 users!                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Step 3: What Breaks at Scale?

**1000 users:**
- Server gets slow → Add more servers
- Database becomes bottleneck → Add caching

**1,000,000 users:**
- Single database can't handle it → Shard the database
- Users in Europe have slow experience → Add multiple regions

### Step 4: Scaled Design (v2)

```
┌─────────────────────────────────────────────────────────────────┐
│                     TODO APP v2 (Scaled)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                      ┌──────────────┐                           │
│                      │     CDN      │                           │
│                      └──────┬───────┘                           │
│                             │                                   │
│                      ┌──────▼───────┐                           │
│    Users ──────────► │Load Balancer │                           │
│                      └──────┬───────┘                           │
│                             │                                   │
│              ┌──────────────┼──────────────┐                    │
│              │              │              │                    │
│        ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐             │
│        │  Server 1 │  │  Server 2 │  │  Server 3 │             │
│        └─────┬─────┘  └─────┬─────┘  └─────┬─────┘             │
│              │              │              │                    │
│              └──────────────┼──────────────┘                    │
│                             │                                   │
│                      ┌──────▼───────┐                           │
│                      │    Cache     │                           │
│                      │   (Redis)    │                           │
│                      └──────┬───────┘                           │
│                             │                                   │
│              ┌──────────────┼──────────────┐                    │
│              │              │              │                    │
│        ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐             │
│        │   DB 1    │  │   DB 2    │  │   DB 3    │             │
│        │(users A-I)│  │(users J-R)│  │(users S-Z)│             │
│        └───────────┘  └───────────┘  └───────────┘             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│                    REMEMBER THIS                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. System Design = Blueprint for large-scale software         │
│                                                                │
│  2. HLD = Big picture (components & connections)               │
│     LLD = Details (classes & code)                             │
│                                                                │
│  3. Good systems are: Scalable, Reliable, Available            │
│                                                                │
│  4. There's no perfect answer—trade-offs are everywhere        │
│                                                                │
│  5. Start simple, then scale                                   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

Now that you understand what system design is, let's learn how the internet works—the foundation everything else is built on.

**Next:** [How the Internet Works →](how_internet_works.md)
