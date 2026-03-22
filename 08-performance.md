# Part 8: Web Performance — Tying Everything Together 🚀

## Week 8 | 10 hours | The capstone — use ALL your knowledge to optimize

---

## Why This Matters

Performance is where EVERYTHING from the previous 7 weeks comes together:
- Browser rendering (Week 1) → optimize the pixel pipeline
- JS engine (Week 2) → write V8-friendly code
- Core JS (Week 3) → avoid blocking the main thread
- Networking (Week 4) → reduce requests, optimize caching
- DOM + CSS (Week 5) → minimize reflows, use containment
- Web APIs (Week 6) → offload to workers, use Service Workers
- Security (Week 7) → CSP, SRI don't add significant overhead

**At FAANG, performance is not optional. It's a core engineering requirement.**
Google uses Core Web Vitals as a ranking factor.
Amazon found that every 100ms of latency costs 1% in revenue.

---

## 📖 Reading Plan (3 hours)

### Must Read:
1. **web.dev: Core Web Vitals**
   - https://web.dev/articles/vitals
   - LCP, INP, CLS — understand each metric deeply

2. **web.dev: "Optimize Largest Contentful Paint"**
   - https://web.dev/articles/optimize-lcp

3. **web.dev: "Optimize Interaction to Next Paint"**
   - https://web.dev/articles/optimize-inp

4. **web.dev: "Optimize Cumulative Layout Shift"**
   - https://web.dev/articles/optimize-cls

5. **"The Cost of JavaScript in 2024"** — Addy Osmani
   - Search for the latest version of this article
   - Covers: parse, compile, execute costs of JS

6. **Chrome DevTools: Performance panel documentation**
   - https://developer.chrome.com/docs/devtools/performance/

---

## 🧠 Deep Dive: Core Web Vitals

### The Three Metrics That Matter

```
┌─────────────────────────────────────────────────────────────────────┐
│ LCP — Largest Contentful Paint                                      │
│                                                                     │
│ WHAT: Time until the largest visible content element renders        │
│ MEASURES: Loading performance                                       │
│ TARGET: ≤ 2.5 seconds                                              │
│                                                                     │
│ What counts as LCP element:                                        │
│ - <img> elements                                                   │
│ - <image> inside <svg>                                             │
│ - <video> poster image                                             │
│ - Element with background-image (CSS)                              │
│ - Block-level text elements (<h1>, <p>, etc.)                      │
│                                                                     │
│ Common LCP problems:                                               │
│ 1. Slow server response (high TTFB)                                │
│ 2. Render-blocking CSS/JS                                          │
│ 3. Large unoptimized images                                        │
│ 4. Client-side rendering (content visible only after JS executes)  │
│ 5. Lazy loading the LCP image (don't lazy load above-the-fold!)   │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ INP — Interaction to Next Paint                                     │
│                                                                     │
│ WHAT: Latency of user interactions (click, tap, keyboard)          │
│ MEASURES: Responsiveness                                            │
│ TARGET: ≤ 200 milliseconds                                         │
│                                                                     │
│ How INP is calculated:                                             │
│ - Measures delay from interaction to next visual update            │
│ - INP = the worst interaction responsiveness (or near-worst)       │
│ - Includes: Input delay + Processing time + Presentation delay     │
│                                                                     │
│ ┌────────────┬──────────────────┬───────────────────┐              │
│ │Input Delay │ Processing Time  │ Presentation Delay│              │
│ │(main thread│ (event handler   │ (render the       │              │
│ │ was busy)  │  execution)      │  visual update)   │              │
│ └────────────┴──────────────────┴───────────────────┘              │
│ ←────────────── INP ───────────────────────────→                   │
│                                                                     │
│ Common INP problems:                                               │
│ 1. Long tasks blocking main thread (>50ms)                         │
│ 2. Heavy event handlers                                            │
│ 3. Large DOM size (>1500 nodes = slow updates)                     │
│ 4. Excessive re-renders (React/Vue)                                │
│ 5. Synchronous layout reads in handlers (layout thrashing)         │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ CLS — Cumulative Layout Shift                                       │
│                                                                     │
│ WHAT: Measures unexpected layout shifts during page lifetime        │
│ MEASURES: Visual stability                                          │
│ TARGET: ≤ 0.1                                                      │
│                                                                     │
│ CLS score = Impact fraction × Distance fraction                    │
│   Impact fraction: % of viewport affected by the shift             │
│   Distance fraction: How far the element moved (as % of viewport)  │
│                                                                     │
│ Common CLS problems:                                               │
│ 1. Images without dimensions (load and push content down)          │
│ 2. Ads/embeds without reserved space                               │
│ 3. Dynamically injected content above existing content             │
│ 4. Web fonts causing FOUT/FOIT (text size changes)                 │
│ 5. Animations using top/left instead of transform                  │
│                                                                     │
│ Fixes:                                                             │
│ 1. Always set width/height on <img> and <video>                    │
│ 2. Use aspect-ratio CSS for responsive images                      │
│ 3. Reserve space for dynamic content (min-height)                  │
│ 4. Use font-display: optional/swap + preload fonts                 │
│ 5. Use transform for animations (no layout shift)                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧠 Deep Dive: Optimization Techniques

### 1. Resource Loading Optimization

```html
<!-- CRITICAL RESOURCES: Load these ASAP -->

<!-- Preconnect to important origins (DNS + TCP + TLS) -->
<link rel="preconnect" href="https://api.example.com">
<link rel="preconnect" href="https://fonts.googleapis.com">

<!-- Preload critical resources (high priority, current page) -->
<link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="/hero-image.webp" as="image">
<link rel="preload" href="/critical.css" as="style">

<!-- Prefetch resources for NEXT page (low priority, idle time) -->
<link rel="prefetch" href="/next-page.js">
<link rel="prefetch" href="/next-page-data.json">

<!-- fetchpriority: Control resource loading priority -->
<img src="hero.jpg" fetchpriority="high">  <!-- LCP image: load first! -->
<img src="thumbnail.jpg" fetchpriority="low">  <!-- Below fold: load later -->
<script src="critical.js" fetchpriority="high"></script>
<script src="analytics.js" fetchpriority="low"></script>

<!-- DEFER non-critical CSS -->
<link rel="stylesheet" href="non-critical.css" media="print" onload="this.media='all'">
<!-- Loads as print (non-blocking), switches to all when loaded -->
```

### 2. Image Optimization (often the biggest win)

```html
<!-- MODERN IMAGE FORMATS: WebP (30% smaller), AVIF (50% smaller) -->
<picture>
  <source srcset="hero.avif" type="image/avif">
  <source srcset="hero.webp" type="image/webp">
  <img src="hero.jpg" alt="Hero image"
       width="1200" height="600"
       loading="eager"
       fetchpriority="high"
       decoding="async">
</picture>

<!-- RESPONSIVE IMAGES: Different sizes for different screens -->
<img src="hero-800.jpg"
     srcset="hero-400.jpg 400w,
             hero-800.jpg 800w,
             hero-1200.jpg 1200w,
             hero-1600.jpg 1600w"
     sizes="(max-width: 600px) 100vw,
            (max-width: 1200px) 50vw,
            33vw"
     alt="Hero"
     width="1200" height="600">
<!-- Browser picks the best image size based on viewport + DPR -->

<!-- LAZY LOADING: For below-the-fold images -->
<img src="photo.jpg" loading="lazy" alt="Photo">
<!-- ⚠️ NEVER lazy load the LCP image! It needs loading="eager" (default) -->

<!-- IMAGE DIMENSIONS: Always set to prevent CLS -->
<img src="photo.jpg" width="800" height="600" alt="Photo">
<!-- Or use CSS aspect-ratio: -->
<style>
  .responsive-img {
    width: 100%;
    aspect-ratio: 4 / 3;  /* maintains ratio without knowing dimensions */
    object-fit: cover;
  }
</style>
```

### 3. JavaScript Optimization

```javascript
// ═══════════════════════════════════════════
// CODE SPLITTING: Only load what you need
// ═══════════════════════════════════════════

// Route-based splitting (most common):
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Settings = React.lazy(() => import('./pages/Settings'));
// Each page is a separate bundle, loaded only when navigated to

// Component-based splitting:
const HeavyChart = React.lazy(() => import('./components/HeavyChart'));
// Chart library only loaded when component is rendered

// Feature-based splitting:
const adminModule = await import('./admin'); // dynamic import
// Only loads admin code if user is an admin


// ═══════════════════════════════════════════
// TREE SHAKING: Remove unused code
// ═══════════════════════════════════════════

// BAD: Imports entire library (200KB)
import _ from 'lodash';
_.debounce(fn, 300);

// GOOD: Imports only what's used (5KB) — tree-shakeable
import { debounce } from 'lodash-es'; // ES modules version
debounce(fn, 300);

// BETTER: Write your own (0KB extra)
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}


// ═══════════════════════════════════════════
// LONG TASK BREAKING: Keep main thread responsive
// ═══════════════════════════════════════════

// BAD: One long task blocks the main thread
function processAllItems(items) {
  items.forEach(item => heavyComputation(item)); // 500ms task = frozen UI
}

// GOOD: Break into chunks, yield to browser between chunks
async function processAllItems(items) {
  const CHUNK_SIZE = 50;

  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const chunk = items.slice(i, i + CHUNK_SIZE);
    chunk.forEach(item => heavyComputation(item));

    // Yield to browser — allow rendering and event handling
    await new Promise(resolve => setTimeout(resolve, 0));
    // Or better: await scheduler.yield() (if available)
  }
}

// BEST: Move to Web Worker
const worker = new Worker('process-worker.js');
worker.postMessage(items);
worker.onmessage = (e) => updateUI(e.data);
```

### 4. Rendering Optimization

```javascript
// ═══════════════════════════════════════════
// VIRTUALIZATION: Render only visible items
// ═══════════════════════════════════════════

// Instead of rendering 10,000 DOM nodes:
// Render only ~20 visible items + buffer
// As user scrolls, recycle DOM nodes with new data

// Libraries: react-window, react-virtuoso, @tanstack/virtual
// Concept: Only items in viewport + overscan are in the DOM


// ═══════════════════════════════════════════
// DEBOUNCE + THROTTLE: Control expensive operations
// ═══════════════════════════════════════════

// Debounce: Wait until user STOPS, then execute once
// Use for: Search input, window resize, form validation
const debouncedSearch = debounce(searchAPI, 300);
input.addEventListener('input', debouncedSearch);

// Throttle: Execute at most once per interval
// Use for: Scroll handler, mouse move, resize observer
const throttledScroll = throttle(handleScroll, 100);
window.addEventListener('scroll', throttledScroll, { passive: true });


// ═══════════════════════════════════════════
// DOM SIZE: Keep it small
// ═══════════════════════════════════════════

// Google recommends:
// - < 1,500 total DOM nodes
// - < 32 nesting depth
// - No parent with > 60 children

// Large DOM → slow style calculations, slow layout, slow paint
// Virtualization is the main tool for large lists/tables
```

### 5. Font Optimization

```css
/* PROBLEM: Web fonts cause FOUT (Flash of Unstyled Text)
   or FOIT (Flash of Invisible Text) */

/* Strategy 1: font-display: swap (shows fallback, then swaps) */
@font-face {
  font-family: 'MyFont';
  src: url('myfont.woff2') format('woff2');
  font-display: swap; /* Show fallback text immediately, swap when loaded */
}

/* Strategy 2: font-display: optional (best for CLS) */
@font-face {
  font-family: 'MyFont';
  src: url('myfont.woff2') format('woff2');
  font-display: optional; /* Use font only if already cached, else use fallback */
  /* No layout shift! But might not use custom font on first visit */
}
```

```html
<!-- PRELOAD fonts (they're discovered late in the waterfall) -->
<link rel="preload" href="/fonts/myfont.woff2" as="font" type="font/woff2" crossorigin>

<!-- SUBSET fonts: Only include characters you need -->
<!-- Use tools like glyphhanger or Google Fonts &text= parameter -->
<!-- Full font: 200KB → Subset for Latin: 20KB -->
```

### 6. Bundle Size Management

```javascript
// ═══════════════════════════════════════════
// ANALYZE: Know what's in your bundle
// ═══════════════════════════════════════════

// Vite: npx vite-bundle-visualizer
// Webpack: npx webpack-bundle-analyzer
// Shows treemap of bundle contents — find the biggest dependencies

// ═══════════════════════════════════════════
// BUDGET: Set limits
// ═══════════════════════════════════════════

// In CI/CD pipeline, fail build if bundle exceeds budget:
// package.json (using bundlesize):
{
  "bundlesize": [
    { "path": "dist/js/*.js", "maxSize": "200 kB" },
    { "path": "dist/css/*.css", "maxSize": "50 kB" }
  ]
}

// Recommended budgets (compressed):
// - Initial JS bundle: < 150-200 KB
// - Per-route chunk: < 50 KB
// - Total CSS: < 50 KB
// - Total initial load: < 500 KB

// ═══════════════════════════════════════════
// REPLACE HEAVY DEPENDENCIES
// ═══════════════════════════════════════════

// moment.js (300KB) → date-fns (tree-shakeable) or dayjs (2KB)
// lodash (70KB) → lodash-es (tree-shake) or native JS
// axios (13KB) → fetch (built-in, 0KB)
// classnames (2KB) → clsx (228B)
// uuid (3KB) → crypto.randomUUID() (built-in, 0KB)
```

---

## 🧠 Deep Dive: Measuring Performance

### Performance API

```javascript
// ═══════════════════════════════════════════
// NAVIGATION TIMING: Full page load metrics
// ═══════════════════════════════════════════
const navigation = performance.getEntriesByType('navigation')[0];

const metrics = {
  // DNS lookup time
  dns: navigation.domainLookupEnd - navigation.domainLookupStart,

  // TCP connection time
  tcp: navigation.connectEnd - navigation.connectStart,

  // TLS handshake time
  tls: navigation.secureConnectionStart > 0
    ? navigation.connectEnd - navigation.secureConnectionStart : 0,

  // Time to First Byte
  ttfb: navigation.responseStart - navigation.requestStart,

  // Content download time
  download: navigation.responseEnd - navigation.responseStart,

  // DOM parsing time
  domParse: navigation.domInteractive - navigation.responseEnd,

  // DOM content loaded
  domContentLoaded: navigation.domContentLoadedEventEnd - navigation.startTime,

  // Full page load
  load: navigation.loadEventEnd - navigation.startTime,
};


// ═══════════════════════════════════════════
// CORE WEB VITALS: Measure in production
// ═══════════════════════════════════════════
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP((metric) => {
  console.log('LCP:', metric.value, 'ms');
  console.log('LCP Element:', metric.entries[0]?.element);
  sendToAnalytics({ name: 'LCP', value: metric.value });
});

onINP((metric) => {
  console.log('INP:', metric.value, 'ms');
  sendToAnalytics({ name: 'INP', value: metric.value });
});

onCLS((metric) => {
  console.log('CLS:', metric.value);
  sendToAnalytics({ name: 'CLS', value: metric.value });
});


// ═══════════════════════════════════════════
// CUSTOM PERFORMANCE MARKS: Measure your code
// ═══════════════════════════════════════════
performance.mark('data-fetch-start');
const data = await fetchData();
performance.mark('data-fetch-end');

performance.measure('Data Fetch', 'data-fetch-start', 'data-fetch-end');

const measure = performance.getEntriesByName('Data Fetch')[0];
console.log(`Data fetch took: ${measure.duration}ms`);


// ═══════════════════════════════════════════
// LONG TASK OBSERVER: Detect tasks > 50ms
// ═══════════════════════════════════════════
const longTaskObserver = new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.warn(`Long task detected: ${entry.duration}ms`);
    // In production: send to monitoring dashboard
  }
});

longTaskObserver.observe({ type: 'longtask', buffered: true });
```

### Chrome DevTools Performance Workflow

```
Step 1: MEASURE (don't optimize blindly)
  → DevTools → Performance tab → Record
  → Interact with the page
  → Stop recording
  → Analyze the flame chart

Step 2: IDENTIFY bottlenecks
  → Look for:
    - Red triangles: long tasks (>50ms)
    - Purple events: Layout (reflow)
    - Green events: Paint
    - Yellow events: Script execution
    - Gray: Idle time

Step 3: ANALYZE specific issues
  → Click on long tasks to see call stack
  → "Bottom-Up" tab: Which functions took most time
  → "Call Tree" tab: Top-down execution flow
  → "Event Log" tab: Chronological events

Step 4: LIGHTHOUSE audit
  → DevTools → Lighthouse tab
  → Run audit: Performance, Accessibility, Best Practices, SEO
  → Get specific recommendations with estimated savings

Step 5: FIX and RE-MEASURE
  → Make one change at a time
  → Re-measure after each change
  → Compare before/after
  → Don't assume — MEASURE
```

---

## 🔨 Build Project (4 hours)

### Project: Performance Audit & Optimization Lab

```
performance-lab/
├── unoptimized/
│   ├── index.html          (intentionally slow page)
│   ├── style.css
│   └── app.js
├── optimized/
│   ├── index.html          (same page, fully optimized)
│   ├── style.css
│   └── app.js
├── monitoring/
│   └── perf-monitor.js     (real-time performance dashboard)
└── report.md               (before/after comparison document)
```

#### Part A: Build an Intentionally Slow Page
```
Include these performance anti-patterns:
1. Large unoptimized images (no srcset, no lazy loading, no dimensions)
2. Render-blocking CSS in <head> (huge stylesheet)
3. Synchronous <script> tags in <head>
4. No font optimization (FOUT visible)
5. Layout thrashing in scroll handler
6. 10,000 DOM nodes (no virtualization)
7. No caching headers
8. Large unused JavaScript (import entire lodash)
9. Animations using top/left (not transform)
10. No code splitting

Measure with Lighthouse → document the scores
```

#### Part B: Optimize Everything
```
Apply ALL optimization techniques:
1. Images: WebP/AVIF, srcset, lazy loading, dimensions, fetchpriority
2. CSS: Critical CSS inline, defer non-critical, containment
3. JS: defer/async, code splitting, tree shaking, dead code elimination
4. Fonts: preload, font-display: optional, subset
5. Rendering: Virtualize the list, use transform for animations
6. DOM: Batch reads/writes, use requestAnimationFrame
7. Caching: Proper Cache-Control headers
8. Bundle: Remove unused dependencies, analyze with visualizer
9. Monitoring: Add web-vitals measurement
10. Service Worker: Pre-cache critical resources

Measure with Lighthouse → document the improved scores
```

#### Part C: Real-Time Performance Monitor
```javascript
// Build a performance dashboard overlay (like a game FPS counter):
// Display in real-time:
// - Current FPS (frames per second)
// - Memory usage (performance.memory)
// - DOM node count
// - Active event listeners count (estimate)
// - Long task indicator (flash red when long task detected)
// - CLS running score
// - Current LCP value
//
// Toggle with keyboard shortcut (Ctrl+Shift+P)
// Should be lightweight itself (< 1KB, no layout thrashing)
```

#### Part D: Write the Performance Report
```markdown
# Performance Optimization Report

## Before:
- Lighthouse Performance Score: ___
- LCP: ___ms
- INP: ___ms
- CLS: ___
- Total JS size: ___KB
- Total transfer size: ___KB
- DOM nodes: ___
- Time to Interactive: ___ms

## After:
- (same metrics)

## Changes Made:
1. [Change]: [Before metric] → [After metric] ([% improvement])
2. ...

## Key Learnings:
- What had the biggest impact?
- What was surprising?
- What tradeoffs were made?
```

---

## ❓ Interview Questions You Must Answer (2 hours)

### Basic:
1. **What are Core Web Vitals? Name and explain each metric.**
2. **How would you optimize LCP for a page with a hero image?**
   - Preload image, use proper format (WebP/AVIF), set fetchpriority="high"
   - Don't lazy load it, reduce server response time, inline critical CSS
3. **What causes layout shift (CLS)? How do you fix it?**
4. **What is code splitting? Why does it matter?**
5. **How does browser caching improve performance?**

### Intermediate:
6. **How would you reduce JavaScript bundle size?**
   - Tree shaking, code splitting, dynamic imports, replace heavy deps
   - Analyze with bundle visualizer, set budgets
7. **What is the critical rendering path? How do you optimize it?**
8. **Explain the difference between `loading="lazy"` and `loading="eager"` on images.**
9. **What is `fetchpriority`? When would you use it?**
10. **How would you measure performance in production?**
    - web-vitals library, PerformanceObserver, RUM (Real User Monitoring)
    - p75 metrics (not averages), segment by device/network

### Advanced:
11. **Design a performance monitoring system for a large web application.**
    - Client: web-vitals + custom marks + PerformanceObserver
    - Collection: beacon API for sending metrics
    - Backend: Time-series database, dashboards
    - Alerting: When p75 exceeds threshold
    - Budgets: CI/CD checks for bundle size, Lighthouse scores

12. **How would you optimize a React/Vue app that renders a 50,000-row table?**
    - Virtualization (only render visible rows)
    - Web Worker for sorting/filtering
    - Debounce search input
    - Memoize row components
    - Consider paginating instead

13. **What is INP? How is it different from FID? How do you optimize it?**
    - INP measures worst interaction, FID only measures first
    - Optimize: break long tasks, use workers, reduce DOM size
    - Use requestIdleCallback for non-urgent work
    - Yield to main thread in long processing

14. **Explain HTTP/2 Server Push, 103 Early Hints, and preload. Compare them.**
    - Server Push: Server sends resources proactively (mostly deprecated)
    - 103 Early Hints: Server sends preload hints before final response
    - Preload: Browser discovers resources early from HTML `<link>`
    - Early Hints is the modern replacement for Server Push

15. **You have a Lighthouse score of 40. Walk me through your optimization process.**
    - Step 1: Identify the biggest opportunities (Lighthouse tells you)
    - Step 2: Fix LCP first (usually biggest impact)
    - Step 3: Fix CLS (set dimensions, reserve space)
    - Step 4: Fix INP (break long tasks, reduce JS)
    - Step 5: Optimize assets (images, fonts, JS bundles)
    - Step 6: Implement caching strategy
    - Step 7: Measure after each change
    - Expected: Each fix should be measurable independently

---

## ✅ Week 8 Completion Checklist

- [ ] Understand all three Core Web Vitals (LCP, INP, CLS) deeply
- [ ] Know optimization techniques for each metric
- [ ] Can use Chrome DevTools Performance panel proficiently
- [ ] Can run and interpret Lighthouse audits
- [ ] Understand resource loading optimization (preload, prefetch, preconnect, fetchpriority)
- [ ] Can optimize images (formats, srcset, lazy loading, dimensions)
- [ ] Can optimize JavaScript (code splitting, tree shaking, long task breaking)
- [ ] Can optimize CSS (critical CSS, containment, font optimization)
- [ ] Can measure performance with web-vitals and PerformanceObserver
- [ ] Built the performance lab (unoptimized → optimized, with measured results)
- [ ] Wrote the performance report with before/after comparison
- [ ] Can answer all 15 interview questions out loud

---

## 🏁 Congratulations — Web Fundamentals Complete!

After 8 weeks, you should now:

| Skill | Status |
|-------|--------|
| Explain browser rendering pipeline in 5 minutes | ✅ |
| Debug V8 optimization issues | ✅ |
| Implement any JS utility from scratch (no libraries) | ✅ |
| Design caching strategies for complex apps | ✅ |
| Build UI components with raw DOM APIs | ✅ |
| Use Service Workers, Web Workers, Streams | ✅ |
| Identify and prevent XSS, CSRF, CORS issues | ✅ |
| Optimize a page from Lighthouse 40 → 90+ | ✅ |
| Answer 120 interview questions on web fundamentals | ✅ |

**Total: 80 hours invested. 8 projects built. 120 interview questions mastered.**

Now continue with the 12-month Staff Frontend Engineer plan →
`~/staff-frontend-engineer-plan.md`
