# Part 4: Networking — HTTP, DNS, TCP, TLS, Caching 🌐

## Week 4 | 10 hours | Understand how data reaches the browser

---

## Why This Matters

Every API call, every asset load, every page navigation goes through the network.
Staff Engineers make architectural decisions about:
- REST vs GraphQL vs tRPC — needs deep HTTP understanding
- Caching strategies — save millions of requests
- CDN architecture — reduce latency globally
- HTTP/2 vs HTTP/3 — protocol-level performance
- Streaming responses — critical for AI features

**If you don't understand networking, you can't optimize it.**

---

## 📖 Reading Plan (3 hours)

### Must Read:
1. **"High Performance Browser Networking"** — Ilya Grigorik (FREE online)
   - https://hpbn.co/
   - Read chapters: 1 (Latency), 2 (TCP), 4 (TLS), 9 (HTTP/1.1), 10 (HTTP/2), 12 (HTTP/3)
   - This is THE networking book for web developers

2. **MDN: HTTP Overview**
   - https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview
   - Read: HTTP Caching, HTTP Headers, CORS

3. **web.dev: Network reliability**
   - Resource hints: preload, prefetch, preconnect
   - https://web.dev/articles/preconnect-and-dns-prefetch

4. **"HTTP/3 explained"** — Daniel Stenberg (author of curl)
   - https://http3-explained.haxx.se/
   - Understand QUIC and why it matters

---

## 🧠 Deep Dive: The Network Stack

### DNS Resolution — From Domain to IP

```
You type: https://app.example.com/dashboard

Step 1: Browser DNS Cache (milliseconds)
  → "Have I resolved app.example.com recently?" → if yes, use cached IP

Step 2: OS DNS Cache (milliseconds)
  → Check /etc/hosts file, then OS resolver cache

Step 3: Router DNS Cache
  → Your home/office router may cache DNS

Step 4: ISP Recursive Resolver
  → ISP's DNS server checks its cache
  → If not cached, starts recursive lookup:

Step 5: Recursive Resolution
  Root Server (.):           "Who handles .com?"  → .com TLD server
  .com TLD Server:           "Who handles example.com?" → authoritative NS
  example.com Authoritative: "app.example.com = 93.184.216.34" → answer!

Step 6: Response flows back through chain
  → Each server caches the result based on TTL (Time To Live)
  → Typical TTL: 60 seconds to 24 hours
```

**DNS Performance Optimization:**
```html
<!-- dns-prefetch: Resolve DNS for domains you'll need soon -->
<link rel="dns-prefetch" href="https://api.example.com">
<link rel="dns-prefetch" href="https://cdn.example.com">
<link rel="dns-prefetch" href="https://fonts.googleapis.com">

<!-- preconnect: DNS + TCP + TLS handshake ahead of time -->
<!-- More aggressive than dns-prefetch — use for critical origins -->
<link rel="preconnect" href="https://api.example.com">
<link rel="preconnect" href="https://cdn.example.com" crossorigin>
```

**DNS Record Types (know these):**
```
A Record:     domain → IPv4 address (93.184.216.34)
AAAA Record:  domain → IPv6 address
CNAME Record: domain → another domain (alias)
MX Record:    domain → mail server
TXT Record:   domain → text (used for verification, SPF, DKIM)
NS Record:    domain → authoritative name server
```

---

### TCP — Reliable Data Transport

```
TCP 3-Way Handshake (connection establishment):

Client                              Server
  │                                    │
  │──── SYN (seq=100) ──────────────→ │  "I want to connect"
  │                                    │
  │←─── SYN-ACK (seq=300, ack=101) ──│  "OK, I accept"
  │                                    │
  │──── ACK (ack=301) ──────────────→ │  "Great, let's go"
  │                                    │
  │←──→ DATA TRANSFER ←──→           │
  │                                    │

Cost: 1.5 round trips before ANY data flows
At 50ms RTT → 75ms just for handshake
```

**TCP Concepts You Must Know:**

```
1. CONGESTION CONTROL (Slow Start)
   - TCP doesn't send all data at once
   - Starts with small window (14KB typically)
   - Doubles window each round trip (exponential growth)
   - Until packet loss → then reduces window

   Impact: First page load of 100KB takes multiple round trips
   because TCP starts slow!

   This is why small initial payloads matter:
   - Keep critical CSS/JS under 14KB compressed
   - This fits in first TCP window → renders faster

2. HEAD-OF-LINE BLOCKING (HTTP/1.1 problem)
   - TCP guarantees ordered delivery
   - If packet 3 of 5 is lost, packets 4 and 5 wait for retransmission
   - Even though they arrived fine!
   - HTTP/2 has this problem at TCP level
   - HTTP/3 (QUIC over UDP) solves this

3. KEEP-ALIVE
   - HTTP/1.1 default: reuse TCP connection for multiple requests
   - Avoids repeated handshakes
   - Connection closes after idle timeout
```

---

### TLS — Encryption

```
TLS 1.3 Handshake (simplified):

Client                              Server
  │                                    │
  │──── ClientHello ────────────────→ │
  │     (supported ciphers,           │
  │      key share,                   │
  │      SNI: hostname)               │
  │                                    │
  │←─── ServerHello ────────────────  │
  │     (chosen cipher,               │
  │      key share,                   │
  │      certificate,                 │
  │      encrypted data starts!)      │
  │                                    │
  │──── Finished ───────────────────→ │
  │                                    │

TLS 1.3: 1 round trip (down from 2 in TLS 1.2)
0-RTT resumption: If previously connected, can send data immediately!
```

**What you must know:**
- TLS adds 1 round trip (50-100ms) to every new connection
- `preconnect` does TCP + TLS ahead of time
- Certificate chain validation happens client-side
- SNI (Server Name Indication): How servers host multiple HTTPS sites on one IP
- HSTS (HTTP Strict Transport Security): Forces HTTPS, prevents downgrade attacks

---

### HTTP/1.1 vs HTTP/2 vs HTTP/3

```
┌─────────────────────────────────────────────────────────────────────┐
│                        HTTP/1.1 (1997)                              │
│                                                                     │
│  - One request at a time per connection                            │
│  - 6 parallel connections per domain (browser limit)               │
│  - Head-of-line blocking at HTTP level                             │
│  - Headers sent as plain text (repeated for every request!)        │
│  - No server push                                                  │
│                                                                     │
│  Workarounds people used:                                          │
│  - Domain sharding (cdn1., cdn2. for more connections)             │
│  - Image sprites (combine images)                                  │
│  - CSS/JS concatenation (fewer requests)                           │
│  - Inlining small resources                                        │
│                                                                     │
│  These workarounds are ANTI-PATTERNS in HTTP/2!                    │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                        HTTP/2 (2015)                                │
│                                                                     │
│  - MULTIPLEXING: Many requests over ONE connection simultaneously  │
│  - Stream prioritization                                           │
│  - HEADER COMPRESSION (HPACK): Headers sent once, then referenced  │
│  - SERVER PUSH: Server can send resources before client asks       │
│  - Binary framing (not text)                                       │
│                                                                     │
│  Still has TCP head-of-line blocking:                              │
│  - One lost TCP packet blocks ALL streams                          │
│  - Even streams whose data wasn't affected                        │
│                                                                     │
│  Impact on frontend architecture:                                  │
│  - STOP concatenating (multiplexing makes it unnecessary)          │
│  - STOP domain sharding (one connection is now sufficient)         │
│  - DO use many small files (better caching granularity)            │
│  - DO use server push for critical CSS/JS                          │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                        HTTP/3 (2022)                                │
│                                                                     │
│  - Uses QUIC (UDP-based) instead of TCP                            │
│  - NO head-of-line blocking (stream independence)                  │
│  - Built-in encryption (TLS 1.3 integrated into QUIC)             │
│  - 0-RTT connection resumption                                     │
│  - Connection migration (survives network changes - e.g., WiFi→4G)│
│  - Better for mobile (unreliable networks)                         │
│                                                                     │
│  Tradeoff:                                                         │
│  - Higher CPU usage (UDP doesn't have OS-level optimization)       │
│  - Some firewalls/middleboxes block UDP                            │
│  - Most browsers fall back to HTTP/2 if QUIC fails                │
└─────────────────────────────────────────────────────────────────────┘
```

**Protocol Comparison:**
```
| Feature              | HTTP/1.1    | HTTP/2       | HTTP/3        |
|----------------------|-------------|--------------|---------------|
| Transport            | TCP         | TCP          | QUIC (UDP)    |
| Multiplexing         | ❌ (1 req)  | ✅ (streams)  | ✅ (streams)   |
| Header compression   | ❌          | ✅ (HPACK)    | ✅ (QPACK)     |
| Server push          | ❌          | ✅            | ✅             |
| HOL blocking         | HTTP + TCP  | TCP only     | ❌ (none!)     |
| Connection setup     | TCP + TLS   | TCP + TLS    | 0-1 RTT       |
| Encryption           | Optional    | Practically  | Always (built-in)|
| Binary               | ❌ (text)   | ✅            | ✅             |
```

---

### HTTP Caching — The Most Important Networking Topic

```
Caching saves bandwidth, reduces latency, and lowers server costs.
A Staff Engineer MUST understand the caching model completely.

Two Types of Caching:
1. STRONG CACHING: Browser doesn't even ask the server (fastest)
2. NEGOTIATED CACHING: Browser asks server "has this changed?" (still saves bandwidth)
```

**Strong Cache (Cache-Control):**
```
Server Response:
Cache-Control: max-age=31536000   (cache for 1 year)

What happens:
1. Browser receives response with Cache-Control header
2. Stores response in disk/memory cache
3. Next request for same URL within max-age → uses cached version
4. NO network request at all! Instant.
5. Shows "(disk cache)" or "(memory cache)" in DevTools Network tab

Common Cache-Control directives:
  max-age=N        → Cache for N seconds
  no-cache         → Must revalidate with server every time (NOT "don't cache"!)
  no-store         → Don't cache at all (truly no caching)
  public           → Can be cached by CDN/proxy (shared cache)
  private          → Only browser can cache (not CDN)
  immutable        → Content will NEVER change (skip revalidation)
  stale-while-revalidate=N → Serve stale cache while fetching fresh in background
```

**Negotiated Cache (ETag / Last-Modified):**
```
First Request:
  Server Response:
    ETag: "abc123"
    Last-Modified: Wed, 21 Oct 2025 07:28:00 GMT
    Cache-Control: no-cache   (must revalidate)

Second Request:
  Browser sends:
    If-None-Match: "abc123"
    If-Modified-Since: Wed, 21 Oct 2025 07:28:00 GMT

  Server checks: Has the resource changed?

  If NOT changed:
    304 Not Modified (no body!)
    → Browser uses cached version
    → Saves bandwidth (no content transferred)

  If changed:
    200 OK (with new body)
    → Browser updates cache
```

**Real-World Caching Strategy for Frontend:**
```
STATIC ASSETS (JS, CSS, images, fonts):
  File: /assets/app.a1b2c3.js  (content hash in filename)
  Cache-Control: public, max-age=31536000, immutable
  Why: Hash changes when content changes → new URL → cache miss
       Old URL is cached forever → no wasted revalidation

HTML FILES:
  File: /index.html
  Cache-Control: no-cache  (or max-age=0, must-revalidate)
  Why: HTML must always be fresh to reference latest asset URLs
       But still allows 304 responses (saves bandwidth)

API RESPONSES:
  Depends on data freshness requirements:
  Cache-Control: private, max-age=60  (user-specific, cache 1 min)
  Cache-Control: public, max-age=300, stale-while-revalidate=600
    → Serve from cache for 5 min, then refresh in background

CDN CACHING:
  CDN-Cache-Control: public, max-age=86400  (separate header for CDN)
  Surrogate-Control: max-age=86400          (Fastly/Varnish specific)
  → CDN and browser can have different cache durations
```

**Cache Invalidation (the hard problem):**
```
Problem: You deployed new code but CDN still serves old cached version.

Solutions:
1. Content-hashed filenames (app.a1b2c3.js → app.d4e5f6.js)
   → New deploy = new filename = cache miss = fresh content
   → This is what Vite/Webpack do automatically

2. Cache purge API (CDN-specific)
   → Tell CDN to delete specific cached resources
   → Cloudflare, Fastly, CloudFront all have purge APIs

3. Short max-age + stale-while-revalidate
   → max-age=60, stale-while-revalidate=3600
   → Fresh for 1 min, stale OK for 1 hour while refetching

4. Cache busting with query strings (?v=2)
   → AVOID: Some CDNs ignore query strings
   → Content hash in filename is better
```

---

### `fetch` API — Deep Understanding

```javascript
// Basic fetch
const response = await fetch('/api/users');
const data = await response.json();

// What you must understand:
// 1. fetch does NOT reject on HTTP errors (404, 500)!
if (!response.ok) {
  throw new Error(`HTTP ${response.status}: ${response.statusText}`);
}

// 2. Request configuration
const response = await fetch('/api/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123',
  },
  body: JSON.stringify({ name: 'Alice' }),
  credentials: 'include',     // send cookies cross-origin
  mode: 'cors',               // CORS mode
  cache: 'no-cache',          // cache behavior
  signal: controller.signal,  // AbortController for cancellation
});

// 3. AbortController — cancel requests
const controller = new AbortController();

// Cancel after 5 seconds
setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch('/api/slow', { signal: controller.signal });
} catch (err) {
  if (err.name === 'AbortError') {
    console.log('Request was cancelled');
  }
}

// 4. Streaming response (critical for AI features!)
const response = await fetch('/api/ai/chat', {
  method: 'POST',
  body: JSON.stringify({ prompt: 'Hello' }),
});

const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  const chunk = decoder.decode(value, { stream: true });
  appendToUI(chunk); // Show each chunk as it arrives
}

// 5. Interceptor pattern (like axios interceptors, but with fetch)
function createFetchWithAuth(baseURL, getToken) {
  return async function fetchWithAuth(path, options = {}) {
    const url = `${baseURL}${path}`;
    const token = await getToken();

    const response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json',
      },
    });

    if (response.status === 401) {
      // Token expired — refresh and retry
      const newToken = await refreshToken();
      return fetch(url, {
        ...options,
        headers: {
          ...options.headers,
          'Authorization': `Bearer ${newToken}`,
        },
      });
    }

    if (!response.ok) {
      throw new FetchError(response.status, await response.text());
    }

    return response;
  };
}
```

---

### REST vs GraphQL vs tRPC

```
┌─────────────────────────────────────────────────────────────────────┐
│ REST                                                                │
│                                                                     │
│ GET    /api/users           → List users                           │
│ GET    /api/users/123       → Get user 123                         │
│ POST   /api/users           → Create user                          │
│ PUT    /api/users/123       → Update user 123                      │
│ DELETE /api/users/123       → Delete user 123                      │
│                                                                     │
│ Pros: Simple, cacheable (GET), well-understood, HTTP semantics     │
│ Cons: Over-fetching (get all fields), under-fetching (need N+1),   │
│       multiple endpoints per resource, versioning challenges       │
│                                                                     │
│ Best for: Public APIs, simple CRUD, when caching matters           │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ GraphQL                                                             │
│                                                                     │
│ POST /graphql                                                      │
│ {                                                                   │
│   query {                                                          │
│     user(id: 123) {                                                │
│       name                   ← Only request fields you need        │
│       posts {                ← Nested data in ONE request          │
│         title                                                      │
│       }                                                            │
│     }                                                              │
│   }                                                                │
│ }                                                                   │
│                                                                     │
│ Pros: No over/under-fetching, one endpoint, strong typing,        │
│       self-documenting (introspection), great for complex UIs      │
│ Cons: Caching is harder (all POST), complexity, N+1 on server,    │
│       security (query depth attacks), learning curve               │
│                                                                     │
│ Best for: Complex UIs, mobile (bandwidth sensitive), multiple      │
│           frontend consumers, rapidly changing requirements        │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ tRPC                                                                │
│                                                                     │
│ // Server                                                          │
│ const appRouter = router({                                         │
│   getUser: procedure                                               │
│     .input(z.object({ id: z.number() }))                          │
│     .query(({ input }) => db.user.findById(input.id)),            │
│ });                                                                │
│                                                                     │
│ // Client — FULLY typed, no codegen!                               │
│ const user = await trpc.getUser.query({ id: 123 });               │
│ // TypeScript knows the exact shape of `user`                      │
│                                                                     │
│ Pros: End-to-end type safety (no codegen), simple, fast DX        │
│ Cons: TypeScript only, tightly couples client+server, same repo,  │
│       not suitable for public APIs                                 │
│                                                                     │
│ Best for: Full-stack TypeScript apps, same team owns FE+BE,       │
│           monorepo setups, internal tools                          │
└─────────────────────────────────────────────────────────────────────┘

Tradeoff Summary:
| Criteria            | REST     | GraphQL  | tRPC      |
|---------------------|----------|----------|-----------|
| Type safety         | Manual   | Codegen  | Automatic |
| Caching             | Easy     | Hard     | Easy      |
| Over-fetching       | Yes      | No       | No        |
| Learning curve      | Low      | High     | Low       |
| Public API          | ✅       | ✅       | ❌        |
| Multiple consumers  | ✅       | ✅       | ❌        |
| Bundle size impact  | None     | Client   | None      |
| Best team structure | Any      | Any      | Full-stack|
```

---

### WebSocket vs SSE vs Long Polling

```
┌─────────────────────────────────────────────────────────────────┐
│ Long Polling                                                     │
│                                                                  │
│ Client: GET /api/updates (hangs until server has data)          │
│ Server: ...waits... responds when data available                │
│ Client: Immediately sends another request                       │
│                                                                  │
│ Pros: Works everywhere, simple                                  │
│ Cons: HTTP overhead per "message", not truly real-time          │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ Server-Sent Events (SSE)                                        │
│                                                                  │
│ Client: new EventSource('/api/stream')                          │
│ Server: Sends events as text/event-stream                      │
│                                                                  │
│ event: update                                                   │
│ data: {"price": 150.25}                                        │
│                                                                  │
│ event: update                                                   │
│ data: {"price": 150.30}                                        │
│                                                                  │
│ Pros: Built-in reconnection, simple API, works with HTTP/2     │
│ Cons: Server → Client only (one direction), text only,         │
│       limited connections per domain in HTTP/1.1 (6)            │
│                                                                  │
│ USE FOR: Notifications, live feeds, AI streaming responses      │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ WebSocket                                                        │
│                                                                  │
│ Client: new WebSocket('wss://example.com/ws')                   │
│ Bi-directional, full-duplex communication                       │
│                                                                  │
│ Pros: True bi-directional, low latency, binary support,        │
│       one connection for all messages                           │
│ Cons: Different protocol (not HTTP), load balancing harder,     │
│       no built-in reconnection, stateful (harder to scale)     │
│                                                                  │
│ USE FOR: Chat, gaming, collaborative editing, live cursors      │
└─────────────────────────────────────────────────────────────────┘

Decision Framework:
- Need server→client only? → SSE (simpler)
- Need bi-directional? → WebSocket
- Need maximum compatibility? → Long Polling (fallback)
- Building AI chat streaming? → SSE or fetch streaming
- Building real-time collaboration? → WebSocket
```

---

## 🔨 Build Project (4 hours)

### Project: HTTP Explorer + Caching Lab

```
networking-lab/
├── caching-demo/
│   ├── server.js          (Node.js server with different caching headers)
│   └── index.html
├── streaming-demo/
│   ├── server.js          (SSE + WebSocket + fetch streaming)
│   └── index.html
└── fetch-wrapper/
    ├── fetchClient.js     (production-grade fetch wrapper)
    └── test.html
```

#### Part A: Caching Lab
```javascript
// Node.js server that serves same file with different caching strategies
// User clicks buttons to request each variant, sees the difference in DevTools

// Route 1: No cache headers
// Route 2: Cache-Control: max-age=30
// Route 3: Cache-Control: no-cache + ETag
// Route 4: Cache-Control: public, max-age=31536000, immutable
// Route 5: Cache-Control: max-age=10, stale-while-revalidate=30

// Display: Network waterfall showing cache hits/misses
// Exercise: Open DevTools Network tab, observe:
//   - 200 (from disk cache) — strong cache
//   - 304 Not Modified — negotiated cache
//   - 200 (real fetch) — no cache
```

#### Part B: Real-Time Communication Comparison
```javascript
// Build 3 versions of a "live stock ticker":
// 1. SSE version: EventSource → server pushes price updates
// 2. WebSocket version: ws connection → bi-directional
// 3. Fetch streaming version: ReadableStream → process chunks

// Display all 3 side by side
// Compare: connection overhead, message size, reconnection behavior
```

#### Part C: Production Fetch Client
```javascript
// Build a fetch wrapper with:
// 1. Base URL configuration
// 2. Automatic auth header injection
// 3. Request/response interceptors
// 4. Automatic retry with exponential backoff
// 5. Request deduplication (same URL in-flight → share response)
// 6. Timeout with AbortController
// 7. Request cancellation
// 8. Type-safe response parsing

class FetchClient {
  constructor(config) {
    this.baseURL = config.baseURL;
    this.timeout = config.timeout || 10000;
    this.interceptors = { request: [], response: [] };
    this.inflightRequests = new Map(); // deduplication
  }

  async request(path, options = {}) {
    // 1. Run request interceptors
    // 2. Check deduplication map
    // 3. Create AbortController with timeout
    // 4. Make fetch call
    // 5. Handle errors, retry if needed
    // 6. Run response interceptors
    // 7. Return typed response
  }

  // Retry with exponential backoff
  async retryWithBackoff(fn, retries = 3) {
    for (let i = 0; i < retries; i++) {
      try {
        return await fn();
      } catch (err) {
        if (i === retries - 1) throw err;
        const delay = Math.min(1000 * Math.pow(2, i), 10000);
        await new Promise(r => setTimeout(r, delay));
      }
    }
  }
}
```

---

## 🧪 DevTools Network Tab Exercises (1 hour)

### Exercise 1: Analyze a Page Load
1. Open DevTools → Network tab
2. Clear cache, load any major website
3. Observe:
   - Waterfall: What loads first? What blocks what?
   - Size: Transferred vs actual size (compression)
   - Time: DNS, TCP, TLS, TTFB, Content Download for each request
   - Headers: Cache-Control, ETag, Content-Encoding
   - Protocol: h2 (HTTP/2) or h3 (HTTP/3)

### Exercise 2: Cache Behavior
1. Load a page (first visit — all resources downloaded)
2. Reload → observe "(disk cache)" and "(memory cache)" entries
3. Hard reload (Cmd+Shift+R) → all resources fetched again
4. Check: Which resources use strong cache? Which use 304?

### Exercise 3: Throttling
1. Network tab → Throttle to "Slow 3G"
2. Load a page → feel the pain of slow network
3. Observe: Which resources are the bottleneck?
4. This is what your users on bad networks experience

### Exercise 4: WebSocket Inspection
1. Open a site that uses WebSocket (many chat apps do)
2. Network tab → Filter by "WS"
3. Click the WebSocket connection → Messages tab
4. See messages flowing in real time (both directions)

---

## ❓ Interview Questions You Must Answer (2 hours)

### Basic:
1. **What happens when you type a URL and press Enter? (Full networking answer)**
   - DNS → TCP handshake → TLS handshake → HTTP request → Response → Parsing
   - Include timing estimates for each step

2. **What is the difference between HTTP/1.1, HTTP/2, and HTTP/3?**
   - Multiplexing, header compression, HOL blocking, QUIC

3. **Explain HTTP caching. What is the difference between `Cache-Control: no-cache` and `no-store`?**
   - no-cache: Cache the resource but revalidate every time (may get 304)
   - no-store: Don't cache at all (never store on disk)

4. **What is an ETag? How does it work?**
   - Server generates hash of content → sends as ETag header
   - Client sends If-None-Match with ETag on next request
   - Server compares → 304 if same, 200 if different

5. **What is the difference between `preload`, `prefetch`, and `preconnect`?**
   - preload: "I NEED this resource for current page, fetch NOW" (high priority)
   - prefetch: "I MIGHT need this for next page, fetch when idle" (low priority)
   - preconnect: "I'll need resources from this origin, start TCP+TLS now"

### Intermediate:
6. **What is CORS? Why does it exist? How does it work?**
   - (Covered in detail in Part 7: Security, but know the basics)
   - Browser security: prevents scripts from making requests to different origins
   - Simple vs Preflight requests (OPTIONS)
   - Access-Control-Allow-Origin header

7. **Explain the TCP slow start algorithm. How does it affect web performance?**
   - Initial window ~14KB, doubles each RTT
   - First page load is limited by congestion window, not bandwidth
   - Impact: Keep critical resources small

8. **What is HTTP/2 Server Push? Is it still recommended?**
   - Server sends resources before client requests them
   - Mostly deprecated/removed — hard to get right, wastes bandwidth
   - Modern alternative: 103 Early Hints

9. **What is the difference between WebSocket and SSE?**
   - WebSocket: Bi-directional, binary, custom protocol
   - SSE: Server→Client only, text, built on HTTP, auto-reconnect
   - When to use which

10. **How would you design the caching strategy for a large SPA?**
    - HTML: no-cache (always fresh)
    - JS/CSS: immutable with content hash (cached forever)
    - API: depends on data freshness needs
    - Images: long max-age + CDN
    - Fonts: immutable (they never change)

### Advanced:
11. **What is HTTP/3's QUIC protocol? How does it solve head-of-line blocking?**
    - UDP-based, streams are independent
    - Lost packet in stream A doesn't block stream B
    - Built-in TLS 1.3, 0-RTT resumption

12. **Explain stale-while-revalidate. When would you use it?**
    - Serve stale cache immediately, fetch fresh in background
    - User gets instant response, next request gets fresh data
    - Perfect for: API data that's OK to be slightly stale

13. **How would you implement request deduplication in a frontend app?**
    - Multiple components request same data simultaneously
    - Share the in-flight promise instead of making duplicate requests
    - TanStack Query / SWR do this automatically

14. **What is a CDN? How does it work at the network level?**
    - Edge servers geographically close to users
    - Anycast routing → DNS resolves to nearest edge
    - Cache at edge → reduce origin server load
    - Invalidation strategies: TTL, purge API, surrogate keys

15. **Design an efficient data fetching strategy for a dashboard with 20 API calls.**
    - Parallel where possible, sequential where dependent
    - Priority: above-the-fold first
    - Stale-while-revalidate for background refresh
    - Deduplication for shared data
    - Streaming for large datasets
    - WebSocket for real-time updates

---

## ✅ Week 4 Completion Checklist

- [ ] Read HPBN chapters (TCP, TLS, HTTP/1.1, HTTP/2, HTTP/3)
- [ ] Can explain full network request lifecycle with timing estimates
- [ ] Understand HTTP caching model (strong cache vs negotiated cache)
- [ ] Can design caching strategy for SPA (HTML, JS/CSS, API, images)
- [ ] Understand HTTP/1.1 → HTTP/2 → HTTP/3 evolution and tradeoffs
- [ ] Understand REST vs GraphQL vs tRPC tradeoffs
- [ ] Understand WebSocket vs SSE vs Long Polling and when to use each
- [ ] Built caching lab and observed behavior in DevTools
- [ ] Built real-time communication demo (SSE + WebSocket + fetch streaming)
- [ ] Built production fetch client with retry, dedup, interceptors
- [ ] Can answer all 15 interview questions out loud

---

Next → `05-dom-and-css.md`
