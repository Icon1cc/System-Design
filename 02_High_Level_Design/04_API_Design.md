# API Design Best Practices

## What is API Design in System Design?

API design defines **how clients interact with your system**. It's the contract between frontend and backend, between services, and between your system and external partners.

```
┌─────────────────────────────────────────────────────────────┐
│                    API AS CONTRACT                          │
│                                                             │
│     CLIENT                 CONTRACT                SERVER   │
│    ┌───────┐              ┌───────┐             ┌───────┐  │
│    │Mobile │─────────────▶│  API  │────────────▶│Service│  │
│    │  Web  │◀─────────────│ Spec  │◀────────────│ Layer │  │
│    │  SDK  │              └───────┘             └───────┘  │
│    └───────┘                                               │
│                                                             │
│    "If I send X, I expect Y back in format Z"              │
└─────────────────────────────────────────────────────────────┘
```

## API Paradigms: REST vs GraphQL vs gRPC

### Quick Comparison

```
┌─────────────────────────────────────────────────────────────┐
│               API PARADIGM COMPARISON                       │
│                                                             │
│  ASPECT         REST        GraphQL       gRPC              │
│  ─────────────────────────────────────────────             │
│  Protocol       HTTP        HTTP          HTTP/2            │
│  Format         JSON        JSON          Protobuf          │
│  Schema         Optional    Required      Required          │
│  Caching        Easy        Complex       Moderate          │
│  Learning       Easy        Moderate      Harder            │
│  Over-fetching  Yes         No            No                │
│  Best for       CRUD apps   Complex UI    Microservices     │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Each

```
REST (Most Common):
├── CRUD operations
├── Public APIs
├── Simple request/response
└── When caching is important

GraphQL:
├── Complex, nested data
├── Mobile apps (minimize requests)
├── Rapid frontend iteration
└── Multiple client types

gRPC:
├── Microservice communication
├── High performance needs
├── Streaming data
└── Polyglot services
```

## RESTful API Design

### Resource Naming Conventions

```
┌─────────────────────────────────────────────────────────────┐
│              REST NAMING CONVENTIONS                        │
│                                                             │
│  USE NOUNS (Resources), NOT VERBS (Actions)                │
│                                                             │
│  ✅ GOOD                    ❌ BAD                          │
│  GET /users                 GET /getUsers                   │
│  POST /users                POST /createUser                │
│  GET /users/123             GET /getUserById?id=123         │
│  DELETE /users/123          POST /deleteUser                │
│                                                             │
│  USE PLURAL NOUNS                                           │
│  ✅ /users                  ❌ /user                        │
│  ✅ /tweets                 ❌ /tweet                       │
│  ✅ /orders                 ❌ /order                       │
└─────────────────────────────────────────────────────────────┘
```

### HTTP Methods

```
┌─────────────────────────────────────────────────────────────┐
│                  HTTP METHODS                               │
│                                                             │
│  METHOD    PURPOSE           IDEMPOTENT    SAFE            │
│  ───────────────────────────────────────────               │
│  GET       Read resource     Yes           Yes             │
│  POST      Create resource   No            No              │
│  PUT       Replace resource  Yes           No              │
│  PATCH     Update resource   No            No              │
│  DELETE    Delete resource   Yes           No              │
│                                                             │
│  IDEMPOTENT: Same request = same result (no side effects)  │
│  SAFE: Doesn't modify server state                         │
└─────────────────────────────────────────────────────────────┘
```

### URL Structure Examples

**Twitter-like API:**
```
Users:
GET    /api/v1/users                    # List users
POST   /api/v1/users                    # Create user
GET    /api/v1/users/{user_id}          # Get user
PUT    /api/v1/users/{user_id}          # Update user
DELETE /api/v1/users/{user_id}          # Delete user

Tweets:
GET    /api/v1/tweets                   # List tweets
POST   /api/v1/tweets                   # Create tweet
GET    /api/v1/tweets/{tweet_id}        # Get tweet
DELETE /api/v1/tweets/{tweet_id}        # Delete tweet

Nested Resources:
GET    /api/v1/users/{user_id}/tweets   # User's tweets
GET    /api/v1/users/{user_id}/followers # User's followers
POST   /api/v1/tweets/{tweet_id}/likes  # Like a tweet
DELETE /api/v1/tweets/{tweet_id}/likes  # Unlike a tweet

Timeline:
GET    /api/v1/timeline/home            # Home timeline
GET    /api/v1/timeline/user/{user_id}  # User timeline
```

### Request/Response Format

**Create Tweet Example:**
```
Request:
POST /api/v1/tweets
Headers:
  Authorization: Bearer <token>
  Content-Type: application/json

Body:
{
  "content": "Hello, world!",
  "media_ids": ["abc123", "def456"],
  "reply_to": null,
  "visibility": "public"
}

Response (201 Created):
{
  "id": "tweet_789xyz",
  "content": "Hello, world!",
  "author": {
    "id": "user_123",
    "username": "johndoe",
    "display_name": "John Doe",
    "avatar_url": "https://..."
  },
  "media": [
    {
      "id": "abc123",
      "type": "image",
      "url": "https://..."
    }
  ],
  "created_at": "2024-01-15T10:30:00Z",
  "like_count": 0,
  "retweet_count": 0,
  "reply_count": 0,
  "visibility": "public"
}
```

## Pagination Strategies

### Offset-Based Pagination

```
┌─────────────────────────────────────────────────────────────┐
│              OFFSET PAGINATION                              │
│                                                             │
│  Request: GET /api/v1/tweets?offset=20&limit=10            │
│                                                             │
│  Page 1: offset=0,  limit=10  → items 1-10                 │
│  Page 2: offset=10, limit=10  → items 11-20                │
│  Page 3: offset=20, limit=10  → items 21-30                │
│                                                             │
│  PROS:                       CONS:                          │
│  ├── Simple to implement     ├── Slow for large offsets    │
│  ├── Random page access      ├── Inconsistent with inserts │
│  └── Familiar to users       └── O(offset) database cost   │
└─────────────────────────────────────────────────────────────┘

Response:
{
  "data": [...],
  "pagination": {
    "offset": 20,
    "limit": 10,
    "total": 1000,
    "has_more": true
  }
}
```

### Cursor-Based Pagination (Preferred for Feeds)

```
┌─────────────────────────────────────────────────────────────┐
│              CURSOR PAGINATION                              │
│                                                             │
│  Request: GET /api/v1/tweets?cursor=abc123&limit=10        │
│                                                             │
│  How it works:                                              │
│  1. First request: no cursor, get first 10                 │
│  2. Response includes cursor pointing to last item         │
│  3. Next request: use cursor to get next 10                │
│                                                             │
│  PROS:                       CONS:                          │
│  ├── O(1) database cost      ├── No random page access     │
│  ├── Consistent with inserts ├── Harder to implement       │
│  └── Scales to millions      └── Opaque cursor values      │
└─────────────────────────────────────────────────────────────┘

Response:
{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MTIzNDV9",
    "previous_cursor": "eyJpZCI6MTIzMzV9",
    "has_more": true
  }
}
```

### Keyset Pagination

```
┌─────────────────────────────────────────────────────────────┐
│              KEYSET PAGINATION                              │
│                                                             │
│  Request: GET /api/v1/tweets?after_id=12345&limit=10       │
│                                                             │
│  SQL: SELECT * FROM tweets                                  │
│       WHERE id < 12345                                      │
│       ORDER BY id DESC                                      │
│       LIMIT 10                                              │
│                                                             │
│  PROS:                       CONS:                          │
│  ├── Very efficient          ├── Requires sortable key     │
│  ├── Index-friendly          ├── Limited sort options      │
│  └── Predictable             └── Exposes internal IDs      │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Each

```
OFFSET: Admin dashboards, small datasets
CURSOR: Social feeds, infinite scroll
KEYSET: Time-series data, logs
```

## API Versioning

### Versioning Strategies

```
┌─────────────────────────────────────────────────────────────┐
│               API VERSIONING                                │
│                                                             │
│  1. URL PATH (Most Common)                                  │
│     /api/v1/users                                           │
│     /api/v2/users                                           │
│                                                             │
│  2. QUERY PARAMETER                                         │
│     /api/users?version=1                                    │
│     /api/users?version=2                                    │
│                                                             │
│  3. HEADER                                                  │
│     Accept: application/vnd.myapi.v1+json                  │
│     X-API-Version: 2                                        │
│                                                             │
│  RECOMMENDATION: Use URL path for simplicity               │
└─────────────────────────────────────────────────────────────┘
```

### Versioning Best Practices

```
DO:
├── Version from day one (/api/v1/...)
├── Support at least 2 versions simultaneously
├── Document deprecation timeline
└── Use semantic versioning for changes

DON'T:
├── Break existing clients without warning
├── Add required fields to existing endpoints
├── Change response structure within a version
└── Remove fields without deprecation period
```

## Error Handling

### HTTP Status Codes

```
┌─────────────────────────────────────────────────────────────┐
│              HTTP STATUS CODES                              │
│                                                             │
│  2xx SUCCESS                                                │
│  ├── 200 OK - Success                                       │
│  ├── 201 Created - Resource created                         │
│  ├── 202 Accepted - Async processing started               │
│  └── 204 No Content - Success, no body                     │
│                                                             │
│  4xx CLIENT ERROR                                           │
│  ├── 400 Bad Request - Invalid input                       │
│  ├── 401 Unauthorized - Not authenticated                  │
│  ├── 403 Forbidden - Not authorized                        │
│  ├── 404 Not Found - Resource doesn't exist                │
│  ├── 409 Conflict - Resource conflict                      │
│  ├── 422 Unprocessable - Validation error                  │
│  └── 429 Too Many Requests - Rate limited                  │
│                                                             │
│  5xx SERVER ERROR                                           │
│  ├── 500 Internal Server Error - Bug/crash                 │
│  ├── 502 Bad Gateway - Upstream failure                    │
│  ├── 503 Service Unavailable - Overloaded                  │
│  └── 504 Gateway Timeout - Upstream timeout                │
└─────────────────────────────────────────────────────────────┘
```

### Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters"
      }
    ],
    "request_id": "req_abc123xyz",
    "documentation_url": "https://api.example.com/docs/errors#VALIDATION_ERROR"
  }
}
```

### Error Handling Best Practices

```
DO:
├── Return consistent error format
├── Include actionable messages
├── Provide request IDs for debugging
├── Log errors with context
└── Never expose stack traces in production

DON'T:
├── Return 200 with error in body
├── Use generic "Something went wrong"
├── Expose internal details
└── Return different error formats
```

## Rate Limiting

### Rate Limiting Strategies

```
┌─────────────────────────────────────────────────────────────┐
│              RATE LIMITING                                  │
│                                                             │
│  COMMON STRATEGIES:                                         │
│  ├── Per user: 100 requests/minute                         │
│  ├── Per API key: 1000 requests/minute                     │
│  ├── Per IP: 50 requests/minute (unauthenticated)         │
│  └── Per endpoint: Different limits for expensive ops     │
│                                                             │
│  RESPONSE HEADERS:                                          │
│  X-RateLimit-Limit: 100                                    │
│  X-RateLimit-Remaining: 45                                 │
│  X-RateLimit-Reset: 1642524800                             │
│                                                             │
│  WHEN EXCEEDED:                                             │
│  HTTP 429 Too Many Requests                                │
│  Retry-After: 60                                           │
└─────────────────────────────────────────────────────────────┘
```

### Rate Limit Response

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 60 seconds.",
    "retry_after": 60
  }
}
```

## Authentication & Authorization

### Common Auth Methods

```
┌─────────────────────────────────────────────────────────────┐
│              AUTHENTICATION METHODS                         │
│                                                             │
│  1. API KEYS                                                │
│     Header: X-API-Key: sk_live_abc123                      │
│     Use for: Server-to-server, simple auth                 │
│                                                             │
│  2. JWT (JSON Web Tokens)                                   │
│     Header: Authorization: Bearer eyJhbGciOi...            │
│     Use for: Stateless auth, microservices                 │
│                                                             │
│  3. OAuth 2.0                                               │
│     Header: Authorization: Bearer <access_token>           │
│     Use for: Third-party access, user consent              │
│                                                             │
│  4. Session-based                                           │
│     Cookie: session_id=abc123                              │
│     Use for: Traditional web apps                          │
└─────────────────────────────────────────────────────────────┘
```

### JWT Structure

```
┌─────────────────────────────────────────────────────────────┐
│                  JWT STRUCTURE                              │
│                                                             │
│  eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo...signature         │
│  ─────────┬─────────   ─────────┬─────   ────┬────         │
│         Header               Payload      Signature         │
│                                                             │
│  Header: {"alg": "HS256", "typ": "JWT"}                    │
│                                                             │
│  Payload: {                                                 │
│    "user_id": "123",                                       │
│    "email": "user@example.com",                            │
│    "roles": ["user"],                                      │
│    "exp": 1642524800,                                      │
│    "iat": 1642521200                                       │
│  }                                                          │
│                                                             │
│  Signature: HMAC-SHA256(header + payload, secret)          │
└─────────────────────────────────────────────────────────────┘
```

## API Design for Interviews

### The API Design Template

When asked to design APIs in an interview:

```
1. LIST CORE OPERATIONS
   "For Twitter, core operations are: tweet, follow, timeline"

2. DEFINE ENDPOINTS
   POST /tweets        - Create tweet
   GET  /timeline/home - Get home timeline
   POST /users/{id}/follow - Follow user

3. SPECIFY REQUEST/RESPONSE
   Request body, response format, status codes

4. DISCUSS PAGINATION
   "Timeline uses cursor-based pagination for consistency"

5. MENTION AUTH & RATE LIMITS
   "JWT auth, 300 requests/minute per user"
```

### Sample Interview Answer

**"Design the APIs for a URL Shortener"**

```
CORE OPERATIONS:
1. Shorten URL
2. Redirect (get original URL)
3. Get analytics

ENDPOINTS:

1. Create Short URL
   POST /api/v1/urls
   Request:
   {
     "long_url": "https://example.com/very/long/path",
     "custom_alias": "my-link",  // optional
     "expires_at": "2025-01-01"  // optional
   }
   Response (201):
   {
     "short_url": "https://short.ly/abc123",
     "long_url": "https://example.com/very/long/path",
     "created_at": "2024-01-15T10:00:00Z",
     "expires_at": "2025-01-01T00:00:00Z"
   }

2. Redirect
   GET /{short_code}
   Response: 302 Redirect to long_url
   Location: https://example.com/very/long/path

3. Get Analytics
   GET /api/v1/urls/{short_code}/analytics
   Response:
   {
     "total_clicks": 1500,
     "clicks_by_day": [...],
     "top_referrers": [...],
     "top_countries": [...]
   }

4. Delete URL
   DELETE /api/v1/urls/{short_code}
   Response: 204 No Content

PAGINATION:
GET /api/v1/urls?cursor=abc&limit=20
- Cursor-based for user's URL list

AUTH:
- API key for server integrations
- JWT for web dashboard

RATE LIMITS:
- Free tier: 100 URLs/day
- Pro tier: 10,000 URLs/day
```

## Complete API Design: Twitter

```
┌─────────────────────────────────────────────────────────────┐
│              TWITTER API DESIGN                             │
└─────────────────────────────────────────────────────────────┘

USERS
─────
POST   /api/v1/users                    # Create account
GET    /api/v1/users/{user_id}          # Get profile
PATCH  /api/v1/users/{user_id}          # Update profile
GET    /api/v1/users/{user_id}/tweets   # User's tweets
GET    /api/v1/users/{user_id}/followers
GET    /api/v1/users/{user_id}/following

TWEETS
──────
POST   /api/v1/tweets                   # Create tweet
GET    /api/v1/tweets/{tweet_id}        # Get tweet
DELETE /api/v1/tweets/{tweet_id}        # Delete tweet
GET    /api/v1/tweets/{tweet_id}/replies

INTERACTIONS
────────────
POST   /api/v1/tweets/{tweet_id}/likes
DELETE /api/v1/tweets/{tweet_id}/likes
POST   /api/v1/tweets/{tweet_id}/retweets
DELETE /api/v1/tweets/{tweet_id}/retweets
POST   /api/v1/users/{user_id}/follow
DELETE /api/v1/users/{user_id}/follow

TIMELINE
────────
GET    /api/v1/timeline/home?cursor=X&limit=20
GET    /api/v1/timeline/user/{user_id}?cursor=X

SEARCH
──────
GET    /api/v1/search/tweets?q=keyword&cursor=X
GET    /api/v1/search/users?q=username&limit=10

MEDIA
─────
POST   /api/v1/media/upload             # Upload image/video
GET    /api/v1/media/{media_id}         # Get media metadata
```

## Common API Design Mistakes

```
┌─────────────────────────────────────────────────────────────┐
│            API DESIGN ANTI-PATTERNS                         │
│                                                             │
│  1. VERBS IN URLs                                           │
│     ❌ POST /createUser                                     │
│     ✅ POST /users                                          │
│                                                             │
│  2. INCONSISTENT NAMING                                     │
│     ❌ /users, /tweet, /Messages                           │
│     ✅ /users, /tweets, /messages                          │
│                                                             │
│  3. NESTED TOO DEEP                                         │
│     ❌ /users/1/tweets/2/comments/3/likes                  │
│     ✅ /comments/3/likes                                    │
│                                                             │
│  4. EXPOSING INTERNAL DETAILS                              │
│     ❌ /api/mysql/users/select                             │
│     ✅ /api/v1/users                                       │
│                                                             │
│  5. NO VERSIONING                                           │
│     ❌ /api/users                                          │
│     ✅ /api/v1/users                                       │
│                                                             │
│  6. RETURNING 200 FOR ERRORS                               │
│     ❌ 200 {"error": "Not found"}                          │
│     ✅ 404 {"error": {...}}                                │
└─────────────────────────────────────────────────────────────┘
```

## Interview Tips

1. **Start with operations** - List what users need to do
2. **Use REST conventions** - Unless there's a reason not to
3. **Design for the client** - Think about who's calling
4. **Mention pagination early** - Shows you think about scale
5. **Don't over-design** - 5-7 endpoints is usually enough
6. **Discuss tradeoffs** - REST vs GraphQL, sync vs async

---

**Next:** [05_Data_Modeling.md](./05_Data_Modeling.md) - Design schemas for your APIs
