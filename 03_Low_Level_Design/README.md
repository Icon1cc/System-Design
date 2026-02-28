# Low Level Design (LLD)

## What is Low Level Design?

Low Level Design (LLD) is the process of designing the **internal structure** of a system—the classes, interfaces, methods, and their interactions. While High Level Design answers "what components do we need?", LLD answers "how do we implement each component?"

```
┌─────────────────────────────────────────────────────────────────────┐
│                     HLD vs LLD Comparison                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   High Level Design (HLD)           Low Level Design (LLD)          │
│   ─────────────────────────         ────────────────────────        │
│                                                                     │
│   • System architecture             • Class diagrams                │
│   • Component interactions          • Method signatures             │
│   • Database choices                • Data structures               │
│   • API contracts                   • Algorithm selection           │
│   • Scaling strategy                • Design patterns               │
│   • Technology stack                • Code organization             │
│                                                                     │
│   "What boxes do we need?"          "What's inside each box?"       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Why LLD Matters in Interviews

Many companies (especially at senior levels) ask LLD questions to evaluate:

1. **Object-Oriented Design Skills** - Can you model real-world problems?
2. **Code Organization** - Do you write maintainable, extensible code?
3. **Design Pattern Knowledge** - Do you know proven solutions?
4. **API Design** - Can you create clean, intuitive interfaces?
5. **Trade-off Analysis** - Can you justify your design decisions?

## Common LLD Interview Questions

```
┌──────────────────────────────────────────────────────────────┐
│                 Popular LLD Questions                         │
├──────────────────────────────────────────────────────────────┤
│  • Design a Parking Lot System                               │
│  • Design an Elevator System                                 │
│  • Design a Chess Game                                       │
│  • Design a Library Management System                        │
│  • Design an ATM Machine                                     │
│  • Design a Vending Machine                                  │
│  • Design a Movie Ticket Booking System                      │
│  • Design a Hotel Booking System                             │
│  • Design a File System                                      │
│  • Design a Rate Limiter                                     │
│  • Design a Cache (LRU/LFU)                                  │
│  • Design a Logger                                           │
└──────────────────────────────────────────────────────────────┘
```

## LLD Interview Framework

Use this 5-step framework for any LLD question:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LLD Interview Framework                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Step 1: CLARIFY Requirements (2-3 min)                             │
│  ├── What are the core use cases?                                   │
│  ├── What entities are involved?                                    │
│  └── What are the constraints?                                      │
│                                                                     │
│  Step 2: IDENTIFY Core Objects (3-5 min)                            │
│  ├── What nouns appear in the requirements?                         │
│  ├── What are the relationships?                                    │
│  └── What behaviors do objects need?                                │
│                                                                     │
│  Step 3: DESIGN Class Hierarchy (10-15 min)                         │
│  ├── Define classes and interfaces                                  │
│  ├── Establish inheritance/composition                              │
│  └── Apply design patterns                                          │
│                                                                     │
│  Step 4: DEFINE APIs and Methods (10 min)                           │
│  ├── Public interfaces                                              │
│  ├── Method signatures                                              │
│  └── Return types and exceptions                                    │
│                                                                     │
│  Step 5: WALK Through Scenarios (5 min)                             │
│  ├── Trace main use cases                                           │
│  ├── Identify edge cases                                            │
│  └── Discuss extensibility                                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Study Order for This Section

1. **01_oop_refresher.md** - Object-Oriented Programming fundamentals
2. **02_solid_principles.md** - The 5 principles of good OO design
3. **03_design_patterns_creational.md** - Singleton, Factory, Builder
4. **04_design_patterns_structural.md** - Adapter, Decorator, Proxy
5. **05_design_patterns_behavioral.md** - Strategy, Observer, State
6. **06_uml_basics.md** - How to draw and read UML diagrams
7. **07_dependency_injection.md** - Loose coupling through DI
8. **08_thread_safety.md** - Writing concurrent-safe code
9. **09_lld_worked_examples.md** - Complete LLD interview solutions

## Key Principles to Remember

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LLD Design Principles                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. PROGRAM TO INTERFACES, NOT IMPLEMENTATIONS                      │
│     → Depend on abstractions for flexibility                        │
│                                                                     │
│  2. FAVOR COMPOSITION OVER INHERITANCE                              │
│     → "Has-a" is often better than "is-a"                           │
│                                                                     │
│  3. ENCAPSULATE WHAT VARIES                                         │
│     → Isolate changing parts from stable code                       │
│                                                                     │
│  4. STRIVE FOR LOOSELY COUPLED DESIGNS                              │
│     → Minimize dependencies between classes                         │
│                                                                     │
│  5. CLASSES SHOULD BE OPEN FOR EXTENSION, CLOSED FOR MODIFICATION   │
│     → Add features without changing existing code                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Common LLD Mistakes

| Mistake | Problem | Better Approach |
|---------|---------|-----------------|
| God class | One class does everything | Split responsibilities |
| Deep inheritance | Fragile hierarchy | Use composition |
| Hardcoded values | Inflexible code | Use configuration |
| No interfaces | Tight coupling | Abstract dependencies |
| Premature optimization | Over-engineering | Start simple, refactor |

---

**Next:** Start with [01_oop_refresher.md](./01_oop_refresher.md) to build your foundation.
