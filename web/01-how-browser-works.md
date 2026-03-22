# Part 1: How the Browser Works 🖥️

## Week 1 | 10 hours | Foundation of everything

---

## Why This Matters

Every frontend decision you make — component structure, CSS strategy, rendering approach,
performance optimization — is ultimately executed by the browser's rendering pipeline.

**Staff Engineers don't just use the browser. They understand it.**

At FAANG interviews, you'll be asked: "What happens when you type a URL and press Enter?"
The difference between Senior and Staff is the DEPTH of your answer.

---

## 🗺️ The Big Picture: URL to Pixels

```
User types URL
      ↓
[1] DNS Resolution — Domain → IP address
      ↓
[2] TCP Connection — 3-way handshake (SYN → SYN-ACK → ACK)
      ↓
[3] TLS Handshake — (if HTTPS) Negotiate encryption
      ↓
[4] HTTP Request/Response — Browser sends request, server responds with HTML
      ↓
[5] HTML Parsing — Bytes → Characters → Tokens → Nodes → DOM Tree
      ↓
[6] CSS Parsing — Bytes → Characters → Tokens → Nodes → CSSOM Tree
      ↓
[7] JavaScript Execution — Can block parsing! (unless async/defer)
      ↓
[8] Render Tree — DOM + CSSOM = Render Tree (only visible nodes)
      ↓
[9] Layout (Reflow) — Calculate exact position and size of every element
      ↓
[10] Paint — Fill in pixels: colors, borders, shadows, text
      ↓
[11] Composite — Layer composition, GPU-accelerated, final pixels on screen
      ↓
User sees the page
```

You must understand EVERY step above in detail. Let's go deep.

---

## 📖 Reading Plan (3 hours)

### Must Read:
1. **"How Browsers Work: Behind the Scenes of Modern Web Browsers"** — Tali Garsiel & Paul Irish
   - https://web.dev/articles/howbrowserswork
   - This is THE definitive article. Read it completely. Take notes.
   - Estimated time: 60-90 minutes

2. **"Inside look at modern web browser"** — Mariko Kosaka (Google, 4-part series)
   - Part 1: https://developer.chrome.com/blog/inside-browser-part1
   - Part 2: https://developer.chrome.com/blog/inside-browser-part2
   - Part 3: https://developer.chrome.com/blog/inside-browser-part3
   - Part 4: https://developer.chrome.com/blog/inside-browser-part4
   - Estimated time: 45-60 minutes

3. **"Rendering Performance"** — Paul Lewis
   - https://web.dev/articles/rendering-performance
   - Focus on: pixel pipeline, layout triggers, paint triggers
   - Estimated time: 30 minutes

### Reference (read as needed):
- MDN: "Critical rendering path" — https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path
- "What forces layout/reflow" — https://gist.github.com/paulirish/5d52fb081b3570c81e3a (Paul Irish's list)

---

## 🧠 Deep Dive: Each Stage Explained

### Stage 1: Navigation & DNS Resolution

```
Browser receives URL: https://www.example.com/page

Step 1: Check browser DNS cache
Step 2: Check OS DNS cache
Step 3: Check router DNS cache
Step 4: Query ISP's DNS resolver
Step 5: Recursive DNS lookup:
        Root DNS → .com TLD → example.com authoritative server
Step 6: Receive IP address (e.g., 93.184.216.34)
Step 7: Cache the result (TTL-based)
```

**What you must know:**
- DNS resolution can take 20-120ms — that's significant
- `dns-prefetch` and `preconnect` hints can start this early
- HTTP/2 and HTTP/3 reduce connection overhead (covered in Part 4)

---

### Stage 2-3: TCP + TLS Connection

```
TCP 3-way handshake:
  Client → SYN → Server
  Client ← SYN-ACK ← Server
  Client → ACK → Server
  (1.5 round trips = ~50-150ms)

TLS 1.3 handshake (HTTPS):
  Client → ClientHello (supported ciphers, key share) → Server
  Client ← ServerHello (chosen cipher, key share, certificate) ← Server
  Client → Finished → Server
  (1 round trip = ~30-100ms, down from 2 in TLS 1.2)
```

**What you must know:**
- Every new connection costs 1-3 round trips before ANY data flows
- `Connection: keep-alive` reuses TCP connections (default in HTTP/1.1)
- HTTP/2 multiplexes multiple requests over ONE connection
- HTTP/3 uses QUIC (UDP-based) — 0-RTT connection possible
- `preconnect` link hint: `<link rel="preconnect" href="https://api.example.com">`

---

### Stage 4: HTTP Request & Response

```
Request:
GET /page HTTP/1.1
Host: www.example.com
Accept: text/html
Accept-Encoding: gzip, br
Cookie: session=abc123

Response:
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Encoding: br
Cache-Control: max-age=3600
Transfer-Encoding: chunked

<!DOCTYPE html>
<html>...
```

**What you must know:**
- Response can be **chunked** — browser starts parsing BEFORE full download
- **Compression** (gzip, Brotli) reduces transfer size by 60-80%
- **Caching headers** determine if browser even makes a request (covered in Part 4)

---

### Stage 5: HTML Parsing → DOM Tree

```
Bytes: 3C 68 74 6D 6C 3E ...
  ↓ (decode based on Content-Type charset)
Characters: <html><head><title>...
  ↓ (tokenizer)
Tokens: [StartTag: html] [StartTag: head] [StartTag: title] [Character: "..."] ...
  ↓ (tree construction)
DOM Tree:
  Document
  └── html
      ├── head
      │   ├── title
      │   │   └── "Page Title"
      │   └── link (rel="stylesheet")
      └── body
          ├── h1
          │   └── "Hello World"
          └── p
              └── "Content here"
```

**Critical Concept: Parser-Blocking Resources**

```html
<!-- CSS is render-blocking (blocks render tree, NOT DOM parsing) -->
<link rel="stylesheet" href="styles.css">

<!-- Regular JS is parser-blocking (blocks DOM parsing AND execution) -->
<script src="app.js"></script>

<!-- async: downloads in parallel, executes ASAP (blocks parsing during execution) -->
<script async src="analytics.js"></script>

<!-- defer: downloads in parallel, executes AFTER DOM is fully parsed -->
<script defer src="app.js"></script>
```

**What you must know:**
```
                        DOM Parsing    Download    Execute When?
<script>                BLOCKED        Sequential  Immediately (blocks parsing)
<script async>          Continues      Parallel    When download completes (blocks briefly)
<script defer>          Continues      Parallel    After DOM parsed, before DOMContentLoaded
<script type="module">  Continues      Parallel    After DOM parsed (defer by default)
```

**Preload Scanner:** While the main parser is blocked by a `<script>`, a secondary
"preload scanner" continues scanning HTML to discover resources to fetch early.
This is why moving CSS to `<head>` matters — preload scanner finds it sooner.

---

### Stage 6: CSS Parsing → CSSOM Tree

```
CSS Text: "body { font-size: 16px; } .title { color: blue; }"
  ↓ (tokenize)
Tokens: [selector: body] [property: font-size] [value: 16px] ...
  ↓ (parse into rules)
CSSOM Tree:
  StyleSheet
  ├── Rule: body
  │   └── font-size: 16px
  └── Rule: .title
      └── color: blue
```

**CSS is render-blocking, NOT parser-blocking:**
- The browser continues parsing HTML while CSS downloads
- But it will NOT render anything until CSSOM is complete
- This is why you see a "white flash" if CSS is slow — browser waits for CSSOM

**Specificity calculation (you'll be asked this):**
```
Inline styles:           1,0,0,0
#id:                     0,1,0,0
.class, [attr], :pseudo: 0,0,1,0
element, ::pseudo:       0,0,0,1

Example:
div.container #main .title → 0,1,2,1
(0 inline, 1 ID, 2 classes, 1 element)
```

---

### Stage 7: JavaScript Execution

**When the parser encounters a `<script>` tag (without async/defer):**
1. Pause DOM parsing
2. If external script: wait for download
3. If CSS is still loading: wait for CSSOM (JS might read styles)
4. Execute JavaScript
5. JS may modify DOM (document.write, appendChild, etc.)
6. Resume DOM parsing

**This is why script placement matters:**
```html
<!-- BAD: Blocks parsing of entire body -->
<head>
  <script src="heavy-app.js"></script>
</head>

<!-- GOOD: DOM parsed first, then scripts execute -->
<head>
  <script defer src="heavy-app.js"></script>
</head>

<!-- ALSO GOOD: Script at bottom, DOM already parsed -->
<body>
  <!-- all content -->
  <script src="heavy-app.js"></script>
</body>
```

---

### Stage 8: Render Tree Construction

```
DOM Tree + CSSOM Tree = Render Tree

DOM:                    CSSOM:                  Render Tree:
html                    html { }                html
├── head                head { }                └── body
│   ├── meta            .hidden {                   ├── h1 (visible)
│   └── title             display: none }            │   └── "Hello"
└── body                h1 { color: red }           └── p (visible)
    ├── h1              p { font-size: 14px }           └── "World"
    ├── p
    └── span.hidden   ← NOT in Render Tree (display: none)
```

**What you must know:**
- `display: none` → NOT in render tree (no space taken, no box generated)
- `visibility: hidden` → IS in render tree (space taken, box generated, just invisible)
- `opacity: 0` → IS in render tree (space taken, box generated, just transparent)
- `<head>`, `<script>`, `<meta>` → NOT in render tree
- Render tree only contains VISIBLE nodes with computed styles

---

### Stage 9: Layout (Reflow)

The browser calculates the **exact position and size** of every element in the render tree.

```
Layout calculates:
- x, y position (relative to viewport and parent)
- width, height
- margins, paddings, borders
- This is a RECURSIVE process (parent affects children)
```

**Layout is EXPENSIVE. These trigger layout:**
```javascript
// Reading these properties triggers synchronous layout:
element.offsetTop / offsetLeft / offsetWidth / offsetHeight
element.clientTop / clientLeft / clientWidth / clientHeight
element.scrollTop / scrollLeft / scrollWidth / scrollHeight
element.getBoundingClientRect()
window.getComputedStyle()
window.scrollY / scrollX

// Writing then reading causes "Forced Synchronous Layout" (very bad):
element.style.width = '100px';   // Write (schedule layout)
const h = element.offsetHeight;   // Read (FORCES layout NOW)
element.style.height = h + 'px'; // Write (schedule layout again)
const w = element.offsetWidth;    // Read (FORCES layout AGAIN)

// BETTER: Batch reads, then batch writes
const h = element.offsetHeight;   // Read
const w = element.offsetWidth;    // Read
element.style.width = '100px';   // Write
element.style.height = h + 'px'; // Write
```

**Layout Thrashing (the #1 performance killer in DOM manipulation):**
```javascript
// BAD: Causes layout on EVERY iteration
for (let i = 0; i < 1000; i++) {
  items[i].style.width = container.offsetWidth + 'px'; // Read + Write in loop!
}

// GOOD: Read once, write many
const width = container.offsetWidth; // One read
for (let i = 0; i < 1000; i++) {
  items[i].style.width = width + 'px'; // Write only
}
```

---

### Stage 10: Paint

Converts the render tree into actual pixels. Paint fills in:
- Colors (background, text)
- Borders
- Shadows
- Text (glyph rendering)
- Images

**Paint is per-layer.** The browser creates multiple layers (like Photoshop layers).

**What triggers paint (but NOT layout):**
```
color, background-color, background-image
border-color, border-radius
box-shadow, text-shadow
outline, visibility
```

**What triggers ONLY composite (cheapest — GPU only):**
```
transform (translate, scale, rotate)
opacity
filter
will-change
```

**This is why CSS animations should use `transform` and `opacity`:**
```css
/* BAD: Triggers layout + paint every frame */
.animate {
  transition: left 0.3s, top 0.3s;
  left: 100px;
  top: 100px;
}

/* GOOD: Only triggers composite (GPU-accelerated) */
.animate {
  transition: transform 0.3s;
  transform: translate(100px, 100px);
}
```

---

### Stage 11: Composite

The browser takes all painted layers and composites them together in the correct order.

**Why layers exist:**
- Elements with `transform`, `opacity`, `will-change`, `position: fixed` get their own layer
- Layers can be independently repainted and composited
- GPU handles composition — very fast

**The Pixel Pipeline (know this diagram):**
```
JavaScript → Style → Layout → Paint → Composite
                                         ↑
                                    GPU handles this

Best case (transform/opacity change):
JavaScript → Style → ───────────── → Composite  ← CHEAPEST

Medium case (color change):
JavaScript → Style → ──── → Paint → Composite

Worst case (width/height change):
JavaScript → Style → Layout → Paint → Composite  ← MOST EXPENSIVE
```

---

## 🔨 Build Project (4 hours)

### Project: Browser Rendering Visualizer

Build a plain HTML/CSS/JS page (NO frameworks) that demonstrates rendering concepts:

```
rendering-demo/
├── index.html
├── style.css
└── script.js
```

**Features to build:**

#### 1. Layout Thrashing Demo
```javascript
// Button 1: "Thrashing" — read/write in loop, show how slow it is
// Button 2: "Batched" — batch reads then writes, show the speedup
// Display: Time taken for each approach

function layoutThrashing() {
  const items = document.querySelectorAll('.item');
  const start = performance.now();

  // Bad: read+write in loop
  items.forEach(item => {
    const width = item.offsetWidth; // forces layout
    item.style.width = (width + 1) + 'px';
  });

  console.log(`Thrashing: ${performance.now() - start}ms`);
}

function batchedLayout() {
  const items = document.querySelectorAll('.item');
  const start = performance.now();

  // Good: read all, then write all
  const widths = Array.from(items).map(item => item.offsetWidth);
  items.forEach((item, i) => {
    item.style.width = (widths[i] + 1) + 'px';
  });

  console.log(`Batched: ${performance.now() - start}ms`);
}
```

#### 2. Paint vs Composite Animation
```javascript
// Toggle between two animations:
// Animation A: Moves element using `left/top` (triggers layout+paint)
// Animation B: Moves element using `transform` (triggers only composite)
// Use Chrome DevTools → Performance tab → record both
// Show the difference in frame rate and paint rectangles
```

#### 3. Reflow Trigger Detector
```javascript
// Build a list of DOM properties
// When user clicks any property name, show whether it triggers:
// - Layout (reflow)
// - Paint
// - Composite only
// Reference: csstriggers.com
```

#### 4. Script Loading Comparison
```html
<!-- Create 4 pages, each loading scripts differently -->
<!-- Page 1: <script> in <head> (parser blocking) -->
<!-- Page 2: <script defer> in <head> -->
<!-- Page 3: <script async> in <head> -->
<!-- Page 4: <script> at end of <body> -->
<!-- Measure DOMContentLoaded and load event timing for each -->
<!-- Display results in a comparison table -->
```

**Use Chrome DevTools throughout:**
- Performance tab → Record page load → analyze Rendering events
- Turn on "Paint flashing" (green rectangles show repaints)
- Turn on "Layout Shift Regions" (blue rectangles show layout changes)
- Layers tab → See compositing layers

---

## 🧪 Experiments to Run in DevTools (1 hour)

### Experiment 1: See the DOM Tree Build
1. Open Chrome DevTools → Network tab
2. Throttle to "Slow 3G"
3. Load any large page (e.g., wikipedia.org)
4. Watch DOM elements appear incrementally (browser streams parsing)

### Experiment 2: See Paint Rectangles
1. DevTools → More tools → Rendering
2. Check "Paint flashing"
3. Scroll a page — see what repaints
4. Hover over elements — see what repaints
5. Open a dropdown — see the paint area

### Experiment 3: See Compositor Layers
1. DevTools → More tools → Layers
2. Load a page with `position: fixed` elements
3. See how fixed elements get their own layer
4. Add `will-change: transform` to an element — see new layer appear
5. Note: Too many layers = memory overhead

### Experiment 4: See Forced Reflow
1. DevTools → Performance tab → Record
2. Run the layout thrashing code above
3. Look for purple "Layout" events
4. See "Forced reflow" warnings in the console

---

## ❓ Interview Questions You Must Answer (2 hours)

Practice answering these OUT LOUD (pretend you're in an interview):

### Basic (Must answer perfectly):
1. **What happens when you type a URL in the browser and press Enter?**
   - Give the complete answer: DNS → TCP → TLS → HTTP → Parse → Render
   - A Staff answer takes 5-8 minutes covering each stage

2. **What is the difference between `<script>`, `<script async>`, and `<script defer>`?**
   - Explain when each downloads and executes relative to DOM parsing
   - Draw the timeline diagram

3. **What is the difference between `display: none`, `visibility: hidden`, and `opacity: 0`?**
   - Which is in the render tree? Which takes space? Which is clickable?

4. **What is the Critical Rendering Path?**
   - HTML → DOM, CSS → CSSOM, DOM + CSSOM → Render Tree → Layout → Paint → Composite
   - What blocks rendering? (CSS, synchronous JS)

5. **What is reflow (layout)? What triggers it?**
   - List at least 5 properties that trigger reflow when read

### Intermediate (Must answer well):
6. **What is layout thrashing? How do you fix it?**
   - Explain with code example
   - Mention `requestAnimationFrame` for batching

7. **Why should animations use `transform` and `opacity` instead of `top/left/width`?**
   - Explain the pixel pipeline: Layout → Paint → Composite
   - GPU acceleration for transform/opacity

8. **What is the preload scanner? Why does it exist?**
   - When main parser is blocked, secondary scanner discovers resources
   - Why this makes `<link>` in `<head>` important

9. **What is a compositing layer? When is one created?**
   - `transform`, `opacity`, `will-change`, `position: fixed`, `video`, `canvas`
   - Tradeoff: More layers = better isolation but more memory

10. **What is the difference between `DOMContentLoaded` and `load` events?**
    - `DOMContentLoaded`: DOM tree built, defer scripts executed (CSS/images may still load)
    - `load`: Everything loaded (images, stylesheets, iframes)

### Advanced (Staff-level depth):
11. **How does the browser handle CSS specificity conflicts?**
    - Specificity calculation, cascade order, `!important`, layers (`@layer`)

12. **Explain how `requestAnimationFrame` works and why it matters for rendering.**
    - Synced to display refresh rate (usually 60fps = 16.67ms per frame)
    - Callback runs BEFORE next paint
    - Better than `setTimeout` for animations (no drift, no wasted frames)

13. **What is `content-visibility: auto` and how does it help performance?**
    - Skips rendering of off-screen content
    - Element has layout, but children are not rendered until visible
    - Can dramatically reduce initial render time for long pages

14. **What are render-blocking vs parser-blocking resources?**
    - CSS: render-blocking (blocks render, not parsing)
    - JS (sync): parser-blocking (blocks parsing AND rendering)
    - Explain why this distinction matters for optimization

15. **How would you optimize the critical rendering path for a large application?**
    - Inline critical CSS
    - Defer non-critical CSS
    - async/defer scripts
    - Preload critical resources
    - Reduce DOM depth
    - Minimize render-blocking resources

---

## ✅ Week 1 Completion Checklist

- [ ] Read all 3 articles (Tali Garsiel, Mariko Kosaka series, Paul Lewis)
- [ ] Can draw the full URL → Pixels pipeline from memory
- [ ] Can explain every stage (DNS → TCP → TLS → HTTP → Parse → Render → Paint → Composite)
- [ ] Built the rendering demo project
- [ ] Ran all 4 DevTools experiments
- [ ] Can answer all 15 interview questions out loud
- [ ] Understand: parser-blocking vs render-blocking
- [ ] Understand: layout thrashing and how to avoid it
- [ ] Understand: pixel pipeline (JS → Style → Layout → Paint → Composite)
- [ ] Understand: why transform/opacity are cheap but width/height are expensive

---

Next → `02-javascript-engine.md`
