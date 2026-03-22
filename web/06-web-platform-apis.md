# Part 6: Web Platform APIs 🧩

## Week 6 | 10 hours | Advanced browser capabilities beyond the basics

---

## Why This Matters

Modern browsers are application platforms — not just document viewers.
Service Workers, Web Workers, Streams, and other APIs enable:
- Offline-capable apps (PWAs)
- CPU-intensive computation without freezing the UI
- Real-time streaming (AI responses, video, file processing)
- Background sync, push notifications, file system access

**A Staff Engineer knows which platform API to reach for and when NOT to use one.**

---

## 📖 Reading Plan (3 hours)

### Must Read:
1. **MDN: Web Workers API**
   - https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API
   - Dedicated Workers, Shared Workers, concepts

2. **MDN: Service Worker API**
   - https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
   - Lifecycle, caching strategies, offline support

3. **"The Offline Cookbook"** — Jake Archibald
   - https://jakearchibald.com/2014/offline-cookbook/
   - Every caching strategy explained with diagrams

4. **MDN: Streams API**
   - https://developer.mozilla.org/en-US/docs/Web/API/Streams_API
   - ReadableStream, WritableStream, TransformStream

5. **web.dev: "Workers overview"**
   - https://web.dev/articles/workers-overview

---

## 🧠 Deep Dive: Web Workers

### The Problem: JavaScript is Single-Threaded

```
Main Thread handles EVERYTHING:
- JavaScript execution
- DOM updates
- Style calculation
- Layout
- Paint
- Event handling
- Network callbacks

If your JS runs for 200ms → UI freezes for 200ms
User sees: jank, unresponsive buttons, dropped frames

Solution: Move expensive computation OFF the main thread → Web Workers
```

### Dedicated Web Worker

```javascript
// ═══════════════════════════════════════════
// main.js (runs on main thread)
// ═══════════════════════════════════════════

// Create a worker (loads script in separate thread)
const worker = new Worker('worker.js');

// Send data TO worker
worker.postMessage({
  type: 'PROCESS_DATA',
  payload: largeDataset // data is COPIED (structured clone), not shared
});

// Receive results FROM worker
worker.onmessage = (event) => {
  const { type, result } = event.data;
  if (type === 'PROCESS_COMPLETE') {
    updateUI(result); // Back on main thread — safe to touch DOM
  }
};

// Handle errors
worker.onerror = (error) => {
  console.error('Worker error:', error.message);
};

// Terminate worker when done
worker.terminate();


// ═══════════════════════════════════════════
// worker.js (runs on separate thread)
// ═══════════════════════════════════════════

// No access to: DOM, window, document, localStorage
// Has access to: fetch, IndexedDB, WebSocket, crypto, timers

self.onmessage = (event) => {
  const { type, payload } = event.data;

  if (type === 'PROCESS_DATA') {
    // Heavy computation — doesn't block main thread!
    const result = processLargeDataset(payload);

    // Send result back to main thread
    self.postMessage({ type: 'PROCESS_COMPLETE', result });
  }
};

function processLargeDataset(data) {
  // Sort, filter, aggregate millions of records
  // This would freeze the UI if done on main thread
  return data
    .filter(item => item.active)
    .map(item => transform(item))
    .sort((a, b) => a.score - b.score);
}
```

### Transferable Objects (Zero-Copy)

```javascript
// Problem: postMessage COPIES data (structured clone)
// For large ArrayBuffers, copying is expensive

// Solution: Transfer ownership (zero-copy, instant)
const buffer = new ArrayBuffer(1024 * 1024 * 100); // 100MB

// SLOW: Copy the buffer (takes time + doubles memory)
worker.postMessage({ data: buffer });

// FAST: Transfer the buffer (instant, zero-copy)
worker.postMessage({ data: buffer }, [buffer]);
// WARNING: `buffer` is now EMPTY in main thread (ownership transferred)
// buffer.byteLength === 0 after transfer

// SharedArrayBuffer: Truly shared memory (both threads access same memory)
// Requires: Cross-Origin-Isolation headers (COOP + COEP)
const sharedBuffer = new SharedArrayBuffer(1024);
const view = new Int32Array(sharedBuffer);

// Use Atomics for thread-safe operations:
Atomics.store(view, 0, 42);       // Thread-safe write
Atomics.load(view, 0);            // Thread-safe read
Atomics.add(view, 0, 1);          // Thread-safe increment
Atomics.wait(view, 0, 42);        // Wait until value changes
Atomics.notify(view, 0);          // Wake waiting threads
```

### Comlink — Simplify Worker Communication

```javascript
// Comlink makes workers feel like regular async function calls
// Instead of postMessage/onmessage boilerplate:

// worker.js
import { expose } from 'comlink';

const api = {
  async processData(data) {
    // heavy computation
    return result;
  },

  async calculateStats(numbers) {
    return {
      mean: numbers.reduce((a, b) => a + b, 0) / numbers.length,
      // ...more stats
    };
  }
};

expose(api);

// main.js
import { wrap } from 'comlink';

const worker = new Worker('worker.js');
const api = wrap(worker);

// Call worker functions like normal async functions!
const result = await api.processData(largeDataset);
const stats = await api.calculateStats([1, 2, 3, 4, 5]);
// No postMessage/onmessage boilerplate!
```

### When to Use Web Workers

```
✅ USE WORKERS FOR:
- Sorting/filtering large datasets (>10,000 items)
- Data transformation/aggregation
- Image processing (canvas operations)
- Parsing large JSON/CSV files
- Encryption/hashing (crypto operations)
- Search indexing (client-side search)
- Complex math/simulation
- Markdown/syntax highlighting of large documents

❌ DON'T USE WORKERS FOR:
- DOM manipulation (workers can't access DOM)
- Simple calculations (overhead of creating worker > computation time)
- Anything that needs synchronous result
- Small data processing (<1ms of work)

⚠️ WORKER LIMITATIONS:
- No DOM access (no document, no window)
- No localStorage/sessionStorage (use IndexedDB instead)
- Communication overhead (data copying via structured clone)
- Each worker = separate thread = memory overhead (~1-5MB per worker)
- Max workers: browser-dependent (usually 20-50)
```

---

## 🧠 Deep Dive: Service Workers

### What is a Service Worker?

```
A Service Worker is a PROXY between your app and the network.
It intercepts every network request and decides what to do:
- Serve from cache? (instant, works offline)
- Fetch from network? (always fresh)
- Both? (serve cache, update in background)

┌─────────┐     ┌──────────────────┐     ┌────────┐
│  Your   │ ──→ │ Service Worker   │ ──→ │Network │
│  App    │ ←── │ (intercepts all  │ ←── │        │
│         │     │  requests)       │     │        │
└─────────┘     │                  │     └────────┘
                │  ┌────────────┐  │
                │  │   Cache    │  │
                │  │   Storage  │  │
                │  └────────────┘  │
                └──────────────────┘
```

### Service Worker Lifecycle

```
┌──────────────┐
│  INSTALLING  │ ← register() called, SW downloads
│              │   install event fires → pre-cache critical assets
└──────┬───────┘
       ↓
┌──────────────┐
│   WAITING    │ ← Installed but old SW still controls pages
│              │   Waits until all tabs with old SW close
│              │   (or skipWaiting() called)
└──────┬───────┘
       ↓
┌──────────────┐
│   ACTIVE     │ ← Controls all pages in scope
│              │   activate event fires → clean up old caches
│              │   fetch event fires → intercept requests
└──────┬───────┘
       ↓
┌──────────────┐
│  REDUNDANT   │ ← Replaced by newer SW or registration failed
└──────────────┘
```

### Registration & Caching

```javascript
// ═══════════════════════════════════════════
// main.js — Register Service Worker
// ═══════════════════════════════════════════

if ('serviceWorker' in navigator) {
  window.addEventListener('load', async () => {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js', {
        scope: '/' // SW controls all pages under this path
      });
      console.log('SW registered:', registration.scope);
    } catch (error) {
      console.error('SW registration failed:', error);
    }
  });
}


// ═══════════════════════════════════════════
// sw.js — Service Worker
// ═══════════════════════════════════════════

const CACHE_NAME = 'app-v1';
const PRECACHE_URLS = [
  '/',
  '/index.html',
  '/styles.css',
  '/app.js',
  '/offline.html'  // fallback page for when offline
];

// INSTALL: Pre-cache critical assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(PRECACHE_URLS))
      .then(() => self.skipWaiting()) // Activate immediately (don't wait)
  );
});

// ACTIVATE: Clean up old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys()
      .then(cacheNames => {
        return Promise.all(
          cacheNames
            .filter(name => name !== CACHE_NAME)
            .map(name => caches.delete(name))
        );
      })
      .then(() => self.clients.claim()) // Take control of all pages immediately
  );
});

// FETCH: Intercept all network requests
self.addEventListener('fetch', (event) => {
  event.respondWith(
    handleRequest(event.request)
  );
});
```

### Caching Strategies (know ALL of these)

```javascript
// ═══════════════════════════════════════════
// Strategy 1: CACHE FIRST (Cache Falling Back to Network)
// Best for: Static assets (CSS, JS, images, fonts)
// ═══════════════════════════════════════════
async function cacheFirst(request) {
  const cached = await caches.match(request);
  if (cached) return cached; // ← Instant! No network needed

  const response = await fetch(request);
  const cache = await caches.open(CACHE_NAME);
  cache.put(request, response.clone()); // Cache for next time
  return response;
}

// ═══════════════════════════════════════════
// Strategy 2: NETWORK FIRST (Network Falling Back to Cache)
// Best for: API requests, frequently updated content
// ═══════════════════════════════════════════
async function networkFirst(request) {
  try {
    const response = await fetch(request);
    const cache = await caches.open(CACHE_NAME);
    cache.put(request, response.clone()); // Update cache with fresh data
    return response;
  } catch (error) {
    const cached = await caches.match(request);
    if (cached) return cached; // ← Offline fallback
    return new Response('Offline', { status: 503 });
  }
}

// ═══════════════════════════════════════════
// Strategy 3: STALE WHILE REVALIDATE
// Best for: Content that should be fast but also fresh
// (avatars, social feeds, non-critical API data)
// ═══════════════════════════════════════════
async function staleWhileRevalidate(request) {
  const cache = await caches.open(CACHE_NAME);
  const cached = await cache.match(request);

  // Fetch fresh version in background (don't await!)
  const fetchPromise = fetch(request).then(response => {
    cache.put(request, response.clone());
    return response;
  });

  // Return cached immediately (or wait for network if no cache)
  return cached || fetchPromise;
}

// ═══════════════════════════════════════════
// Strategy 4: CACHE ONLY
// Best for: Pre-cached assets that never change
// ═══════════════════════════════════════════
async function cacheOnly(request) {
  return caches.match(request);
}

// ═══════════════════════════════════════════
// Strategy 5: NETWORK ONLY
// Best for: Non-GET requests, analytics, real-time data
// ═══════════════════════════════════════════
async function networkOnly(request) {
  return fetch(request);
}

// ═══════════════════════════════════════════
// ROUTING: Apply different strategies to different requests
// ═══════════════════════════════════════════
async function handleRequest(request) {
  const url = new URL(request.url);

  // Static assets → cache first
  if (url.pathname.match(/\.(js|css|png|jpg|svg|woff2)$/)) {
    return cacheFirst(request);
  }

  // API calls → network first (with offline fallback)
  if (url.pathname.startsWith('/api/')) {
    return networkFirst(request);
  }

  // HTML pages → stale while revalidate
  if (request.headers.get('accept')?.includes('text/html')) {
    return staleWhileRevalidate(request);
  }

  // Everything else → network first
  return networkFirst(request);
}
```

### Workbox — Production Service Worker Library

```javascript
// Workbox (by Google) simplifies Service Worker development
// Instead of writing all the above manually:

import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { CacheFirst, NetworkFirst, StaleWhileRevalidate } from 'workbox-strategies';
import { ExpirationPlugin } from 'workbox-expiration';

// Pre-cache app shell (auto-generated by build tool)
precacheAndRoute(self.__WB_MANIFEST);

// Cache images with cache-first strategy
registerRoute(
  ({ request }) => request.destination === 'image',
  new CacheFirst({
    cacheName: 'images',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 100,
        maxAgeSeconds: 30 * 24 * 60 * 60, // 30 days
      }),
    ],
  })
);

// Cache API calls with network-first strategy
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkFirst({
    cacheName: 'api-cache',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 50,
        maxAgeSeconds: 5 * 60, // 5 minutes
      }),
    ],
  })
);

// Cache pages with stale-while-revalidate
registerRoute(
  ({ request }) => request.mode === 'navigate',
  new StaleWhileRevalidate({
    cacheName: 'pages',
  })
);
```

---

## 🧠 Deep Dive: Streams API

### Why Streams Matter

```
Traditional approach:
  fetch('/api/large-file') → wait for ENTIRE response → process all at once
  Memory: Need to hold entire response in memory
  Time: User waits until everything is downloaded

Streaming approach:
  fetch('/api/large-file') → process each chunk as it arrives
  Memory: Only hold current chunk in memory
  Time: User sees data immediately as it arrives

Critical for:
- AI chat responses (tokens stream in one by one)
- Large file uploads/downloads
- Video/audio processing
- Real-time data processing
```

### ReadableStream

```javascript
// fetch() returns a ReadableStream in response.body
const response = await fetch('/api/large-data');

// Read stream manually
const reader = response.body.getReader();
const decoder = new TextDecoder();

let result = '';
while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  const chunk = decoder.decode(value, { stream: true });
  result += chunk;
  updateProgressUI(result.length); // Show progress as data arrives
}

// Create your own ReadableStream
const stream = new ReadableStream({
  start(controller) {
    // Called once when stream is created
    controller.enqueue('Hello ');  // Push data into stream
    controller.enqueue('World');
    controller.close();            // Signal no more data
  },

  pull(controller) {
    // Called when consumer wants more data (backpressure)
  },

  cancel(reason) {
    // Called when consumer cancels the stream
  }
});

// Consume with for-await-of (async iterator)
for await (const chunk of stream) {
  console.log(chunk);
}
```

### TransformStream (pipe data through transformations)

```javascript
// TransformStream: takes input, transforms it, produces output
// Like Unix pipes: input | transform | output

// Example: Line-by-line processor for SSE
const lineDecoder = new TransformStream({
  transform(chunk, controller) {
    const text = new TextDecoder().decode(chunk);
    const lines = text.split('\n');
    for (const line of lines) {
      if (line.trim()) {
        controller.enqueue(line);
      }
    }
  }
});

// Example: JSON parser stream
const jsonParser = new TransformStream({
  transform(chunk, controller) {
    try {
      const parsed = JSON.parse(chunk);
      controller.enqueue(parsed);
    } catch (e) {
      // Incomplete JSON, buffer it
    }
  }
});

// Pipe streams together:
const response = await fetch('/api/stream');
const reader = response.body
  .pipeThrough(lineDecoder)
  .pipeThrough(jsonParser)
  .getReader();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  console.log('Parsed object:', value);
}
```

### AI Streaming Pattern (you'll use this A LOT)

```javascript
// Stream AI responses from OpenAI/Anthropic API
async function streamAIResponse(prompt, onChunk) {
  const response = await fetch('/api/ai/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ prompt }),
  });

  if (!response.ok) throw new Error(`HTTP ${response.status}`);

  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = '';

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += decoder.decode(value, { stream: true });

    // SSE format: "data: {...}\n\n"
    const lines = buffer.split('\n\n');
    buffer = lines.pop() || ''; // Keep incomplete chunk in buffer

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data === '[DONE]') return;

        try {
          const parsed = JSON.parse(data);
          const token = parsed.choices?.[0]?.delta?.content || '';
          if (token) onChunk(token); // Display each token as it arrives
        } catch (e) {
          // Skip malformed chunks
        }
      }
    }
  }
}

// Usage:
let fullResponse = '';
await streamAIResponse('Explain closures', (token) => {
  fullResponse += token;
  document.getElementById('output').textContent = fullResponse;
  // User sees text appearing word by word — great UX!
});
```

---

## 🧠 Other Important Web APIs

### Broadcast Channel (tab-to-tab communication)

```javascript
// Communicate between tabs/windows of the same origin
const channel = new BroadcastChannel('app-events');

// Tab 1: Send message
channel.postMessage({ type: 'LOGOUT', timestamp: Date.now() });

// Tab 2: Receive message
channel.onmessage = (event) => {
  if (event.data.type === 'LOGOUT') {
    // User logged out in another tab → log out here too
    window.location.href = '/login';
  }
};

// Use case: Sync auth state across tabs (logout everywhere)
// Use case: Sync theme changes across tabs
// Use case: Notify other tabs of data changes
```

### AbortController (cancel anything)

```javascript
// AbortController can cancel: fetch, event listeners, streams, anything

// 1. Cancel fetch requests
const controller = new AbortController();
const response = fetch('/api/data', { signal: controller.signal });
controller.abort(); // Cancels the fetch

// 2. Auto-cancel on timeout
const controller = AbortController.timeout(5000);
// Automatically aborts after 5 seconds

// 3. Remove event listeners
const controller = new AbortController();

window.addEventListener('resize', handleResize, { signal: controller.signal });
window.addEventListener('scroll', handleScroll, { signal: controller.signal });
document.addEventListener('click', handleClick, { signal: controller.signal });

// Remove ALL three listeners at once!
controller.abort();

// 4. React/Vue cleanup pattern
function useEffect() {
  const controller = new AbortController();

  fetch('/api/data', { signal: controller.signal });
  window.addEventListener('resize', handler, { signal: controller.signal });

  // Cleanup: abort cancels EVERYTHING
  return () => controller.abort();
}
```

### Clipboard API

```javascript
// Modern async clipboard API
async function copyToClipboard(text) {
  try {
    await navigator.clipboard.writeText(text);
    showToast('Copied!');
  } catch (err) {
    // Fallback for older browsers
    const textarea = document.createElement('textarea');
    textarea.value = text;
    document.body.appendChild(textarea);
    textarea.select();
    document.execCommand('copy');
    document.body.removeChild(textarea);
  }
}

// Read from clipboard
async function pasteFromClipboard() {
  const text = await navigator.clipboard.readText();
  return text;
}

// Copy rich content (images, HTML)
async function copyImage(blob) {
  await navigator.clipboard.write([
    new ClipboardItem({ 'image/png': blob })
  ]);
}
```

### requestIdleCallback

```javascript
// Run low-priority work when browser is idle
// Prevents blocking critical rendering/interaction

function processBackgroundTasks(tasks) {
  function workLoop(deadline) {
    // deadline.timeRemaining() tells you how much idle time is left
    while (deadline.timeRemaining() > 1 && tasks.length > 0) {
      const task = tasks.shift();
      task(); // Run one task
    }

    if (tasks.length > 0) {
      // More tasks? Schedule for next idle period
      requestIdleCallback(workLoop);
    }
  }

  requestIdleCallback(workLoop);
}

// Use cases:
// - Analytics event batching
// - Pre-fetching data for likely next page
// - Non-critical DOM updates
// - Logging
// - Pre-computing search indexes
```

---

## 🔨 Build Project (4 hours)

### Project: Offline-Capable AI Note-Taking App

```
offline-notes-app/
├── index.html
├── app.js           (main application logic)
├── sw.js            (service worker — offline support)
├── worker.js        (web worker — search indexing)
├── stream-utils.js  (streaming helpers)
└── style.css
```

**Features to build:**

#### Feature 1: Service Worker for Offline Support
```
- Pre-cache app shell (HTML, CSS, JS)
- Cache API responses with stale-while-revalidate
- Show offline fallback page when no network
- Background sync: queue saves when offline, sync when online
```

#### Feature 2: Web Worker for Search
```
- User types in search box
- Send search query to Web Worker
- Worker searches through all notes (could be thousands)
- Returns results without blocking UI
- Implement: fuzzy search with scoring
```

#### Feature 3: Streaming AI Summary
```
- User clicks "Summarize" on a long note
- Fetch streaming response from AI API (mock it with a local server)
- Display tokens as they stream in
- Allow cancellation mid-stream (AbortController)
- Show loading state, error handling
```

#### Feature 4: Cross-Tab Sync
```
- Use BroadcastChannel to sync note changes across tabs
- Edit in one tab → other tab updates instantly
- Handle conflict resolution (last-write-wins or merge)
```

---

## ❓ Interview Questions You Must Answer (2 hours)

### Basic:
1. **What is a Web Worker? When would you use one?**
2. **What is a Service Worker? How is it different from a Web Worker?**
   - Web Worker: Parallel computation, no DOM, communicates with main thread
   - Service Worker: Network proxy, intercepts requests, enables offline, push notifications
3. **What are the main Service Worker caching strategies?**
   - Cache First, Network First, Stale While Revalidate, Cache Only, Network Only
4. **What is the Streams API? Why does it matter for modern web apps?**
5. **How does `AbortController` work? What can you cancel with it?**

### Intermediate:
6. **How would you implement offline support for a web application?**
7. **Explain the Service Worker lifecycle (install, waiting, activate).**
8. **What is the difference between `postMessage` copy and transferable objects?**
9. **How would you implement streaming AI responses in a frontend app?**
10. **What is `BroadcastChannel`? When would you use it vs `SharedWorker`?**
    - BroadcastChannel: Simple pub/sub between tabs, same origin
    - SharedWorker: Single worker shared by all tabs, runs code, maintains state

### Advanced:
11. **How would you architect a large-scale offline-first application?**
    - Service Worker for caching, IndexedDB for data, background sync for writes
    - Conflict resolution strategy (CRDT, last-write-wins, merge)
12. **What is `SharedArrayBuffer`? What security headers does it require?**
    - Shared memory between threads, requires COOP + COEP headers
    - Use Atomics for thread-safe operations
13. **Explain backpressure in streams. How does it work?**
    - Consumer reads slower than producer writes
    - ReadableStream signals backpressure → producer slows down
    - Prevents memory overflow
14. **How would you use Web Workers to build client-side full-text search?**
15. **Design a file upload system with progress, pause/resume, and chunked uploads.**
    - Streams API for reading file chunks
    - Web Worker for compression
    - AbortController for pause/cancel
    - IndexedDB for tracking upload progress

---

## ✅ Week 6 Completion Checklist

- [ ] Can create and communicate with Web Workers
- [ ] Understand Transferable objects and SharedArrayBuffer
- [ ] Can implement Service Worker with all 5 caching strategies
- [ ] Understand Service Worker lifecycle
- [ ] Can use Streams API for reading, transforming, and piping data
- [ ] Can implement AI streaming response pattern
- [ ] Know when to use: BroadcastChannel, AbortController, requestIdleCallback
- [ ] Built offline-capable app with Service Worker
- [ ] Built Web Worker search
- [ ] Built streaming AI response UI
- [ ] Can answer all 15 interview questions out loud

---

Next → `07-security-and-storage.md`
