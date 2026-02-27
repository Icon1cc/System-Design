# High-Level Design (HLD) - Complete Guide

## What is High-Level Design?

High-Level Design is the **architectural blueprint** of a system. It answers the question: **"How do we build this system at scale?"**

Think of it like designing a city:
- You decide where to put roads, buildings, and utilities
- You don't worry about the color of individual houses yet
- You focus on how everything connects and flows

```
┌─────────────────────────────────────────────────────────────────┐
│                    HIGH-LEVEL DESIGN                            │
│                                                                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │
│  │ Client  │───▶│   API   │───▶│ Service │───▶│Database │     │
│  │  Apps   │    │ Gateway │    │  Layer  │    │  Layer  │     │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘     │
│                                                                 │
│  Focus: Components, Connections, Data Flow, Scale              │
└─────────────────────────────────────────────────────────────────┘
```

## HLD vs LLD: The Key Difference

| Aspect | High-Level Design | Low-Level Design |
|--------|-------------------|------------------|
| **Focus** | System architecture | Class/module design |
| **Question** | "What components do we need?" | "How do we code each component?" |
| **Output** | Architecture diagrams | Class diagrams, code structure |
| **Abstraction** | 30,000 feet view | Ground-level view |
| **Examples** | "Use Redis for caching" | "Implement LRU eviction in cache class" |

## Files in This Section

### Core Framework
1. **[01_Design_Framework.md](./01_Design_Framework.md)** - Step-by-step approach to any design
2. **[02_Requirements_Gathering.md](./02_Requirements_Gathering.md)** - How to extract requirements
3. **[03_Capacity_Estimation.md](./03_Capacity_Estimation.md)** - Back-of-envelope calculations
4. **[04_API_Design.md](./04_API_Design.md)** - Designing clean APIs

### Design Decisions
5. **[05_Data_Modeling.md](./05_Data_Modeling.md)** - Schema design strategies
6. **[06_Database_Selection.md](./06_Database_Selection.md)** - Choosing the right database
7. **[07_Designing_For_Scale.md](./07_Designing_For_Scale.md)** - Scaling strategies
8. **[08_High_Availability.md](./08_High_Availability.md)** - Building reliable systems

### Advanced Considerations
9. **[09_Multi_Region_Design.md](./09_Multi_Region_Design.md)** - Global system design
10. **[10_Read_Write_Patterns.md](./10_Read_Write_Patterns.md)** - Optimizing for workload types
11. **[11_Bottleneck_Detection.md](./11_Bottleneck_Detection.md)** - Finding and fixing bottlenecks

### Worked Examples
12. **[12_HLD_Example_Twitter.md](./12_HLD_Example_Twitter.md)** - Full Twitter design walkthrough
13. **[13_HLD_Example_Uber.md](./13_HLD_Example_Uber.md)** - Full Uber design walkthrough

## The Golden Framework (Memorize This!)

```
┌────────────────────────────────────────────────────────────────┐
│                  SYSTEM DESIGN FRAMEWORK                       │
│                                                                │
│  1. REQUIREMENTS (5 min)                                       │
│     ├── Functional: What should it do?                         │
│     └── Non-functional: How well should it do it?              │
│                                                                │
│  2. CAPACITY ESTIMATION (5 min)                                │
│     ├── Users, QPS, Storage                                    │
│     └── Bandwidth, Memory                                      │
│                                                                │
│  3. HIGH-LEVEL DESIGN (15 min)                                 │
│     ├── API Design                                             │
│     ├── Data Model                                             │
│     └── Architecture Diagram                                   │
│                                                                │
│  4. DEEP DIVE (15 min)                                         │
│     ├── Scale bottlenecks                                      │
│     ├── Database choices                                       │
│     └── Caching strategy                                       │
│                                                                │
│  5. WRAP UP (5 min)                                            │
│     ├── Tradeoffs discussed                                    │
│     └── Future improvements                                    │
└────────────────────────────────────────────────────────────────┘
```

## Quick Reference: Common Components

| Component | Purpose | When to Use |
|-----------|---------|-------------|
| Load Balancer | Distribute traffic | Multiple servers |
| API Gateway | Single entry point | Microservices |
| Cache (Redis) | Fast reads | Read-heavy, hot data |
| CDN | Static content | Global users, media |
| Message Queue | Async processing | Decoupling, spikes |
| Database (SQL) | Structured data | ACID needed |
| Database (NoSQL) | Flexible schema | Scale, unstructured |
| Search Engine | Full-text search | Search features |
| Object Storage | Files/blobs | Media, backups |

## How to Study This Section

```
Week 1: Master the Framework
├── Day 1-2: Design Framework + Requirements
├── Day 3-4: Capacity Estimation (practice math!)
└── Day 5-7: API Design + Data Modeling

Week 2: Design Decisions
├── Day 1-2: Database Selection
├── Day 3-4: Designing for Scale
└── Day 5-7: High Availability + Multi-Region

Week 3: Practice & Polish
├── Day 1-3: Study worked examples
├── Day 4-5: Practice bottleneck detection
└── Day 6-7: Mock designs with timer
```

## Interview Tips for HLD

1. **Always start with requirements** - Don't jump to solutions
2. **Think out loud** - Interviewers want to see your process
3. **Draw diagrams** - Visual communication is powerful
4. **Justify decisions** - "I chose X because Y"
5. **Acknowledge tradeoffs** - Nothing is perfect
6. **Stay high-level** - Don't get lost in implementation details

---

**Next Step:** Start with [01_Design_Framework.md](./01_Design_Framework.md)
