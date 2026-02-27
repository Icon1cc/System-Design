# System Design Mastery: From Zero to FAANG Interview Ready

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                    SYSTEM DESIGN INTERVIEW PREPARATION                        ║
║                     Complete Self-Study Repository                            ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

Welcome! This repository will take you from knowing nothing about system design to confidently tackling interviews at Google, Amazon, Meta, Netflix, and other top tech companies.

## What is System Design?

System design is the process of defining the architecture, components, and data flow of a system to satisfy specific requirements. In interviews, you're asked to design large-scale distributed systems like Twitter, Uber, or Netflix.

**Two Types of System Design:**

| High-Level Design (HLD) | Low-Level Design (LLD) |
|------------------------|------------------------|
| Big picture architecture | Code-level implementation |
| Components and their interactions | Classes, interfaces, methods |
| "How does Netflix stream videos?" | "Design a parking lot system in code" |
| Focuses on scalability, availability | Focuses on OOP, design patterns |
| Whiteboard diagrams | UML diagrams + code |

---

## Your Learning Roadmap

```
                              YOUR JOURNEY
                              ============

Week 1-2: FOUNDATIONS (Start Here!)
    │
    ├── What is System Design?
    ├── How the Internet Works
    ├── Latency, Throughput, Availability
    ├── CAP Theorem
    └── SQL vs NoSQL
            │
            ▼
Week 3-4: CORE BUILDING BLOCKS
    │
    ├── Load Balancers
    ├── Caching (Redis)
    ├── Message Queues (Kafka)
    ├── CDNs
    └── Databases Deep Dive
            │
            ▼
Week 5-6: SCALABILITY PATTERNS
    │
    ├── Sharding & Partitioning
    ├── Replication
    ├── Microservices
    ├── Event-Driven Architecture
    └── Consistency Patterns
            │
            ▼
Week 7-8: CASE STUDIES (The Real Practice)
    │
    ├── URL Shortener
    ├── Twitter Timeline
    ├── WhatsApp
    ├── Netflix
    ├── Uber
    └── More...
            │
            ▼
Week 9-10: INTERVIEW PREP & PRACTICE
    │
    ├── Mock Interviews
    ├── Communication Skills
    ├── Time Management
    └── Final Review
            │
            ▼
        ┌───────────┐
        │  READY!   │
        │  FOR YOUR │
        │ INTERVIEW │
        └───────────┘
```

---

## Repository Structure

```
System-Design/
│
├── 01_Foundations/           ← START HERE - Basic concepts
│   ├── what_is_system_design.md
│   ├── how_internet_works.md
│   ├── latency_throughput.md
│   └── ...more fundamentals
│
├── 02_High_Level_Design/     ← The interview framework
│   ├── hld_framework.md
│   ├── capacity_estimation.md
│   └── ...design strategies
│
├── 03_Low_Level_Design/      ← OOP & design patterns
│   ├── oop_refresher.md
│   ├── solid_principles.md
│   └── ...design patterns
│
├── 04_Core_Components/       ← Building blocks deep dive
│   ├── load_balancers.md
│   ├── caching_systems.md
│   └── ...infrastructure components
│
├── 05_Scalability_Patterns/  ← Advanced patterns
│   ├── sharding.md
│   ├── replication.md
│   └── ...scaling strategies
│
├── 06_Case_Studies/          ← Full interview examples
│   ├── url_shortener.md
│   ├── twitter_timeline.md
│   └── ...real system designs
│
├── 07_Interview_Prep/        ← Interview strategies
│   ├── interview_flow.md
│   ├── communication_tips.md
│   └── ...preparation guides
│
├── 08_Hands_On_Projects/     ← Code implementations
│   ├── lru_cache/
│   ├── rate_limiter/
│   └── ...mini projects
│
├── 09_Advanced_Topics/       ← Deep expertise
│   ├── distributed_transactions.md
│   ├── consensus_algorithms.md
│   └── ...advanced concepts
│
└── resources/                ← Curated learning materials
    ├── books.md
    ├── blogs.md
    └── ...external resources
```

---

## How System Design Interviews Work at Big Tech

### The Format

| Company | Duration | Focus |
|---------|----------|-------|
| Google | 45-60 min | Scalability, trade-offs |
| Amazon | 45 min | Leadership principles + design |
| Meta | 45 min | Scale (billions of users) |
| Netflix | 60 min | Streaming, availability |
| Microsoft | 45-60 min | Cloud architecture |

### What Interviewers Evaluate

```
┌─────────────────────────────────────────────────────────────────┐
│                    EVALUATION CRITERIA                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. REQUIREMENTS GATHERING (10%)                                │
│     └── Do you ask clarifying questions?                        │
│                                                                 │
│  2. HIGH-LEVEL DESIGN (30%)                                     │
│     └── Can you identify major components?                      │
│                                                                 │
│  3. DEEP DIVE (30%)                                             │
│     └── Can you explain component details?                      │
│                                                                 │
│  4. SCALABILITY (20%)                                           │
│     └── Can you handle millions of users?                       │
│                                                                 │
│  5. TRADE-OFFS (10%)                                            │
│     └── Do you understand pros/cons of decisions?               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### The Interview Flow

```
0:00 ──────► 0:05 ──────► 0:15 ──────► 0:35 ──────► 0:45
  │           │            │            │            │
  │           │            │            │            │
  ▼           ▼            ▼            ▼            ▼
Problem    Clarify     High-Level   Deep Dive    Wrap Up
Given      Requirements  Design      Components   Q&A
           & Scope      Diagram      & Scale
```

---

## Common Beginner Mistakes (Avoid These!)

### Mistake 1: Jumping into the solution immediately
**Fix:** Always spend 3-5 minutes gathering requirements first.

### Mistake 2: Over-engineering from the start
**Fix:** Start simple, then scale. Don't mention Kubernetes in minute one.

### Mistake 3: Not doing back-of-envelope calculations
**Fix:** Always estimate: users, requests/second, storage needs.

### Mistake 4: Ignoring trade-offs
**Fix:** Every decision has pros and cons. State them explicitly.

### Mistake 5: Designing in silence
**Fix:** Think out loud! The interviewer wants to see your thought process.

### Mistake 6: Memorizing solutions without understanding
**Fix:** Understand WHY each component exists, not just WHAT it does.

### Mistake 7: Forgetting about failure scenarios
**Fix:** Always ask: "What happens when X fails?"

---

## How to Use This Repository Effectively

### Study Strategy

```
┌────────────────────────────────────────────────────────────────┐
│                      DAILY STUDY PLAN                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   Morning (1 hour):                                            │
│   └── Read one concept deeply (Foundations/Components)         │
│                                                                │
│   Afternoon (1 hour):                                          │
│   └── Work through one case study                              │
│                                                                │
│   Evening (30 min):                                            │
│   └── Review and make notes / implement mini project           │
│                                                                │
│   Weekend:                                                     │
│   └── Mock interview practice (explain designs out loud)       │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Pro Tips

1. **Read actively**: Draw diagrams as you read
2. **Explain to someone**: Teaching is the best learning
3. **Time yourself**: Practice 45-minute design sessions
4. **Record yourself**: Listen to how you communicate
5. **Build projects**: The hands-on section makes concepts stick

---

## Quick Reference Card

### The Magic Formula for Any Design

```
┌─────────────────────────────────────────────────────────┐
│              SYSTEM DESIGN FORMULA                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. REQUIREMENTS → What are we building?                │
│                                                         │
│  2. ESTIMATION  → How big is it?                        │
│                                                         │
│  3. INTERFACE   → What APIs do we need?                 │
│                                                         │
│  4. DATA MODEL  → What data do we store?                │
│                                                         │
│  5. HIGH-LEVEL  → What are the main components?         │
│                                                         │
│  6. DEEP DIVE   → How does each component work?         │
│                                                         │
│  7. SCALE       → How do we handle growth?              │
│                                                         │
│  8. TRADE-OFFS  → What are the pros/cons?               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Essential Numbers to Memorize

| Metric | Value | Notes |
|--------|-------|-------|
| L1 cache | 0.5 ns | Fastest |
| L2 cache | 7 ns | |
| RAM | 100 ns | |
| SSD | 150 μs | 1000x slower than RAM |
| HDD | 10 ms | Avoid for hot data |
| Network (same DC) | 0.5 ms | |
| Network (cross-continent) | 150 ms | |

### Data Size Quick Math

```
1 KB  = 1,000 bytes       (a short email)
1 MB  = 1,000 KB          (a photo)
1 GB  = 1,000 MB          (a movie)
1 TB  = 1,000 GB          (small database)
1 PB  = 1,000 TB          (large company's data)

Characters: 1 byte (ASCII), 2-4 bytes (Unicode)
Integer: 4-8 bytes
Timestamp: 8 bytes
UUID: 16 bytes
```

---

## Let's Begin!

Ready to start your journey? Head to **[01_Foundations/what_is_system_design.md](01_Foundations/what_is_system_design.md)** and begin!

```
     ╔══════════════════════════════════════════════════════╗
     ║                                                      ║
     ║   "The best time to start was yesterday.            ║
     ║    The next best time is now."                      ║
     ║                                                      ║
     ║                        Good luck on your journey!    ║
     ║                                                      ║
     ╚══════════════════════════════════════════════════════╝
```

---

## Contributing

Found an error? Have a suggestion? Feel free to open an issue or submit a pull request.

## License

This repository is for educational purposes. Share knowledge freely!
