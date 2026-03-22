# Web Performance & Optimization — A Senior Architect's Masterclass Roadmap

---

## Phase 0: The Mental Model — Why Performance Matters

Before any technique, internalize this:

- **Performance IS a feature.** Google proved that a 100ms delay in search results costs 0.2% revenue. Amazon showed every 100ms of latency costs 1% of sales.
- **Performance is NOT just "make it faster."** It's about **perceived performance**, **time to interactive**, **resource efficiency**, and **scalability under load**.
- The three pillars:
  1. **Network** — how fast bytes travel
  2. **Parse & Render** — how fast the browser turns bytes into pixels
  3. **Runtime** — how fast your code responds after the page is alive

---

## Phase 1: How the Browser Actually Works (Non-Negotiable Foundation)

You **cannot** optimize what you don't understand. Most engineers skip this and end up cargo-culting.

### 1.1 — The Critical Rendering Path (CRP)

This is the single most important concept in web performance.

```
Navigation → DNS → TCP → TLS → HTTP Request → Response
    → HTML Parsing → DOM Construction
    → CSS Parsing → CSSOM Construction
    → DOM + CSSOM → Render Tree
    → Layout (Reflow)
    → Paint
    → Composite
```

**What to study deeply:**

- How the HTML parser works (tokenizer → tree construction)
- What **blocks rendering** (CSS is render-blocking, JS is parser-blocking)
- The difference between **DOM**, **CSSOM**, **Render Tree**, **Layout Tree**, and **Layer Tree**
- What triggers **reflow** vs **repaint** vs **composite-only** changes
- How the **compositor thread** works independently from the main thread

**Key resource:** "How Browsers Work" by Tali Garsiel (the canonical deep-dive)

### 1.2 — The Network Layer

```
DNS Lookup → TCP Handshake → TLS Negotiation → HTTP Request/Response
```

**Understand these cold:**

- DNS resolution chain (recursive resolver → root → TLD → authoritative)
- TCP 3-way handshake and why it costs 1 RTT
- TLS 1.3 handshake (1-RTT vs 0-RTT resumption)
- HTTP/1.1 limitations (head-of-line blocking, 6 connections per origin)
- HTTP/2 multiplexing, header compression (HPACK), server push
- HTTP/3 (QUIC) — UDP-based, 0-RTT, no TCP head-of-line blocking
- Connection: keep-alive, connection pooling, connection coalescing

### 1.3 — The Rendering Pipeline (Deep)

```
JavaScript → Style → Layout → Paint → Composite
```

| Stage | What happens | Cost |
|-------|-------------|------|
| **JavaScript** | Event handlers, DOM manipulation, frameworks | Main thread blocked |
| **Style** | Selector matching, computed styles | O(n × m) worst case |
| **Layout** | Geometry calculation (position, size) | Can cascade (reflow) |
| **Paint** | Fill pixels (colors, images, text, shadows) | Per-layer |
| **Composite** | GPU compositing of layers | Cheap, off main thread |

**The goal:** Push as much work as possible to **composite-only** changes.

- `transform` and `opacity` are composite-only (GPU-accelerated)
- `width`, `height`, `top`, `left` trigger layout → paint → composite (expensive)
- `color`, `background` trigger paint → composite (medium)

---

## Phase 2: Core Web Vitals & Metrics That Matter

### 2.1 — The Metrics

| Metric | What it measures | Target |
|--------|-----------------|--------|
| **FCP** (First Contentful Paint) | First pixel of content | < 1.8s |
| **LCP** (Largest Contentful Paint) | Largest visible element rendered | < 2.5s |
| **INP** (Interaction to Next Paint) | Responsiveness to user input | < 200ms |
| **CLS** (Cumulative Layout Shift) | Visual stability | < 0.1 |
| **TTFB** (Time to First Byte) | Server responsiveness | < 800ms |
| **TBT** (Total Blocking Time) | Main thread blocked time (lab metric) | < 200ms |

### 2.2 — Lab vs Field Data

- **Lab** (Lighthouse, WebPageTest): Synthetic, controlled, reproducible
- **Field** (CrUX, RUM): Real user data, 75th percentile, what Google ranks on
- **Never optimize only for lab.** A site can score 100 in Lighthouse and be slow for real users on 3G in India.

### 2.3 — Tools You Must Master

| Tool | Use case |
|------|----------|
| **Chrome DevTools → Performance tab** | Frame-by-frame main thread analysis |
| **Chrome DevTools → Network tab** | Waterfall, timing breakdown, caching |
| **Chrome DevTools → Lighthouse** | Automated audit |
| **WebPageTest** | Multi-location, multi-device, filmstrip, waterfall |
| **CrUX Dashboard** | Real-user field data |
| **Performance Observer API** | Custom RUM in production |
| **`requestAnimationFrame` profiling** | Runtime jank detection |

---

## Phase 3: Network Optimization

### 3.1 — Reduce Bytes

| Technique | Details |
|-----------|---------|
| **Compression** | Brotli (11-15% better than gzip). Ensure server sends `Content-Encoding: br` |
| **Minification** | HTML, CSS, JS — remove whitespace, shorten variables |
| **Tree Shaking** | Dead code elimination (ES modules only, not CommonJS) |
| **Image Optimization** | WebP/AVIF, responsive `srcset`, lazy loading, proper sizing |
| **Font Optimization** | `font-display: swap`, subset fonts, variable fonts, preload critical fonts |

### 3.2 — Reduce Requests

| Technique | Details |
|-----------|---------|
| **Bundling** | Combine modules (but don't over-bundle — code splitting matters) |
| **CSS/JS Inlining** | Critical CSS inline, defer the rest |
| **Sprites / SVG symbols** | For icon sets (less relevant with HTTP/2) |
| **Data URIs** | For tiny assets < 2KB |

### 3.3 — Reduce Latency

| Technique | Details |
|-----------|---------|
| **CDN** | Edge servers close to users |
| **DNS Prefetch** | `<link rel="dns-prefetch" href="//cdn.example.com">` |
| **Preconnect** | `<link rel="preconnect" href="//api.example.com">` (DNS + TCP + TLS) |
| **Preload** | `<link rel="preload" href="critical.css" as="style">` |
| **Prefetch** | `<link rel="prefetch" href="next-page.js">` (low-priority future navigation) |
| **Early Hints (103)** | Server sends hints before full response |
| **HTTP/2 Server Push** | Push critical resources with HTML (being deprecated in favor of Early Hints) |

### 3.4 — Caching Strategy

```
Cache-Control: public, max-age=31536000, immutable    → Hashed static assets
Cache-Control: no-cache                                → HTML pages (always revalidate)
Cache-Control: private, max-age=0, must-revalidate     → User-specific content
ETag + If-None-Match                                   → Conditional requests (304)
```

**The pattern that works at scale:**

- HTML: `no-cache` (always check for new version)
- JS/CSS/Images with content hash in filename: `max-age=1 year, immutable`
- API responses: `private, max-age=0` or `stale-while-revalidate`

**Also study:**

- Service Worker caching strategies (Cache First, Network First, Stale-While-Revalidate)
- CDN cache invalidation patterns
- `Vary` header implications

---

## Phase 4: Rendering Optimization

### 4.1 — Critical CSS

Extract and inline the CSS needed for above-the-fold content. Defer the rest.

```html
<style>/* inlined critical CSS */</style>
<link rel="preload" href="full.css" as="style" onload="this.rel='stylesheet'">
```

### 4.2 — JavaScript Loading Strategies

| Attribute | Behavior |
|-----------|----------|
| `<script>` | Blocks HTML parsing |
| `<script defer>` | Downloads in parallel, executes after HTML parsing, in order |
| `<script async>` | Downloads in parallel, executes immediately when ready, no order guarantee |
| `<script type="module">` | Deferred by default |
| Dynamic `import()` | Code splitting, loaded on demand |

**Rule:** Use `defer` for everything unless you have a specific reason not to.

### 4.3 — Layout Thrashing

This is the #1 runtime performance killer most engineers don't know about:

```javascript
// BAD — forces synchronous layout on every iteration
for (let i = 0; i < elements.length; i++) {
  elements[i].style.width = container.offsetWidth + 'px'; // read then write
}

// GOOD — batch reads, then batch writes
const width = container.offsetWidth; // single read
for (let i = 0; i < elements.length; i++) {
  elements[i].style.width = width + 'px'; // batch writes
}
```

**Properties that trigger forced synchronous layout when read:**

`offsetTop`, `offsetWidth`, `scrollTop`, `clientHeight`, `getComputedStyle()`, `getBoundingClientRect()`

### 4.4 — Virtual DOM & Reconciliation (React-specific)

- Understand React's fiber architecture and concurrent rendering
- `React.memo`, `useMemo`, `useCallback` — when they help and when they hurt
- Key prop strategy — why random keys destroy performance
- Virtualized lists for large datasets (`react-window`, `@tanstack/virtual`)

---

## Phase 5: JavaScript Runtime Performance

### 5.1 — Main Thread Budget

At 60fps, you have **16.67ms per frame**. The browser needs ~6ms for its own work. You get **~10ms** for JavaScript.

### 5.2 — Long Tasks & Yielding

Any task > 50ms is a "Long Task" and degrades INP.

```javascript
// Break long work into chunks
function processInChunks(items, chunkSize = 50) {
  let index = 0;

  function nextChunk() {
    const end = Math.min(index + chunkSize, items.length);
    while (index < end) {
      processItem(items[index++]);
    }
    if (index < items.length) {
      scheduler.postTask(nextChunk, { priority: 'background' });
      // or: setTimeout(nextChunk, 0) as fallback
    }
  }

  nextChunk();
}
```

### 5.3 — Web Workers

Move heavy computation off the main thread entirely:

- **Web Workers** — parallel JS execution, no DOM access
- **SharedArrayBuffer + Atomics** — shared memory between threads
- **Comlink** — makes Web Worker communication feel like local function calls

### 5.4 — Memory Management

- Understand V8's garbage collector (Scavenger for young gen, Mark-Compact for old gen)
- Avoid memory leaks: detached DOM nodes, forgotten event listeners, closures holding references, uncleared timers
- Use DevTools → Memory tab → Heap Snapshots to find leaks
- Use `WeakMap`, `WeakSet`, `WeakRef` for cache patterns

---

## Phase 6: Image & Media Optimization

Images are typically **50-70% of total page weight**.

| Format | Use case | Savings |
|--------|----------|---------|
| **AVIF** | Best compression, photos & illustrations | 50% smaller than JPEG |
| **WebP** | Good compression, wide support | 25-35% smaller than JPEG |
| **JPEG** | Fallback for photos | Baseline |
| **PNG** | Transparency needed | Use WebP/AVIF with alpha instead |
| **SVG** | Icons, logos, illustrations | Infinitely scalable |

**Responsive images pattern:**

```html
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img
    src="image.jpg"
    srcset="image-400.jpg 400w, image-800.jpg 800w, image-1200.jpg 1200w"
    sizes="(max-width: 600px) 100vw, 50vw"
    loading="lazy"
    decoding="async"
    width="800"
    height="600"
    alt="Description"
  >
</picture>
```

**Critical:** Always set `width` and `height` (or `aspect-ratio`) to prevent CLS.

---

## Phase 7: Architecture-Level Performance Patterns

### 7.1 — Rendering Strategies

| Strategy | When to use |
|----------|-------------|
| **SSG** (Static Site Generation) | Content that rarely changes |
| **SSR** (Server-Side Rendering) | Dynamic content, SEO-critical, fast FCP |
| **CSR** (Client-Side Rendering) | App-like experiences, behind auth |
| **ISR** (Incremental Static Regeneration) | Static + periodic updates |
| **Streaming SSR** | Large pages, progressive rendering |
| **Partial Hydration / Islands** | Mix static + interactive (Astro pattern) |
| **RSC** (React Server Components) | Zero JS for server-rendered components |

### 7.2 — Code Splitting Strategy

```
Entry Point Splitting → Route-based splitting → Component-based splitting
```

- Split by route (each page is a chunk)
- Lazy load below-the-fold components
- Split vendor code separately (changes less often → better caching)
- Analyze with `webpack-bundle-analyzer` or `source-map-explorer`

### 7.3 — Edge Computing

- **Edge Functions** (Cloudflare Workers, Vercel Edge) — run logic at CDN edge
- **Edge SSR** — server-render at the edge, near the user
- **KV Storage at Edge** — cache data at edge for sub-ms reads

---

## Phase 8: Monitoring & Continuous Performance

### 8.1 — Real User Monitoring (RUM)

```javascript
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    sendToAnalytics({
      metric: entry.name,
      value: entry.startTime,
      id: entry.id,
    });
  }
});

observer.observe({ type: 'largest-contentful-paint', buffered: true });
observer.observe({ type: 'first-input', buffered: true });
observer.observe({ type: 'layout-shift', buffered: true });
```

### 8.2 — Performance Budgets

Set and enforce:

- **Bundle size budget:** main JS < 170KB gzipped
- **Image budget:** total < 500KB per page
- **LCP budget:** < 2.5s at 75th percentile
- **INP budget:** < 200ms at 75th percentile

Enforce in CI with Lighthouse CI, bundlesize, or size-limit.

### 8.3 — Regression Detection

- Run Lighthouse in CI on every PR
- Compare bundle sizes against main branch
- Set up alerts on CrUX/RUM data for regressions
- WebPageTest API for automated before/after comparisons

---

## Learning Path & Order

| Week | Focus | Deliverable |
|------|-------|-------------|
| 1-2 | Browser internals (CRP, rendering pipeline) | Diagram the full rendering pipeline from memory |
| 3-4 | Network layer (HTTP/2, caching, CDN) | Configure optimal caching for a real project |
| 5-6 | Core Web Vitals & tooling | Audit 3 real sites, create improvement reports |
| 7-8 | Image optimization & responsive media | Implement responsive image pipeline |
| 9-10 | JavaScript runtime performance | Profile and fix jank in a real app |
| 11-12 | Architecture patterns (SSR, code splitting, edge) | Implement SSR with streaming for a project |
| 13-14 | Monitoring & budgets | Set up RUM + performance budgets in CI |
| 15-16 | Deep dive: pick your weakest area and go deeper | Write a technical blog post teaching what you learned |

---

## Key Resources (Curated, Not a Dump)

1. **web.dev/performance** — Google's canonical performance guides
2. **"High Performance Browser Networking"** by Ilya Grigorik (free online) — the networking bible
3. **WebPageTest documentation** — learn waterfall analysis
4. **Chrome DevTools documentation** — Performance tab deep-dives
5. **"How Browsers Work"** by Tali Garsiel — rendering engine internals
6. **web.dev/vitals** — Core Web Vitals guides with real case studies
