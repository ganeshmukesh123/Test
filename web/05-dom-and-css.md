# Part 5: DOM APIs & CSS Deep Knowledge 🎨

## Week 5 | 10 hours | Master the platform, not just the framework

---

## Why This Matters

You've built UIs with Vue for 7 years — Vue handles DOM manipulation for you.
But in FAANG interviews, you'll be asked to:
- Build UI components with RAW DOM APIs (no framework)
- Explain CSS layout algorithms (how does flexbox actually work?)
- Debug layout issues that frameworks can't hide
- Understand performance implications of DOM operations

**A Staff Engineer knows what the framework does for them AND can work without it.**

---

## 📖 Reading Plan (3 hours)

### DOM:
1. **javascript.info — DOM section** (read all)
   - https://javascript.info/document
   - Walking the DOM, searching, modifying, styles, events

2. **MDN: Introduction to the DOM**
   - https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction

3. **MDN: Event reference**
   - https://developer.mozilla.org/en-US/docs/Web/Events
   - Focus on: bubbling, capturing, delegation, passive listeners

### CSS:
4. **"Every Layout"** — Heydon Pickering & Andy Bell
   - https://every-layout.dev/ (paid, but excellent)
   - Alternative (free): "CSS Layout" by Rachel Andrew

5. **MDN: CSS Layout**
   - Flexbox: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout
   - Grid: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout
   - Containing Block: https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block

6. **"What No One Told You About Z-Index"** — Philip Walton
   - https://philipwalton.com/articles/what-no-one-told-you-about-z-index/

---

## 🧠 Deep Dive: DOM

### DOM Tree Navigation

```javascript
// The DOM is a TREE of nodes
// Every element, text, comment is a node

document.documentElement  // <html>
document.head             // <head>
document.body             // <body>

// Navigation properties:
element.parentNode        // parent node
element.parentElement     // parent element (null for document)
element.childNodes        // NodeList (includes text, comments)
element.children          // HTMLCollection (elements only)
element.firstChild        // first child node (may be text)
element.firstElementChild // first child element
element.lastChild
element.lastElementChild
element.nextSibling       // next node (may be text)
element.nextElementSibling // next element
element.previousSibling
element.previousElementSibling
```

### DOM Searching

```javascript
// By ID (fastest — hash map lookup)
document.getElementById('myId')           // single element

// By CSS selector (most flexible)
document.querySelector('.class')          // first match
document.querySelectorAll('.class')       // all matches (static NodeList)

// By class/tag (returns live collection — updates automatically)
document.getElementsByClassName('class')  // live HTMLCollection
document.getElementsByTagName('div')      // live HTMLCollection

// Important difference:
const staticList = document.querySelectorAll('div');
const liveList = document.getElementsByTagName('div');

document.body.appendChild(document.createElement('div'));

staticList.length; // unchanged (snapshot at query time)
liveList.length;   // +1 (live — reflects current DOM)

// element.closest() — searches UP the tree
button.closest('.card');  // nearest ancestor matching selector
// Extremely useful for event delegation!

// element.matches() — tests if element matches selector
element.matches('.active'); // true/false
```

### DOM Manipulation

```javascript
// Creating elements
const div = document.createElement('div');
div.className = 'card';
div.textContent = 'Hello';
div.innerHTML = '<span>Hello</span>'; // parses HTML (security risk with user input!)

// Setting attributes
div.setAttribute('data-id', '123');
div.dataset.id;  // '123' (dataset API for data-* attributes)

// Inserting
parent.appendChild(child);                    // append at end
parent.insertBefore(newChild, referenceChild); // insert before
parent.prepend(child);                         // insert at beginning
parent.append(child);                          // append at end (accepts strings too)
element.before(newElement);                    // insert before element
element.after(newElement);                     // insert after element

// Removing
element.remove();                // modern way
parent.removeChild(child);       // old way

// Replacing
parent.replaceChild(newChild, oldChild);  // old way
oldElement.replaceWith(newElement);       // modern way

// Cloning
const clone = element.cloneNode(true);  // true = deep clone (with children)
const shallow = element.cloneNode(false); // false = element only, no children
```

### DOM Performance — DocumentFragment

```javascript
// BAD: Each appendChild triggers potential reflow
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  ul.appendChild(li); // 1000 potential reflows!
}

// GOOD: Build in DocumentFragment, insert once
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  fragment.appendChild(li); // No reflow — fragment is not in DOM
}
ul.appendChild(fragment); // ONE reflow

// ALSO GOOD: Build with innerHTML (faster for large HTML strings)
const html = Array.from({ length: 1000 }, (_, i) => `<li>Item ${i}</li>`).join('');
ul.innerHTML = html; // ONE parse + ONE reflow

// BEST for many updates: Use requestAnimationFrame batching
function batchDOMUpdates(updates) {
  requestAnimationFrame(() => {
    updates.forEach(update => update());
  });
}
```

---

### Event System — Complete Understanding

```
EVENT PHASES:
                        ┌─────────┐
                        │ WINDOW  │
                        └────┬────┘
                 CAPTURING ↓   ↑ BUBBLING
                        ┌────┴────┐
                        │DOCUMENT │
                        └────┬────┘
                 CAPTURING ↓   ↑ BUBBLING
                        ┌────┴────┐
                        │  <html> │
                        └────┬────┘
                 CAPTURING ↓   ↑ BUBBLING
                        ┌────┴────┐
                        │  <body> │
                        └────┬────┘
                 CAPTURING ↓   ↑ BUBBLING
                        ┌────┴────┐
                        │  <div>  │
                        └────┬────┘
                 CAPTURING ↓   ↑ BUBBLING
                        ┌────┴────┐
                        │ <button>│  ← TARGET (Phase 2)
                        └─────────┘

Phase 1: CAPTURING — event travels DOWN from window to target
Phase 2: TARGET — event reaches the clicked element
Phase 3: BUBBLING — event travels UP from target to window
```

```javascript
// addEventListener — third argument controls phase
element.addEventListener('click', handler, false); // BUBBLING (default)
element.addEventListener('click', handler, true);  // CAPTURING
element.addEventListener('click', handler, {
  capture: true,    // listen during capture phase
  once: true,       // auto-remove after first trigger
  passive: true,    // promise to not call preventDefault (performance!)
  signal: controller.signal  // AbortController for removal
});

// Stopping propagation
event.stopPropagation();          // Stop event from traveling further
event.stopImmediatePropagation(); // Stop + prevent other handlers on same element
event.preventDefault();           // Prevent default browser behavior (not propagation)

// IMPORTANT: stopPropagation vs preventDefault
// Click on <a href="/page">:
//   stopPropagation() → link still navigates! (default behavior happens)
//   preventDefault()  → link does NOT navigate (but event still bubbles!)
//   Both needed if you want to stop everything
```

**Event Delegation (crucial pattern):**
```javascript
// Instead of adding listener to each child:
// ❌ BAD: 1000 listeners for 1000 items
document.querySelectorAll('.item').forEach(item => {
  item.addEventListener('click', handleClick);
});

// ✅ GOOD: 1 listener on parent, detect which child was clicked
document.querySelector('.list').addEventListener('click', (event) => {
  const item = event.target.closest('.item'); // find nearest .item ancestor
  if (!item) return; // click wasn't on an item

  handleClick(item);
});

// Benefits:
// 1. Fewer event listeners (memory efficient)
// 2. Works for dynamically added elements
// 3. Single point of event handling
```

**Passive Event Listeners (performance):**
```javascript
// Scroll and touch events block rendering while handler runs
// because browser doesn't know if you'll call preventDefault()

// { passive: true } tells browser: "I won't call preventDefault()"
// Browser can start scrolling IMMEDIATELY without waiting for handler

// GOOD: Passive scroll listener (browser scrolls immediately)
window.addEventListener('scroll', handleScroll, { passive: true });

// BAD: Non-passive scroll listener (browser waits for handler before scrolling)
window.addEventListener('scroll', handleScroll); // default is passive: false for scroll

// Chrome shows warning: "Added non-passive event listener to a
// scroll-blocking event. Consider marking event handler as 'passive'..."
```

### Observer APIs

```javascript
// 1. IntersectionObserver — detect when element enters/leaves viewport
// USE FOR: Lazy loading, infinite scroll, analytics (visibility tracking)
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // Element is visible
      loadImage(entry.target);
      observer.unobserve(entry.target); // stop observing after load
    }
  });
}, {
  root: null,           // viewport (default)
  rootMargin: '100px',  // start loading 100px before visible
  threshold: 0.1        // trigger when 10% visible
});

document.querySelectorAll('img[data-src]').forEach(img => {
  observer.observe(img);
});

// 2. MutationObserver — detect DOM changes
// USE FOR: Third-party script monitoring, DOM-based reactivity
const mutationObserver = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    console.log('DOM changed:', mutation.type);
    // mutation.type: 'childList', 'attributes', 'characterData'
    // mutation.addedNodes, mutation.removedNodes
  });
});

mutationObserver.observe(element, {
  childList: true,    // watch for added/removed children
  attributes: true,   // watch for attribute changes
  subtree: true,      // watch entire subtree
  characterData: true // watch for text changes
});

// 3. ResizeObserver — detect element size changes
// USE FOR: Responsive components, chart resizing
const resizeObserver = new ResizeObserver((entries) => {
  entries.forEach(entry => {
    const { width, height } = entry.contentRect;
    console.log(`Element resized: ${width}x${height}`);
    // Resize chart, adjust layout, etc.
  });
});

resizeObserver.observe(element);

// 4. PerformanceObserver — detect performance entries
// USE FOR: Monitoring Core Web Vitals, resource timing
const perfObserver = new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    if (entry.entryType === 'largest-contentful-paint') {
      console.log('LCP:', entry.startTime);
    }
  }
});

perfObserver.observe({ type: 'largest-contentful-paint', buffered: true });
```

---

## 🧠 Deep Dive: CSS Internals

### Box Model — The Foundation

```
┌──────────────────────────────────────────────────┐
│                    MARGIN                         │
│  ┌──────────────────────────────────────────┐    │
│  │                BORDER                     │    │
│  │  ┌──────────────────────────────────┐    │    │
│  │  │            PADDING                │    │    │
│  │  │  ┌──────────────────────────┐    │    │    │
│  │  │  │        CONTENT           │    │    │    │
│  │  │  │   width × height        │    │    │    │
│  │  │  └──────────────────────────┘    │    │    │
│  │  └──────────────────────────────────┘    │    │
│  └──────────────────────────────────────────┘    │
└──────────────────────────────────────────────────┘

box-sizing: content-box (default)
  → width = content only
  → Total width = width + padding + border

box-sizing: border-box (recommended)
  → width = content + padding + border
  → Total width = width (what you set is what you get)

ALWAYS use: *, *::before, *::after { box-sizing: border-box; }
```

### Block Formatting Context (BFC)

```css
/* A BFC is an isolated layout context. Elements inside a BFC
   don't affect layout outside, and vice versa. */

/* What creates a BFC: */
float: left/right;
position: absolute/fixed;
display: inline-block;
display: flow-root;         /* best way — specifically designed for this */
overflow: hidden/auto/scroll; /* creates BFC as side effect */
display: flex/grid;          /* flex/grid items create BFCs */
contain: layout;

/* Why BFC matters: */

/* Problem 1: Margin Collapsing */
/* Adjacent vertical margins collapse (take the larger value) */
.box1 { margin-bottom: 20px; }
.box2 { margin-top: 30px; }
/* Gap between them: 30px (not 50px!) — margins collapsed */

/* BFC prevents margin collapsing between parent and child: */
.parent {
  display: flow-root; /* creates BFC — child margins stay inside */
}

/* Problem 2: Clearing Floats */
/* Parent with only floated children has zero height */
.parent {
  display: flow-root; /* BFC contains floats — parent gets height */
}

/* Problem 3: Float wrapping */
/* Text wraps around float. BFC sibling doesn't wrap: */
.sibling {
  display: flow-root; /* won't overlap with float */
}
```

### Stacking Context & Z-Index

```
CRITICAL: z-index ONLY works within the same stacking context!

A new stacking context is created by:
- Root element (<html>)
- position: relative/absolute/fixed/sticky + z-index (not auto)
- display: flex/grid child + z-index (not auto)
- opacity less than 1
- transform (any value)
- filter (any value)
- will-change: opacity/transform
- isolation: isolate
- mix-blend-mode (not normal)
- contain: paint/layout/content/strict

THE GOTCHA:
```

```html
<div style="position: relative; z-index: 1;">  <!-- Stacking context A -->
  <div style="position: relative; z-index: 999;">
    I have z-index 999 but I'm INSIDE context A (z-index: 1)
  </div>
</div>

<div style="position: relative; z-index: 2;">  <!-- Stacking context B -->
  <div style="position: relative; z-index: 1;">
    I have z-index 1 but I'm INSIDE context B (z-index: 2)
    I appear ABOVE the z-index: 999 element!
  </div>
</div>

<!-- Context B (z-index: 2) is above Context A (z-index: 1) -->
<!-- So ALL children of B are above ALL children of A -->
<!-- The z-index: 999 inside A cannot escape context A -->
```

```css
/* Best practice: Use isolation: isolate to create intentional stacking contexts */
.modal-overlay {
  isolation: isolate; /* nothing inside can affect stacking outside */
  z-index: 100;
}

/* Even better: Use CSS custom properties for z-index management */
:root {
  --z-dropdown: 100;
  --z-sticky: 200;
  --z-modal-overlay: 300;
  --z-modal: 400;
  --z-toast: 500;
  --z-tooltip: 600;
}
```

### Flexbox — How It Actually Works

```css
/* Flexbox layout algorithm (simplified): */

/* 1. Determine available space on main axis */
/* 2. Calculate each item's base size (flex-basis or width/height) */
/* 3. Distribute remaining space based on flex-grow */
/* 4. Shrink items if needed based on flex-shrink */
/* 5. Align items on cross axis */

.container {
  display: flex;
  flex-direction: row;     /* main axis = horizontal */
  justify-content: center; /* align on MAIN axis */
  align-items: center;     /* align on CROSS axis */
  gap: 16px;               /* gap between items */
  flex-wrap: wrap;          /* allow wrapping to new lines */
}

.item {
  flex-grow: 1;    /* how much of remaining space to take (ratio) */
  flex-shrink: 1;  /* how much to shrink when space is tight (ratio) */
  flex-basis: 200px; /* starting size before grow/shrink */
  /* Shorthand: flex: 1 1 200px; (grow shrink basis) */
}

/* Common shortcuts: */
flex: 1;       /* = flex: 1 1 0%  → grow equally, start from 0 */
flex: auto;    /* = flex: 1 1 auto → grow equally, start from content size */
flex: none;    /* = flex: 0 0 auto → don't grow or shrink */
flex: 0 1 auto; /* default → don't grow, can shrink, natural size */
```

**Flexbox algorithm detail:**
```
Container width: 600px
Item A: flex: 2 1 100px  (grow:2, shrink:1, basis:100px)
Item B: flex: 1 1 100px  (grow:1, shrink:1, basis:100px)
Item C: flex: 1 1 100px  (grow:1, shrink:1, basis:100px)

Step 1: Total base size = 100 + 100 + 100 = 300px
Step 2: Remaining space = 600 - 300 = 300px
Step 3: Total grow = 2 + 1 + 1 = 4
Step 4: Distribute:
  A gets: 100 + (2/4 × 300) = 100 + 150 = 250px
  B gets: 100 + (1/4 × 300) = 100 + 75  = 175px
  C gets: 100 + (1/4 × 300) = 100 + 75  = 175px
  Total: 250 + 175 + 175 = 600px ✓
```

### Grid — How It Actually Works

```css
.container {
  display: grid;

  /* Define columns and rows */
  grid-template-columns: 1fr 2fr 1fr;  /* 3 columns: 25% 50% 25% */
  grid-template-rows: auto 1fr auto;   /* header, main, footer */
  gap: 16px;

  /* Named areas (powerful for layout) */
  grid-template-areas:
    "header  header  header"
    "sidebar content aside"
    "footer  footer  footer";
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.aside   { grid-area: aside; }
.footer  { grid-area: footer; }

/* fr unit: fraction of remaining space */
/* 1fr 2fr 1fr → 1/4, 2/4, 1/4 of remaining space */

/* minmax: responsive without media queries */
grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
/* Creates as many 250px+ columns as fit, grows to fill */

/* auto-fill vs auto-fit: */
/* auto-fill: Creates empty tracks if space allows */
/* auto-fit: Collapses empty tracks (items stretch to fill) */
```

### CSS Containment (`contain`)

```css
/* contain tells the browser that an element's internals are independent */
/* Browser can optimize: skip recalculating stuff outside the container */

.card {
  contain: layout;   /* Layout changes inside don't affect outside */
  contain: paint;    /* Painting inside doesn't overflow */
  contain: size;     /* Element's size doesn't depend on children */
  contain: style;    /* Counters/quotes don't escape */

  contain: content;  /* = layout + paint (most common) */
  contain: strict;   /* = layout + paint + size (most aggressive) */
}

/* content-visibility: The BIG performance win */
.offscreen-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* estimated height for layout */
}
/* Browser skips rendering this element entirely until it's near viewport */
/* Can reduce initial render time by 50%+ for long pages */
```

### Modern CSS Features You Must Know

```css
/* 1. Container Queries (game-changer for component libraries) */
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card { flex-direction: row; }
}
@container card (max-width: 399px) {
  .card { flex-direction: column; }
}
/* Components respond to their CONTAINER size, not viewport */
/* This is what design systems need for truly reusable components */

/* 2. CSS Layers (@layer) — specificity management */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; }
}
@layer components {
  .button { background: blue; }
}
@layer utilities {
  .bg-red { background: red; } /* wins over .button even with lower specificity */
}
/* Later layers win regardless of specificity */
/* Perfect for design system + utility class coexistence */

/* 3. CSS Nesting (native!) */
.card {
  background: white;

  & .title {
    font-size: 1.5rem;
  }

  &:hover {
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  }

  @media (width < 768px) {
    padding: 1rem;
  }
}

/* 4. :has() selector (parent selector — was impossible before!) */
.card:has(img) {
  /* Style card differently if it contains an image */
  grid-template-rows: 200px auto;
}

form:has(:invalid) button[type="submit"] {
  /* Disable submit button if any field is invalid */
  opacity: 0.5;
  pointer-events: none;
}

/* 5. Logical Properties (for internationalization) */
.element {
  margin-inline-start: 1rem;  /* = margin-left in LTR, margin-right in RTL */
  margin-inline-end: 1rem;
  padding-block: 1rem;        /* = padding-top + padding-bottom */
  inline-size: 300px;         /* = width in horizontal writing mode */
  block-size: 200px;          /* = height in horizontal writing mode */
}
```

---

## 🔨 Build Project (4 hours)

### Project: Build UI Components with Raw DOM + CSS

**No Vue. No React. No framework. Raw DOM APIs + CSS.**

```
dom-css-project/
├── components/
│   ├── autocomplete.html    (search with dropdown suggestions)
│   ├── virtual-list.html    (render 10,000 items efficiently)
│   ├── modal.html           (accessible modal with focus trap)
│   ├── carousel.html        (image carousel with touch support)
│   └── tooltip.html         (positioned tooltip with collision detection)
├── css-demos/
│   ├── stacking-context.html
│   ├── container-queries.html
│   └── layout-algorithms.html
└── style.css
```

#### Component 1: Autocomplete (Interview favorite)
```javascript
// Requirements:
// - Text input that shows dropdown of filtered suggestions
// - Keyboard navigation (ArrowUp, ArrowDown, Enter, Escape)
// - Debounced API call (don't call on every keystroke)
// - Highlight matching text in suggestions
// - Accessible: ARIA roles (combobox, listbox, option)
// - Click outside closes dropdown
// - Loading state while fetching
// Build with: createElement, addEventListener, classList, dataset
```

#### Component 2: Virtual List (Performance pattern)
```javascript
// Requirements:
// - Render 100,000 items without crashing the browser
// - Only render visible items + small buffer
// - Smooth scrolling
// - Dynamic item heights
// Algorithm:
// 1. Calculate total height (sum of all item heights)
// 2. Create a container with that total height (scrollbar correct)
// 3. On scroll: calculate which items are visible
// 4. Render only visible items, positioned with transform: translateY()
// 5. Remove items that scroll out of view
```

#### Component 3: Accessible Modal
```javascript
// Requirements:
// - Focus trap (Tab cycles within modal, doesn't escape)
// - Return focus to trigger element on close
// - Close on Escape key
// - Close on backdrop click
// - Prevent body scroll when open
// - ARIA: role="dialog", aria-modal="true", aria-labelledby, aria-describedby
// - Announce to screen readers
```

---

## ❓ Interview Questions You Must Answer (2 hours)

### Basic:
1. **What is event bubbling? How is it different from capturing?**
2. **What is event delegation? Build an example.**
3. **What is the CSS Box Model? What does `box-sizing: border-box` do?**
4. **Explain CSS specificity. How is it calculated?**
5. **What is the difference between `position: relative`, `absolute`, `fixed`, `sticky`?**

### Intermediate:
6. **What is a stacking context? How does z-index work?**
7. **What is a Block Formatting Context? When is one created?**
8. **Explain the flexbox algorithm. How do `flex-grow`, `flex-shrink`, `flex-basis` work?**
9. **What is `IntersectionObserver`? How would you implement lazy loading?**
10. **Build an autocomplete component with raw DOM APIs.** (45 min)

### Advanced:
11. **What is `content-visibility: auto`? How does it improve performance?**
12. **Explain CSS containment (`contain` property). When would you use it?**
13. **What are Container Queries? Why are they important for design systems?**
14. **What is `CSS @layer`? How does it change specificity management?**
15. **Build a virtual list that renders 100,000 items efficiently.** (45 min)

---

## ✅ Week 5 Completion Checklist

- [ ] Can manipulate DOM without any framework
- [ ] Understand event phases: capturing, target, bubbling
- [ ] Can implement event delegation pattern
- [ ] Know all Observer APIs (Intersection, Mutation, Resize, Performance)
- [ ] Understand Box Model, BFC, stacking context deeply
- [ ] Understand flexbox and grid layout algorithms
- [ ] Know modern CSS: container queries, @layer, :has(), nesting, logical properties
- [ ] Understand CSS containment and content-visibility
- [ ] Built: Autocomplete, Virtual List, Modal with raw DOM
- [ ] Can answer all 15 interview questions out loud

---

Next → `06-web-platform-apis.md`
