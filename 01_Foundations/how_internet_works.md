# How the Internet Works

## The Simple Explanation

When you type "google.com" in your browser, a LOT happens in the next 100 milliseconds:

```
┌─────────────────────────────────────────────────────────────────┐
│                  WHAT HAPPENS IN 100ms                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Your browser asks: "Where is google.com?"     (DNS)         │
│  2. DNS replies: "It's at 142.250.80.46"                        │
│  3. Your computer connects to that address        (TCP)         │
│  4. They shake hands to establish trust           (TLS)         │
│  5. Your browser asks for the webpage             (HTTP)        │
│  6. Google's server sends back the page                         │
│  7. Your browser displays it                                    │
│                                                                 │
│  Total time: ~100-500 milliseconds                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Let's understand each piece.

---

## Part 1: DNS (Domain Name System)

### What is DNS?

**Analogy:** DNS is like a phone book for the internet.

- You know your friend's name (google.com)
- But you need their phone number (142.250.80.46) to call them
- DNS translates names to numbers

```
┌────────────────────────────────────────────────────────────────┐
│                        DNS LOOKUP                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   You: "I want to visit google.com"                            │
│                │                                               │
│                ▼                                               │
│   ┌─────────────────────┐                                      │
│   │   DNS Resolver      │  (Usually your ISP)                  │
│   │   "Let me look      │                                      │
│   │    that up..."      │                                      │
│   └──────────┬──────────┘                                      │
│              │                                                 │
│              ▼                                                 │
│   ┌─────────────────────┐                                      │
│   │   Root DNS Server   │  "Ask .com servers"                  │
│   └──────────┬──────────┘                                      │
│              │                                                 │
│              ▼                                                 │
│   ┌─────────────────────┐                                      │
│   │   .com DNS Server   │  "Ask Google's servers"              │
│   └──────────┬──────────┘                                      │
│              │                                                 │
│              ▼                                                 │
│   ┌─────────────────────┐                                      │
│   │  Google DNS Server  │  "It's 142.250.80.46"                │
│   └──────────┬──────────┘                                      │
│              │                                                 │
│              ▼                                                 │
│   Your Browser: "Got it! Connecting..."                        │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### DNS Caching

Because DNS lookups take time (~20-100ms), results are cached at multiple levels:

```
CACHING HIERARCHY (Checked in order):

1. Browser Cache        (~2 minutes TTL)
   ↓ miss
2. Operating System     (~5 minutes TTL)
   ↓ miss
3. Router Cache         (~24 hours TTL)
   ↓ miss
4. ISP DNS Resolver     (~hours to days)
   ↓ miss
5. Full DNS Resolution  (recursive lookup)
```

### DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Maps name to IPv4 | google.com → 142.250.80.46 |
| **AAAA** | Maps name to IPv6 | google.com → 2607:f8b0:4004:800::200e |
| **CNAME** | Alias to another name | www.google.com → google.com |
| **MX** | Mail server | gmail.com → mx.google.com |
| **NS** | Name server | google.com → ns1.google.com |
| **TXT** | Text data | Used for verification |

### Interview Relevance

**Why DNS matters in system design:**
- DNS can be a single point of failure
- DNS propagation takes time (TTL)
- Geographic DNS routing for global services
- DNS load balancing possibilities

---

## Part 2: TCP/IP (The Transport Layer)

### IP Addresses

Every device on the internet has an IP address—like a home address for your computer.

```
IPv4: 192.168.1.1       (32 bits, ~4 billion addresses)
IPv6: 2001:0db8:85a3... (128 bits, essentially infinite)
```

### What is TCP?

**Analogy:** TCP is like sending a registered letter with delivery confirmation.

TCP ensures:
1. **Reliable delivery** - Data arrives intact
2. **Ordered delivery** - Packets arrive in sequence
3. **Error checking** - Corrupted data is resent
4. **Flow control** - Sender doesn't overwhelm receiver

### The TCP Handshake

Before any data is sent, TCP does a "three-way handshake":

```
┌─────────────────────────────────────────────────────────────────┐
│                   TCP THREE-WAY HANDSHAKE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│     Client                                   Server             │
│        │                                        │               │
│        │───────── SYN (seq=x) ─────────────────►│               │
│        │        "Hey, want to connect?"         │               │
│        │                                        │               │
│        │◄──────── SYN-ACK (seq=y, ack=x+1) ─────│               │
│        │        "Sure! I heard you."            │               │
│        │                                        │               │
│        │───────── ACK (ack=y+1) ────────────────►│               │
│        │        "Great, confirmed!"             │               │
│        │                                        │               │
│        │◄═══════ CONNECTION ESTABLISHED ═══════►│               │
│        │                                        │               │
└─────────────────────────────────────────────────────────────────┘

This takes 1 round-trip time (RTT) before data can flow.
```

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented | Connectionless |
| **Reliability** | Guaranteed delivery | Best effort |
| **Ordering** | Ordered packets | No ordering |
| **Speed** | Slower (overhead) | Faster |
| **Use case** | Web, email, file transfer | Video streaming, gaming, DNS |

```
WHEN TO USE WHAT:

TCP (Transmission Control Protocol):
├── Web browsing (HTTP/HTTPS)
├── Email
├── File transfers
└── When you CANNOT lose data

UDP (User Datagram Protocol):
├── Live video streaming
├── Online gaming
├── DNS queries
├── VoIP calls
└── When SPEED matters more than perfection
```

### Ports

Ports are like apartment numbers in a building (IP address):

```
IP Address: 192.168.1.1     (The building)
Port: 443                    (The apartment)

Combined: 192.168.1.1:443    (Full address)
```

**Common Ports:**
| Port | Service |
|------|---------|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 27017 | MongoDB |

---

## Part 3: HTTP/HTTPS (Application Layer)

### What is HTTP?

HTTP (HyperText Transfer Protocol) is how browsers and servers communicate.

```
┌─────────────────────────────────────────────────────────────────┐
│                    HTTP REQUEST/RESPONSE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENT                                              SERVER     │
│    │                                                    │       │
│    │──────── HTTP REQUEST ────────────────────────────►│       │
│    │                                                    │       │
│    │   GET /index.html HTTP/1.1                        │       │
│    │   Host: www.example.com                           │       │
│    │   User-Agent: Chrome/91.0                         │       │
│    │   Accept: text/html                               │       │
│    │                                                    │       │
│    │◄─────── HTTP RESPONSE ────────────────────────────│       │
│    │                                                    │       │
│    │   HTTP/1.1 200 OK                                 │       │
│    │   Content-Type: text/html                         │       │
│    │   Content-Length: 1234                            │       │
│    │                                                    │       │
│    │   <html>...</html>                                │       │
│    │                                                    │       │
└─────────────────────────────────────────────────────────────────┘
```

### HTTP Methods

| Method | Purpose | Idempotent? | Safe? |
|--------|---------|-------------|-------|
| **GET** | Retrieve data | Yes | Yes |
| **POST** | Create resource | No | No |
| **PUT** | Replace resource | Yes | No |
| **PATCH** | Partial update | No | No |
| **DELETE** | Remove resource | Yes | No |
| **HEAD** | Get headers only | Yes | Yes |
| **OPTIONS** | Get allowed methods | Yes | Yes |

### HTTP Status Codes

```
2xx: SUCCESS
├── 200 OK                 - Request succeeded
├── 201 Created            - Resource created
└── 204 No Content         - Success, no body

3xx: REDIRECTION
├── 301 Moved Permanently  - Resource moved (cached)
├── 302 Found              - Temporary redirect
└── 304 Not Modified       - Use cached version

4xx: CLIENT ERROR
├── 400 Bad Request        - Invalid request
├── 401 Unauthorized       - Authentication needed
├── 403 Forbidden          - Permission denied
├── 404 Not Found          - Resource doesn't exist
└── 429 Too Many Requests  - Rate limited

5xx: SERVER ERROR
├── 500 Internal Error     - Server broke
├── 502 Bad Gateway        - Upstream server error
├── 503 Service Unavailable- Server overloaded
└── 504 Gateway Timeout    - Upstream timeout
```

### HTTPS: HTTP + Security

HTTPS adds TLS (Transport Layer Security) encryption:

```
┌─────────────────────────────────────────────────────────────────┐
│                    TLS HANDSHAKE (Simplified)                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Client                                              Server     │
│    │                                                    │       │
│    │──── "Hello, I support these encryption methods" ──►│       │
│    │                                                    │       │
│    │◄─── "Hello, let's use AES-256. Here's my cert" ───│       │
│    │                                                    │       │
│    │     [Client verifies certificate]                  │       │
│    │                                                    │       │
│    │──── "Here's a secret key (encrypted)" ────────────►│       │
│    │                                                    │       │
│    │◄═══════ ENCRYPTED CONNECTION ═════════════════════►│       │
│    │                                                    │       │
│    │     All data is now encrypted                      │       │
│    │                                                    │       │
└─────────────────────────────────────────────────────────────────┘

This adds 1-2 more round trips before data flows.
```

### HTTP/1.1 vs HTTP/2 vs HTTP/3

```
HTTP/1.1 (1997):
├── One request per connection
├── Head-of-line blocking
└── Text-based headers

HTTP/2 (2015):
├── Multiple requests per connection (multiplexing)
├── Server push capability
├── Binary protocol
└── Header compression

HTTP/3 (2022):
├── Uses QUIC (UDP-based)
├── Faster connection setup
├── Better handling of packet loss
└── Built-in encryption
```

---

## Putting It All Together

### Complete Request Flow

```
┌─────────────────────────────────────────────────────────────────┐
│              FULL JOURNEY: You → google.com                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. YOU TYPE "google.com"                                       │
│     │                                                           │
│  2. │ DNS RESOLUTION (~20ms)                                    │
│     │   Browser cache? → OS cache? → Router? → ISP? → Full DNS │
│     │   Result: 142.250.80.46                                   │
│     │                                                           │
│  3. │ TCP HANDSHAKE (~30ms)                                     │
│     │   SYN → SYN-ACK → ACK                                     │
│     │   Connection established                                  │
│     │                                                           │
│  4. │ TLS HANDSHAKE (~50ms)                                     │
│     │   Client Hello → Server Hello → Key Exchange              │
│     │   Secure channel established                              │
│     │                                                           │
│  5. │ HTTP REQUEST                                              │
│     │   GET / HTTP/2                                            │
│     │   Host: google.com                                        │
│     │                                                           │
│  6. │ GOOGLE'S SERVERS                                          │
│     │   Load balancer → Web server → Business logic             │
│     │   Generates HTML response                                 │
│     │                                                           │
│  7. │ HTTP RESPONSE (~50ms)                                     │
│     │   200 OK + HTML content                                   │
│     │                                                           │
│  8. ▼ BROWSER RENDERS                                           │
│       Parse HTML → Fetch CSS/JS → Render page                   │
│                                                                 │
│  TOTAL: ~200-500ms                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Deep Dives

### Question: "What happens when you type google.com?"

This is a CLASSIC interview question. Here's a complete answer framework:

```
COMPLETE ANSWER STRUCTURE:

1. URL Parsing
   └── Browser parses the URL, determines protocol (HTTPS)

2. DNS Resolution
   └── Check caches → Recursive DNS lookup → Get IP

3. TCP Connection
   └── Three-way handshake to establish connection

4. TLS Handshake
   └── Exchange certificates, agree on encryption

5. HTTP Request
   └── Send GET request with headers

6. Server Processing
   └── Load balancer → Web server → Application → Database

7. HTTP Response
   └── Server sends HTML, status codes, headers

8. Browser Rendering
   └── Parse HTML, build DOM, fetch resources, paint
```

### Question: "How would you optimize this flow?"

```
OPTIMIZATION STRATEGIES:

1. DNS
   ├── Use DNS prefetching
   ├── Lower TTL for dynamic content
   └── DNS over HTTPS (DoH) for privacy

2. Connection
   ├── HTTP/2 for multiplexing
   ├── Connection keep-alive
   └── TCP Fast Open

3. TLS
   ├── TLS 1.3 (faster handshake)
   ├── Session resumption
   └── OCSP stapling

4. HTTP
   ├── HTTP/2 server push
   ├── Compression (gzip, brotli)
   └── Caching headers

5. Application
   ├── CDN for static content
   ├── Edge computing
   └── Response streaming
```

---

## Common Pitfalls

### Pitfall 1: Forgetting DNS propagation time
When you change DNS records, it can take hours to days for all caches to update.

### Pitfall 2: Assuming TCP is instant
TCP handshake adds latency. For global services, this matters!

### Pitfall 3: Ignoring HTTPS overhead
TLS adds 1-2 round trips. Use TLS 1.3 and session resumption to minimize.

### Pitfall 4: Not understanding HTTP/2 benefits
HTTP/2 multiplexing eliminates head-of-line blocking at the HTTP level.

---

## Key Numbers to Remember

| Operation | Time |
|-----------|------|
| DNS lookup (cached) | 0-1 ms |
| DNS lookup (uncached) | 20-100 ms |
| TCP handshake (same region) | 1-10 ms |
| TCP handshake (cross-continent) | 100-200 ms |
| TLS handshake | 50-100 ms |
| HTTP request/response | Depends on payload |

---

## Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│                    REMEMBER THIS                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. DNS translates domain names to IP addresses                │
│                                                                │
│  2. TCP provides reliable, ordered data delivery               │
│                                                                │
│  3. HTTP is the protocol for web communication                 │
│                                                                │
│  4. HTTPS = HTTP + TLS encryption                              │
│                                                                │
│  5. Each step adds latency - optimize the critical path        │
│                                                                │
│  6. Caching at every layer is crucial for performance          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

Now that you understand how data travels across the internet, let's learn about measuring and optimizing that journey.

**Next:** [Latency vs Throughput →](latency_throughput.md)
