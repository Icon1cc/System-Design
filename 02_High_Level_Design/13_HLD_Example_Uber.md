# HLD Example: Uber System Design

## The Interview Scenario

**Interviewer:** "Design a ride-sharing service like Uber."

This is a complex system design question that tests your ability to handle real-time location tracking, matching algorithms, and high-throughput geospatial queries.

---

## Step 1: Requirements Gathering (5 minutes)

### Clarifying Questions

```
YOU: "Let me clarify the requirements. Should I focus on the
     core ride-hailing experience or include features like
     Uber Eats or scheduled rides?"

INTERVIEWER: "Focus on core ride-hailing: request ride, match
             driver, track ride, payment."

YOU: "What scale should I design for?"

INTERVIEWER: "Design for a city like New York with 500,000
             active drivers and 10 million riders."

YOU: "What latency is acceptable for matching?"

INTERVIEWER: "Riders should see nearby drivers within 1 second,
             and be matched within 5 seconds."

YOU: "Should I handle surge pricing?"

INTERVIEWER: "Yes, include basic surge pricing."
```

### Requirements Summary

```
┌─────────────────────────────────────────────────────────────┐
│                UBER REQUIREMENTS                            │
│                                                             │
│  FUNCTIONAL REQUIREMENTS:                                   │
│  ✓ Rider requests a ride (pickup, destination)            │
│  ✓ See nearby available drivers                           │
│  ✓ Match rider with nearest driver                        │
│  ✓ Real-time location tracking (driver position)         │
│  ✓ ETA calculation                                         │
│  ✓ Trip completion and payment                            │
│  ✓ Rating system (driver and rider)                       │
│  ✓ Basic surge pricing                                     │
│  ✗ Uber Eats (out of scope)                               │
│  ✗ Scheduled rides (out of scope)                         │
│  ✗ Carpooling/UberPool (out of scope)                     │
│                                                             │
│  NON-FUNCTIONAL REQUIREMENTS:                               │
│  - 500K active drivers                                     │
│  - 10M registered riders                                   │
│  - 1M rides per day (peak city)                           │
│  - Match latency < 5 seconds                              │
│  - Location update every 4 seconds                        │
│  - 99.99% availability                                     │
│  - Global service (multiple cities)                       │
│                                                             │
│  KEY CHARACTERISTICS:                                       │
│  - Write-heavy (location updates)                         │
│  - Real-time requirements                                  │
│  - Geospatial queries critical                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 2: Capacity Estimation (5 minutes)

### Traffic Estimates

```
┌─────────────────────────────────────────────────────────────┐
│              TRAFFIC ESTIMATION                             │
│                                                             │
│  LOCATION UPDATES (Write-heavy!):                          │
│  ─────────────────────────────────────────                 │
│  - Active drivers: 500,000                                 │
│  - Update frequency: Every 4 seconds                      │
│  - Updates/second: 500K / 4 = 125,000 updates/sec        │
│  - Peak (1.5x): ~190,000 updates/sec                      │
│                                                             │
│  THIS IS MASSIVE WRITE LOAD!                               │
│                                                             │
│  RIDE REQUESTS:                                             │
│  ─────────────────────────────────────────                 │
│  - 1M rides/day                                            │
│  - Peak hour: 10% of daily = 100K rides                   │
│  - Ride requests/sec: 100K / 3600 = ~28 requests/sec     │
│  - Peak: ~50 requests/sec                                 │
│                                                             │
│  GEOSPATIAL QUERIES:                                        │
│  ─────────────────────────────────────────                 │
│  - Each ride request: ~5-10 nearby driver queries        │
│  - Query rate: 50 × 10 = 500 queries/sec                 │
│  - Plus rider app showing nearby cars: ~10K queries/sec  │
│                                                             │
│  TOTAL QPS BREAKDOWN:                                       │
│  - Location writes: 190K/sec (DOMINANT)                   │
│  - Geospatial reads: 10K/sec                              │
│  - Ride operations: 50/sec                                │
└─────────────────────────────────────────────────────────────┘
```

### Storage Estimates

```
┌─────────────────────────────────────────────────────────────┐
│              STORAGE ESTIMATION                             │
│                                                             │
│  DRIVER LOCATIONS (Hot data):                              │
│  ─────────────────────────────────────────                 │
│  - 500K drivers                                            │
│  - Per driver: driver_id (8B) + lat (8B) + lng (8B)      │
│                + timestamp (8B) + status (1B) = ~40 bytes │
│  - Total: 500K × 40B = 20 MB                              │
│  - Easily fits in memory!                                  │
│                                                             │
│  TRIP HISTORY (Growing data):                              │
│  ─────────────────────────────────────────                 │
│  - 1M trips/day                                            │
│  - Per trip: ~1 KB (details, route, payment)             │
│  - Daily: 1 GB                                             │
│  - Yearly: 365 GB                                          │
│  - 5 years: ~2 TB                                          │
│                                                             │
│  LOCATION HISTORY (Time-series):                           │
│  ─────────────────────────────────────────                 │
│  - 190K updates/sec × 40 bytes = 7.6 MB/sec              │
│  - Daily: 650 GB                                           │
│  - Keep 7 days: 4.5 TB                                     │
│  - Then archive/aggregate                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 3: High-Level Design (15 minutes)

### Architecture Overview

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                      UBER HIGH-LEVEL ARCHITECTURE                              │
│                                                                                │
│   ┌──────────────────────────────────────────────────────────────────────┐   │
│   │                        CLIENT APPS                                     │   │
│   │          Rider App                      Driver App                    │   │
│   │    (Request rides, track)         (Accept rides, navigate)           │   │
│   └────────────────┬─────────────────────────────┬────────────────────────┘   │
│                    │                             │                             │
│                    │ HTTPS/WebSocket            │ HTTPS/WebSocket             │
│                    │                             │                             │
│   ┌────────────────▼─────────────────────────────▼────────────────────────┐   │
│   │                        LOAD BALANCER                                   │   │
│   │                  (Geographic + WebSocket aware)                       │   │
│   └────────────────────────────────┬──────────────────────────────────────┘   │
│                                    │                                           │
│   ┌────────────────────────────────▼──────────────────────────────────────┐   │
│   │                         API GATEWAY                                    │   │
│   │              Auth │ Rate Limit │ Request Routing                      │   │
│   └────────┬────────────────┬────────────────┬────────────────┬───────────┘   │
│            │                │                │                │               │
│   ┌────────▼──────┐ ┌───────▼───────┐ ┌─────▼─────┐ ┌────────▼────────┐     │
│   │    User       │ │   Location    │ │   Ride    │ │    Matching     │     │
│   │   Service     │ │   Service     │ │  Service  │ │    Service      │     │
│   │               │ │               │ │           │ │                 │     │
│   │ - Profiles    │ │ - Track       │ │ - Request │ │ - Find drivers  │     │
│   │ - Auth        │ │ - Store locs  │ │ - Trip    │ │ - Assign driver │     │
│   │ - Ratings     │ │ - Geospatial  │ │ - Status  │ │ - ETA calc      │     │
│   └───────┬───────┘ └───────┬───────┘ └─────┬─────┘ └────────┬────────┘     │
│           │                 │                │                │               │
│   ┌───────┴─────────────────┴────────────────┴────────────────┴───────────┐   │
│   │                           DATA LAYER                                   │   │
│   │                                                                        │   │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│   │  │  User DB    │  │ Location    │  │  Trip DB    │  │   Pricing   │  │   │
│   │  │  (MySQL)    │  │  Store      │  │  (MySQL)    │  │   (Redis)   │  │   │
│   │  │             │  │(Redis+Geo)  │  │  Sharded    │  │             │  │   │
│   │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│   │                                                                        │   │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                   │   │
│   │  │  Location   │  │   Maps/     │  │  Payment    │                   │   │
│   │  │  History    │  │   Routing   │  │  Service    │                   │   │
│   │  │ (Cassandra) │  │  (External) │  │  (Stripe)   │                   │   │
│   │  └─────────────┘  └─────────────┘  └─────────────┘                   │   │
│   └────────────────────────────────────────────────────────────────────────┘   │
│                                                                                │
│   ┌────────────────────────────────────────────────────────────────────────┐   │
│   │                    MESSAGE QUEUE (Kafka)                                │   │
│   │         Location Events │ Trip Events │ Notification Events           │   │
│   └────────────────────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────────────────┘
```

### Core Components

```
┌─────────────────────────────────────────────────────────────┐
│                 COMPONENT BREAKDOWN                          │
│                                                             │
│  1. LOCATION SERVICE (Most critical!)                       │
│     - Receives driver location updates                     │
│     - Stores current location for all drivers             │
│     - Handles geospatial queries                          │
│     - Must handle 190K writes/sec                         │
│                                                             │
│  2. MATCHING SERVICE                                         │
│     - Finds available drivers near pickup point           │
│     - Selects best driver (distance, rating, ETA)        │
│     - Sends ride requests to drivers                      │
│     - Handles acceptance/rejection                        │
│                                                             │
│  3. RIDE SERVICE                                             │
│     - Manages ride lifecycle                               │
│     - Tracks ride status (requested → completed)          │
│     - Stores trip details                                  │
│     - Calculates fare                                      │
│                                                             │
│  4. USER SERVICE                                             │
│     - Driver and rider profiles                           │
│     - Authentication                                       │
│     - Ratings management                                   │
│                                                             │
│  5. PRICING SERVICE                                          │
│     - Base fare calculation                                │
│     - Surge pricing (supply/demand)                       │
│     - Promotions and discounts                            │
│                                                             │
│  6. NOTIFICATION SERVICE                                     │
│     - Push notifications                                   │
│     - SMS fallback                                         │
│     - Real-time updates via WebSocket                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 4: API Design

```
┌─────────────────────────────────────────────────────────────┐
│                    UBER API                                 │
│                                                             │
│  DRIVER LOCATION (High frequency):                         │
│  ────────────────────────────────────────────              │
│  POST /api/v1/drivers/location                            │
│  Body: {                                                    │
│    "driver_id": "abc123",                                  │
│    "latitude": 40.7128,                                    │
│    "longitude": -74.0060,                                  │
│    "timestamp": 1642524800,                                │
│    "heading": 180,                                         │
│    "speed": 25                                             │
│  }                                                          │
│  Response: 200 OK                                          │
│                                                             │
│  GET /api/v1/drivers/nearby                               │
│  Params: ?lat=40.71&lng=-74.00&radius=5km                │
│  Response: {                                               │
│    "drivers": [                                            │
│      {"driver_id": "x", "lat": 40.72, "lng": -74.01,     │
│       "eta_minutes": 3}                                    │
│    ]                                                        │
│  }                                                          │
│                                                             │
│  RIDE OPERATIONS:                                           │
│  ────────────────────────────────────────────              │
│  POST /api/v1/rides                                        │
│  Body: {                                                    │
│    "rider_id": "user123",                                  │
│    "pickup": {"lat": 40.71, "lng": -74.00},              │
│    "destination": {"lat": 40.75, "lng": -73.98},         │
│    "ride_type": "UberX"                                    │
│  }                                                          │
│  Response: {                                               │
│    "ride_id": "ride456",                                   │
│    "status": "matching",                                   │
│    "estimated_fare": {"min": 15, "max": 20, "surge": 1.2}│
│  }                                                          │
│                                                             │
│  GET /api/v1/rides/{ride_id}                              │
│  Response: Full ride details + driver location            │
│                                                             │
│  POST /api/v1/rides/{ride_id}/accept (Driver)             │
│  POST /api/v1/rides/{ride_id}/cancel                      │
│  POST /api/v1/rides/{ride_id}/start                       │
│  POST /api/v1/rides/{ride_id}/complete                    │
│                                                             │
│  WEBSOCKET (Real-time):                                     │
│  ────────────────────────────────────────────              │
│  ws://api.uber.com/v1/rides/{ride_id}/track              │
│  - Rider: Receive driver location updates                 │
│  - Driver: Receive navigation updates                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 5: Deep Dive - Location Service

### The Core Challenge

```
┌─────────────────────────────────────────────────────────────┐
│              THE LOCATION CHALLENGE                         │
│                                                             │
│  REQUIREMENTS:                                              │
│  1. Store 500K driver locations                           │
│  2. Update 190K times per second                          │
│  3. Query "drivers within 5km" in < 100ms                │
│                                                             │
│  NAIVE APPROACH:                                            │
│  SELECT * FROM drivers                                     │
│  WHERE distance(lat, lng, 40.71, -74.00) < 5              │
│                                                             │
│  PROBLEM:                                                   │
│  - Full table scan of 500K rows                           │
│  - Distance calculation for each                          │
│  - Way too slow (seconds)                                 │
│                                                             │
│  WE NEED EFFICIENT GEOSPATIAL INDEXING                    │
└─────────────────────────────────────────────────────────────┘
```

### Solution: Geospatial Indexing with Geohash

```
┌─────────────────────────────────────────────────────────────┐
│                    GEOHASH CONCEPT                          │
│                                                             │
│  Geohash converts lat/lng into a string that groups       │
│  nearby locations together                                  │
│                                                             │
│  EXAMPLE:                                                   │
│  Location (40.7128, -74.0060) → "dr5ru7"                  │
│                                                             │
│  PRECISION:                                                 │
│  Length    Cell Size         Use Case                      │
│  ──────────────────────────────────────                   │
│  4         ~20 km           Country-level                 │
│  5         ~5 km            City area                     │
│  6         ~1.2 km          Neighborhood                  │
│  7         ~150 m           Block level                   │
│  8         ~40 m            Building level                │
│                                                             │
│  HOW IT HELPS:                                              │
│  - Nearby points share prefix                             │
│  - Query by prefix is O(1) lookup                        │
│  - Convert radius search to prefix search                │
│                                                             │
│  ┌─────────────────────────────────────┐                  │
│  │  dr5ru │ dr5rv │ dr5rw │           │                  │
│  ├────────┼───────┼───────┼───────────│                  │
│  │ dr5rg  │dr5rh★ │ dr5rj │           │   ★ = query     │
│  ├────────┼───────┼───────┼───────────│   point         │
│  │ dr5rd  │ dr5re │ dr5rf │           │                  │
│  └─────────────────────────────────────┘                  │
│                                                             │
│  To find drivers within 1km of ★:                         │
│  Search: dr5rh + 8 neighboring cells                     │
└─────────────────────────────────────────────────────────────┘
```

### Location Storage Architecture

```
┌─────────────────────────────────────────────────────────────┐
│           LOCATION SERVICE ARCHITECTURE                     │
│                                                             │
│              Driver App                                     │
│                  │                                          │
│                  │ POST /location                          │
│                  │ (every 4 seconds)                       │
│                  ▼                                          │
│         ┌───────────────┐                                  │
│         │   Location    │                                  │
│         │   Service     │                                  │
│         └───────┬───────┘                                  │
│                 │                                           │
│     ┌───────────┼───────────┐                              │
│     │           │           │                              │
│     ▼           ▼           ▼                              │
│ ┌───────┐  ┌───────┐  ┌───────┐                          │
│ │Redis 1│  │Redis 2│  │Redis 3│  (Cluster by geohash)   │
│ │ NYC   │  │  LA   │  │Chicago│                          │
│ └───────┘  └───────┘  └───────┘                          │
│                                                             │
│ REDIS DATA STRUCTURE:                                       │
│ ┌─────────────────────────────────────────────────────┐   │
│ │ Key: driver_locations:{geohash_prefix}             │   │
│ │ Type: Sorted Set (ZSET) or GEO                     │   │
│ │                                                     │   │
│ │ Using Redis GEO commands:                          │   │
│ │ GEOADD drivers -74.0060 40.7128 driver_123        │   │
│ │ GEORADIUS drivers -74.00 40.71 5 km               │   │
│ │                                                     │   │
│ │ Returns: driver_ids within radius                  │   │
│ └─────────────────────────────────────────────────────┘   │
│                                                             │
│ WHY REDIS:                                                  │
│ - In-memory: Sub-millisecond latency                      │
│ - GEO commands: Native geospatial support                │
│ - 190K writes/sec: Easily handled                        │
│ - Data is ephemeral anyway (only current location)       │
└─────────────────────────────────────────────────────────────┘
```

### Handling Location Updates

```
┌─────────────────────────────────────────────────────────────┐
│           LOCATION UPDATE FLOW                              │
│                                                             │
│  Driver App             Location Service           Redis    │
│      │                        │                      │      │
│      │ Location update        │                      │      │
│      │───────────────────────▶│                      │      │
│      │ {driver_id, lat, lng}  │                      │      │
│      │                        │                      │      │
│      │                        │ Calculate geohash    │      │
│      │                        │ (precision 6)        │      │
│      │                        │                      │      │
│      │                        │ GEOADD               │      │
│      │                        │─────────────────────▶│      │
│      │                        │                      │      │
│      │                        │ Update status        │      │
│      │                        │─────────────────────▶│      │
│      │                        │                      │      │
│      │         200 OK         │                      │      │
│      │◀───────────────────────│                      │      │
│      │                        │                      │      │
│      │                        │ (Async) Log to       │      │
│      │                        │ Kafka for history   │      │
│      │                        │                      │      │
│                                                             │
│  OPTIMIZATION:                                              │
│  - Batch updates from driver (send 3-5 at once)          │
│  - Client-side deduplication if position unchanged       │
│  - Skip update if moved < 10 meters                       │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 6: Deep Dive - Matching Algorithm

### The Matching Problem

```
┌─────────────────────────────────────────────────────────────┐
│              DRIVER MATCHING PROBLEM                        │
│                                                             │
│  INPUTS:                                                    │
│  - Rider pickup location                                   │
│  - Available drivers nearby                                │
│                                                             │
│  GOAL:                                                      │
│  - Find the BEST driver, not just the nearest             │
│                                                             │
│  FACTORS TO CONSIDER:                                       │
│  1. Distance to rider (ETA)                               │
│  2. Driver rating                                          │
│  3. Driver acceptance rate                                 │
│  4. Ride direction (going same way as driver?)           │
│  5. Driver earnings today (fairness)                      │
│  6. Vehicle type match                                     │
│                                                             │
│  SIMPLE SCORING:                                            │
│  score = w1 × (1/ETA) + w2 × rating + w3 × acceptance_rate│
│                                                             │
│  Pick driver with highest score                           │
└─────────────────────────────────────────────────────────────┘
```

### Matching Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      MATCHING ALGORITHM FLOW                                 │
│                                                                              │
│   Rider                Matching             Location           Driver        │
│     │                  Service              Service              │           │
│     │                     │                    │                 │           │
│     │ Request ride        │                    │                 │           │
│     │ (pickup, dest)      │                    │                 │           │
│     │────────────────────▶│                    │                 │           │
│     │                     │                    │                 │           │
│     │                     │ Get nearby drivers │                 │           │
│     │                     │ (5km radius)       │                 │           │
│     │                     │───────────────────▶│                 │           │
│     │                     │                    │                 │           │
│     │                     │ [driver1, driver2, │                 │           │
│     │                     │  driver3, ...]     │                 │           │
│     │                     │◀───────────────────│                 │           │
│     │                     │                    │                 │           │
│     │                     │ Filter: available, │                 │           │
│     │                     │ correct vehicle    │                 │           │
│     │                     │                    │                 │           │
│     │                     │ Calculate ETA for  │                 │           │
│     │                     │ each (batch call   │                 │           │
│     │                     │ to Maps API)       │                 │           │
│     │                     │                    │                 │           │
│     │                     │ Score each driver  │                 │           │
│     │                     │                    │                 │           │
│     │                     │ Select top driver  │                 │           │
│     │                     │────────────────────┼────────────────▶│           │
│     │                     │                    │   Ride request  │           │
│     │                     │                    │   (push notif)  │           │
│     │                     │                    │                 │           │
│     │                     │◀───────────────────┼─────────────────│           │
│     │                     │                    │   Accept/Reject │           │
│     │                     │                    │                 │           │
│     │  Driver assigned    │                    │                 │           │
│     │◀────────────────────│                    │                 │           │
│     │                     │                    │                 │           │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Handling Driver Rejection

```
┌─────────────────────────────────────────────────────────────┐
│           REJECTION HANDLING                                │
│                                                             │
│  SCENARIO: Selected driver rejects the ride               │
│                                                             │
│  APPROACH 1: Sequential (Simple)                          │
│  ─────────────────────────────────                        │
│  1. Ask driver 1, wait 15 seconds                        │
│  2. If reject/timeout, ask driver 2                      │
│  3. Repeat until accepted or no drivers                  │
│                                                             │
│  PROBLEM: Can take 60+ seconds if multiple rejections    │
│                                                             │
│  APPROACH 2: Parallel with timeout (Better)               │
│  ─────────────────────────────────────────                │
│  1. Send request to top 3 drivers simultaneously         │
│  2. First to accept gets the ride                        │
│  3. Cancel others                                         │
│                                                             │
│  ADVANTAGE: Faster matching                               │
│  CHALLENGE: Driver sees ride "disappear" if another accepts│
│                                                             │
│  UBER'S APPROACH: Hybrid                                   │
│  ─────────────────────────────────                        │
│  1. Send to best driver with 10-second timeout           │
│  2. If no response, expand to next 2-3 drivers           │
│  3. Dynamically adjust based on acceptance rates         │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 7: Deep Dive - Real-Time Tracking

### WebSocket Architecture

```
┌─────────────────────────────────────────────────────────────┐
│           REAL-TIME TRACKING                                │
│                                                             │
│  REQUIREMENTS:                                              │
│  - Rider sees driver location in real-time                │
│  - Updates every 2-4 seconds                              │
│  - Low latency (< 500ms)                                  │
│                                                             │
│  ARCHITECTURE:                                              │
│                                                             │
│   Driver App          WebSocket           Rider App        │
│       │               Servers                │              │
│       │                  │                   │              │
│       │ Location update  │                   │              │
│       │─────────────────▶│                   │              │
│       │                  │                   │              │
│       │                  │ Lookup: Which     │              │
│       │                  │ rider is tracking │              │
│       │                  │ this driver?     │              │
│       │                  │                   │              │
│       │                  │ Push location     │              │
│       │                  │──────────────────▶│              │
│       │                  │                   │              │
│       │                  │                   │ Update map   │
│       │                  │                   │              │
│                                                             │
│  WEBSOCKET SERVER DESIGN:                                   │
│  ┌────────────────────────────────────────────────────┐   │
│  │                                                      │   │
│  │  Connection Manager                                  │   │
│  │  - Map: user_id → WebSocket connection             │   │
│  │  - Map: ride_id → [rider_conn, driver_conn]       │   │
│  │                                                      │   │
│  │  When location update received:                     │   │
│  │  1. Find active ride for this driver               │   │
│  │  2. Get rider's WebSocket connection               │   │
│  │  3. Push location to rider                         │   │
│  │                                                      │   │
│  └────────────────────────────────────────────────────┘   │
│                                                             │
│  SCALING WEBSOCKETS:                                        │
│  - Each server handles ~100K connections                  │
│  - Use Redis Pub/Sub for cross-server communication       │
│  - Sticky sessions by ride_id                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 8: Surge Pricing

```
┌─────────────────────────────────────────────────────────────┐
│              SURGE PRICING                                  │
│                                                             │
│  CONCEPT:                                                   │
│  When demand > supply, increase prices to:                │
│  1. Encourage more drivers to come online                 │
│  2. Reduce rider demand                                    │
│  3. Balance the market                                     │
│                                                             │
│  IMPLEMENTATION:                                            │
│                                                             │
│  1. DIVIDE CITY INTO ZONES (H3 hexagons or geohash)      │
│     ┌─────┬─────┬─────┐                                   │
│     │ 1.0 │ 1.2 │ 1.0 │  (surge multiplier per zone)    │
│     ├─────┼─────┼─────┤                                   │
│     │ 1.5 │ 2.0 │ 1.3 │  Zone with 2.0x = high demand   │
│     ├─────┼─────┼─────┤                                   │
│     │ 1.0 │ 1.2 │ 1.0 │                                   │
│     └─────┴─────┴─────┘                                   │
│                                                             │
│  2. CALCULATE SURGE PER ZONE (every 1-5 minutes):        │
│                                                             │
│     demand = ride_requests_last_5_min                     │
│     supply = available_drivers_in_zone                    │
│                                                             │
│     ratio = demand / supply                               │
│                                                             │
│     surge = {                                              │
│       ratio < 1.0: 1.0 (no surge)                        │
│       ratio 1.0-1.5: 1.2                                  │
│       ratio 1.5-2.0: 1.5                                  │
│       ratio 2.0-3.0: 2.0                                  │
│       ratio > 3.0: 2.5 (cap)                             │
│     }                                                      │
│                                                             │
│  3. STORE IN REDIS:                                        │
│     Key: surge:{zone_id}                                  │
│     Value: multiplier                                      │
│     TTL: 5 minutes                                        │
│                                                             │
│  4. APPLY AT RIDE REQUEST:                                 │
│     - Look up zone for pickup location                   │
│     - Get surge multiplier                                │
│     - base_fare × surge = final_fare                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 9: Data Model

```
┌─────────────────────────────────────────────────────────────┐
│                    DATA MODEL                               │
│                                                             │
│  USERS TABLE (MySQL)                                        │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ user_id        BIGINT       PRIMARY KEY              │ │
│  │ type           ENUM         (rider, driver, both)    │ │
│  │ name           VARCHAR(100)                          │ │
│  │ email          VARCHAR(255) UNIQUE                   │ │
│  │ phone          VARCHAR(20)  UNIQUE                   │ │
│  │ rating         DECIMAL(3,2) DEFAULT 5.0             │ │
│  │ created_at     TIMESTAMP                            │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  DRIVERS TABLE (MySQL)                                      │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ driver_id      BIGINT       PRIMARY KEY (=user_id)   │ │
│  │ vehicle_type   ENUM         (UberX, UberXL, Black)  │ │
│  │ license_plate  VARCHAR(20)                           │ │
│  │ vehicle_model  VARCHAR(100)                          │ │
│  │ status         ENUM         (offline, available,     │ │
│  │                              on_trip)                │ │
│  │ acceptance_rate DECIMAL(3,2)                         │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  RIDES TABLE (Sharded by ride_id)                         │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ ride_id        BIGINT       PRIMARY KEY              │ │
│  │ rider_id       BIGINT       FOREIGN KEY              │ │
│  │ driver_id      BIGINT       FOREIGN KEY              │ │
│  │ status         ENUM         (requested, matched,     │ │
│  │                              started, completed,     │ │
│  │                              cancelled)              │ │
│  │ pickup_lat     DECIMAL(10,7)                         │ │
│  │ pickup_lng     DECIMAL(10,7)                         │ │
│  │ dest_lat       DECIMAL(10,7)                         │ │
│  │ dest_lng       DECIMAL(10,7)                         │ │
│  │ requested_at   TIMESTAMP                            │ │
│  │ started_at     TIMESTAMP                            │ │
│  │ completed_at   TIMESTAMP                            │ │
│  │ fare_amount    DECIMAL(10,2)                         │ │
│  │ surge_mult     DECIMAL(3,2)                          │ │
│  │ distance_km    DECIMAL(10,2)                         │ │
│  │                                                       │ │
│  │ INDEX (rider_id, requested_at)                      │ │
│  │ INDEX (driver_id, requested_at)                     │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  DRIVER_LOCATIONS (Redis GEO)                              │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ Key: driver_locations                               │ │
│  │ Type: GEO (built on Sorted Set)                    │ │
│  │ Members: driver_id with lat/lng                    │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  LOCATION_HISTORY (Cassandra - Time series)               │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ Partition Key: (driver_id, date)                    │ │
│  │ Clustering Key: timestamp DESC                      │ │
│  │ Columns: lat, lng, speed, heading                  │ │
│  │ TTL: 7 days                                         │ │
│  └──────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 10: Complete Ride Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         COMPLETE RIDE LIFECYCLE                              │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ PHASE 1: REQUEST                                                      │  │
│  │ 1. Rider opens app, sees nearby drivers (from Location Service)     │  │
│  │ 2. Rider enters destination, requests ride                          │  │
│  │ 3. System calculates fare estimate (with surge if applicable)       │  │
│  │ 4. Rider confirms request                                            │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                     │                                        │
│                                     ▼                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ PHASE 2: MATCHING                                                     │  │
│  │ 1. Matching Service queries nearby available drivers                │  │
│  │ 2. Calculates ETA for each driver (Maps API)                        │  │
│  │ 3. Scores and ranks drivers                                          │  │
│  │ 4. Sends ride request to top driver                                 │  │
│  │ 5. Driver accepts (or reject → try next driver)                    │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                     │                                        │
│                                     ▼                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ PHASE 3: PICKUP                                                       │  │
│  │ 1. Driver navigates to pickup (receives turn-by-turn)              │  │
│  │ 2. Rider tracks driver in real-time (WebSocket)                    │  │
│  │ 3. Driver arrives, notifies rider                                   │  │
│  │ 4. Rider gets in car                                                 │  │
│  │ 5. Driver starts trip                                                │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                     │                                        │
│                                     ▼                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ PHASE 4: TRIP                                                         │  │
│  │ 1. Driver follows navigation to destination                         │  │
│  │ 2. Location tracked every 4 seconds                                 │  │
│  │ 3. Rider can share trip with contacts                               │  │
│  │ 4. System tracks actual route for fare calculation                  │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                     │                                        │
│                                     ▼                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ PHASE 5: COMPLETION                                                   │  │
│  │ 1. Driver ends trip at destination                                  │  │
│  │ 2. Final fare calculated (distance × rate + time + surge)          │  │
│  │ 3. Payment processed (via Stripe/stored card)                       │  │
│  │ 4. Both parties prompted to rate                                    │  │
│  │ 5. Receipt sent via email                                           │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Step 11: Final Architecture Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      COMPLETE UBER ARCHITECTURE                              │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                         MOBILE CLIENTS                                   ││
│  │              Rider App                    Driver App                    ││
│  │   (iOS/Android, React Native)      (iOS/Android, React Native)         ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                         │
│  ┌─────────────────────────────────▼───────────────────────────────────────┐│
│  │                   EDGE / GATEWAY LAYER                                   ││
│  │   CDN │ WAF │ Load Balancer │ API Gateway │ WebSocket Gateway          ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                         │
│  ┌───────────┬──────────┬──────────┼──────────┬──────────┬────────────────┐│
│  │           │          │          │          │          │                ││
│  ▼           ▼          ▼          ▼          ▼          ▼                ││
│ ┌─────┐  ┌───────┐  ┌────────┐  ┌─────┐  ┌───────┐  ┌─────────┐         ││
│ │User │  │Locatio│  │Matching│  │Ride │  │Pricing│  │Notifica-│         ││
│ │Svc  │  │n Svc  │  │Service │  │Svc  │  │Service│  │tion Svc │         ││
│ └──┬──┘  └───┬───┘  └────┬───┘  └──┬──┘  └───┬───┘  └────┬────┘         ││
│    │         │           │         │         │           │               ││
│  ┌─┴─────────┴───────────┴─────────┴─────────┴───────────┴─────────────┐ ││
│  │                        DATA LAYER                                    │ ││
│  │                                                                      │ ││
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐        │ ││
│  │  │  User DB  │  │ Location  │  │  Ride DB  │  │  Surge    │        │ ││
│  │  │  (MySQL)  │  │  (Redis   │  │  (MySQL   │  │  (Redis)  │        │ ││
│  │  │           │  │   GEO)    │  │  Sharded) │  │           │        │ ││
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘        │ ││
│  │                                                                      │ ││
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐                       │ ││
│  │  │  Session  │  │ Location  │  │  Payment  │                       │ ││
│  │  │  (Redis)  │  │  History  │  │  (Stripe) │                       │ ││
│  │  │           │  │(Cassandra)│  │           │                       │ ││
│  │  └───────────┘  └───────────┘  └───────────┘                       │ ││
│  └──────────────────────────────────────────────────────────────────────┘ ││
│                                                                            ││
│  ┌──────────────────────────────────────────────────────────────────────┐ ││
│  │                      ASYNC LAYER (Kafka)                              │ ││
│  │    Location Events │ Trip Events │ Payment Events │ Analytics       │ ││
│  └──────────────────────────────────────────────────────────────────────┘ ││
│                                                                            ││
│  ┌──────────────────────────────────────────────────────────────────────┐ ││
│  │                    EXTERNAL SERVICES                                  │ ││
│  │   Maps/Routing (Google) │ Payment (Stripe) │ SMS (Twilio)           │ ││
│  └──────────────────────────────────────────────────────────────────────┘ ││
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Step 12: Tradeoffs & Discussion Points

```
┌─────────────────────────────────────────────────────────────┐
│              KEY TRADEOFFS                                  │
│                                                             │
│  1. LOCATION STORAGE: Redis GEO vs PostGIS               │
│     We chose Redis: In-memory, fast writes, ephemeral data│
│                                                             │
│  2. MATCHING: Nearest vs Best driver                      │
│     We chose: Scoring algorithm balancing multiple factors│
│                                                             │
│  3. REAL-TIME: Polling vs WebSocket                       │
│     We chose: WebSocket for lower latency, less traffic   │
│                                                             │
│  4. SURGE: Real-time vs Pre-computed                      │
│     We chose: Pre-computed every 2-5 min for efficiency   │
│                                                             │
│  POTENTIAL FOLLOW-UP QUESTIONS:                            │
│  ─────────────────────────────────────────                 │
│  • "How would you add carpooling (UberPool)?"             │
│  • "How would you handle multi-city deployment?"          │
│  • "How would you prevent fraud?"                         │
│  • "How would you add scheduled rides?"                   │
│  • "How would you handle driver preferences?"             │
│  • "How would you ensure fairness for drivers?"           │
└─────────────────────────────────────────────────────────────┘
```

---

## Interview Evaluation Criteria

```
┌─────────────────────────────────────────────────────────────┐
│         WHAT INTERVIEWERS LOOK FOR                          │
│                                                             │
│  ✓ GEOSPATIAL UNDERSTANDING                                 │
│    - Knows about geohash or similar indexing               │
│    - Understands radius queries                            │
│                                                             │
│  ✓ REAL-TIME SYSTEMS                                        │
│    - WebSocket or long-polling discussion                  │
│    - Location update frequency tradeoffs                   │
│                                                             │
│  ✓ WRITE-HEAVY DESIGN                                       │
│    - Recognizes 190K writes/sec challenge                  │
│    - Appropriate storage choice (Redis, Cassandra)        │
│                                                             │
│  ✓ MATCHING ALGORITHM                                       │
│    - More than just "nearest driver"                      │
│    - Considers multiple factors                           │
│                                                             │
│  ✓ DYNAMIC PRICING                                          │
│    - Supply/demand concept                                 │
│    - Zone-based implementation                            │
│                                                             │
│  ✓ SCALE AWARENESS                                          │
│    - Sharding strategy                                     │
│    - Caching strategy                                      │
│    - Multi-city considerations                            │
└─────────────────────────────────────────────────────────────┘
```

---

**Congratulations!** You've completed the High-Level Design section. You should now have a solid foundation for tackling any system design interview.

**Key Takeaways:**
1. Always start with requirements
2. Do capacity estimation to inform decisions
3. Draw clear architecture diagrams
4. Deep dive into the hardest problems
5. Discuss tradeoffs openly
