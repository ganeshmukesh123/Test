# Part 2: JavaScript Engine Internals 🔧

## Week 2 | 10 hours | Understand HOW your code actually runs

---

## Why This Matters

You've written JavaScript for 7 years. But do you know:
- Why some code is 100x faster than similar code?
- How V8 optimizes your functions at runtime?
- Why memory leaks happen and how to find them?
- How the event loop ACTUALLY works (not the simplified version)?

**Staff Engineers understand the engine. It changes how you write code.**

---

## 🗺️ The Big Picture: JavaScript Engine Architecture

```
Your JavaScript Code
        ↓
┌─────────────────────────────────────────────────┐
│                    V8 ENGINE                     │
│                                                  │
│  [1] Parser ──→ AST (Abstract Syntax Tree)      │
│        ↓                                         │
│  [2] Ignition (Interpreter) ──→ Bytecode        │
│        ↓                                         │
│  [3] TurboFan (JIT Compiler) ──→ Machine Code   │
│        ↓                                         │
│  [4] Execution on CPU                           │
│                                                  │
│  ┌──────────────┐    ┌────────────────────┐     │
│  │  CALL STACK  │    │    MEMORY HEAP     │     │
│  │  (execution) │    │  (object storage)  │     │
│  └──────────────┘    └────────────────────┘     │
│                                                  │
│  ┌──────────────────────────────────────┐       │
│  │     GARBAGE COLLECTOR (Orinoco)      │       │
│  │  Mark-Sweep  |  Scavenge  |  Compact │       │
│  └──────────────────────────────────────┘       │
│                                                  │
└─────────────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────────────┐
│              BROWSER / NODE.js RUNTIME           │
│                                                  │
│  ┌──────────────────────────────────────┐       │
│  │            EVENT LOOP                 │       │
│  │  Macrotask Queue (setTimeout, I/O)    │       │
│  │  Microtask Queue (Promises, queueMT)  │       │
│  │  Animation Frames (rAF)               │       │
│  └──────────────────────────────────────┘       │
│                                                  │
│  Web APIs: DOM, fetch, setTimeout, etc.          │
│  (NOT part of V8 — provided by browser/Node)     │
└─────────────────────────────────────────────────┘
```

---

## 📖 Reading Plan (3 hours)

### Must Read:
1. **"JavaScript V8 Engine Explained"** — Lydia Hallie (visual guide)
   - https://www.lydiahallie.com/ (search for V8/event loop articles)
   - Visual, excellent for understanding compilation pipeline

2. **"How JavaScript Works: Under the Hood of V8"** — official V8 blog
   - https://v8.dev/blog
   - Read: "Launching Ignition and TurboFan"
   - Read: "Trash talk: the Orinoco garbage collector"
   - Read: "Blazingly fast parsing"

3. **"JavaScript Event Loop Explained"** — Jake Archibald
   - https://www.youtube.com/watch?v=cCOL7MC4Pl0 (video — "In The Loop")
   - This is the BEST event loop explanation ever made. Watch it twice.
   - Then read: https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/

4. **"Memory Management"** — MDN
   - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management
   - Covers: allocation, garbage collection, memory leaks

### Reference:
- "Visualizing memory management in V8" — Andrey Pechkurov (deepkit blog)
- "Understanding V8's Bytecode" — Franziska Hinkelmann

---

## 🧠 Deep Dive: The Compilation Pipeline

### Stage 1: Parsing → AST

```javascript
// Your code:
function add(a, b) {
  return a + b;
}

// Parser produces AST (Abstract Syntax Tree):
{
  type: "FunctionDeclaration",
  name: "add",
  params: [
    { type: "Identifier", name: "a" },
    { type: "Identifier", name: "b" }
  ],
  body: {
    type: "ReturnStatement",
    argument: {
      type: "BinaryExpression",
      operator: "+",
      left: { type: "Identifier", name: "a" },
      right: { type: "Identifier", name: "b" }
    }
  }
}
```

**What you must know:**
- Parsing is NOT free — large bundles take time to parse
- V8 uses **lazy parsing**: functions are fully parsed only when called
- This is why code splitting matters — less code to parse upfront
- You can see parse time in DevTools → Performance → "Parse Script" events

**Lazy Parsing Example:**
```javascript
// V8 does EAGER parsing of this (it's executed immediately):
const result = calculateTotal(items);

// V8 does LAZY parsing of this (parsed when first called):
function rareBranchHandler() {
  // This entire function body is only fully parsed when called
  // Saves startup time for functions that may never run
}
```

---

### Stage 2: Ignition (Interpreter) → Bytecode

```javascript
// Your code:
function add(a, b) {
  return a + b;
}

// Ignition produces bytecode (simplified):
// LdaNamedProperty a
// Add b
// Return
```

**What you must know:**
- Ignition interprets bytecode line by line — fast to start, slow to run repeatedly
- Every function starts in the interpreter
- Ignition collects **type feedback** (what types does `a + b` actually receive?)
- This type feedback is used by TurboFan for optimization

---

### Stage 3: TurboFan (JIT Compiler) → Optimized Machine Code

```
Ignition observes: add() is called 1000 times with (number, number)
TurboFan says: "I'll compile an optimized version that assumes numbers"

Optimized machine code: (simplified)
  Load register A with first argument
  Load register B with second argument
  Integer ADD instruction  ← CPU-level addition, incredibly fast
  Return result
```

**Critical Concept: Deoptimization**
```javascript
function add(a, b) {
  return a + b;
}

// First 1000 calls with numbers — TurboFan optimizes for numbers
for (let i = 0; i < 1000; i++) {
  add(i, i + 1); // number + number → optimized!
}

// Then you call with a string:
add("hello", "world"); // string + string!
// TurboFan: "My assumption was wrong!"
// DEOPTIMIZATION: Falls back to slow interpreter bytecode
// Has to re-learn types
```

**What you must know:**
- **Monomorphic** calls (same types every time) = fast (optimized)
- **Polymorphic** calls (2-4 different types) = slower
- **Megamorphic** calls (many different types) = slowest (not optimized)

```javascript
// GOOD: Monomorphic — always same shape
function processUser(user) {
  return user.name + ' ' + user.age;
}
processUser({ name: 'Alice', age: 30 });
processUser({ name: 'Bob', age: 25 });
// V8 optimizes: always { name: string, age: number }

// BAD: Megamorphic — different shapes
processUser({ name: 'Alice', age: 30 });
processUser({ name: 'Bob', age: 25, email: 'bob@test.com' });
processUser({ firstName: 'Charlie', years: 20 });
// V8 can't optimize — shapes keep changing
```

---

### Hidden Classes (Shapes / Maps)

V8 creates **hidden classes** (called "Maps" internally) to track object shapes.

```javascript
// V8 creates hidden class transitions:
const obj = {};           // Hidden class C0: {}
obj.x = 1;               // Hidden class C1: { x }
obj.y = 2;               // Hidden class C2: { x, y }

// Objects with SAME property order share hidden classes (fast):
const a = { x: 1, y: 2 }; // Hidden class: { x, y }
const b = { x: 3, y: 4 }; // Same hidden class! Shared optimization.

// Objects with DIFFERENT property order = different hidden classes (slow):
const c = { y: 2, x: 1 }; // Different hidden class: { y, x }
// c and a have different hidden classes even though same properties!
```

**Rules for V8-friendly code:**
1. Always initialize object properties in the same order
2. Don't add properties dynamically after creation
3. Don't delete properties (`delete obj.x` destroys hidden class)
4. Keep objects "monomorphic" — same shape throughout their lifetime

---

## 🧠 Deep Dive: The Event Loop

### This is the MOST important concept for frontend interviews

```
┌─────────────────────────────────────────────┐
│                CALL STACK                    │
│  (synchronous code executes here)           │
│  One function at a time. Single-threaded.   │
└──────────────────┬──────────────────────────┘
                   │
                   ↓ (when stack is empty)
┌─────────────────────────────────────────────┐
│              EVENT LOOP                      │
│                                              │
│  1. Execute all MICROTASKS                  │
│     (Promise.then, queueMicrotask,          │
│      MutationObserver)                       │
│                                              │
│  2. Execute ONE MACROTASK                   │
│     (setTimeout, setInterval, I/O,          │
│      MessageChannel)                         │
│                                              │
│  3. If it's time for rendering:             │
│     a. requestAnimationFrame callbacks      │
│     b. Style calculation                    │
│     c. Layout                               │
│     d. Paint                                │
│                                              │
│  4. Go back to step 1                       │
└─────────────────────────────────────────────┘
```

### The Critical Rule:
**ALL microtasks are drained BEFORE the next macrotask or render.**

```javascript
// What order does this print?
console.log('1: Sync');

setTimeout(() => console.log('2: Timeout'), 0);

Promise.resolve().then(() => console.log('3: Promise'));

queueMicrotask(() => console.log('4: Microtask'));

requestAnimationFrame(() => console.log('5: rAF'));

console.log('6: Sync');

// Answer:
// 1: Sync          (synchronous, runs immediately)
// 6: Sync          (synchronous, runs immediately)
// 3: Promise       (microtask, runs before any macrotask)
// 4: Microtask     (microtask, runs before any macrotask)
// 5: rAF           (runs before next paint, after microtasks)
// 2: Timeout       (macrotask, runs after microtasks and rAF)
```

### Microtask Queue Can Starve Everything Else:
```javascript
// DANGER: This creates an infinite microtask loop
// The page will FREEZE — no rendering, no macrotasks
function infiniteMicrotask() {
  Promise.resolve().then(infiniteMicrotask);
}
infiniteMicrotask(); // Page is now completely frozen

// Compare with macrotask — this does NOT freeze the page:
function infiniteMacrotask() {
  setTimeout(infiniteMacrotask, 0);
}
infiniteMacrotask(); // Page stays responsive (renders between timeouts)
```

### `requestAnimationFrame` Deep Dive:
```javascript
// rAF runs BEFORE the browser paints
// Typically 60fps = one rAF callback every ~16.67ms

// GOOD animation pattern:
function animate() {
  element.style.transform = `translateX(${position}px)`;
  position += 2;

  if (position < 500) {
    requestAnimationFrame(animate); // Schedule next frame
  }
}
requestAnimationFrame(animate);

// BAD: Using setTimeout for animation
function badAnimate() {
  element.style.transform = `translateX(${position}px)`;
  position += 2;
  setTimeout(badAnimate, 16); // NOT synced to display, causes jank
}
```

### `requestIdleCallback`:
```javascript
// Runs when browser is IDLE (no pending tasks)
// Good for low-priority work
requestIdleCallback((deadline) => {
  // deadline.timeRemaining() tells you how much time you have
  while (deadline.timeRemaining() > 0 && tasks.length > 0) {
    performTask(tasks.pop());
  }

  if (tasks.length > 0) {
    requestIdleCallback(processRemainingTasks); // Continue when idle again
  }
});

// React's Concurrent Mode is inspired by this concept
// Break work into chunks, yield to browser between chunks
```

---

## 🧠 Deep Dive: Memory Management & Garbage Collection

### Memory Lifecycle:
```
1. ALLOCATE: When you create variables, objects, functions
2. USE: Read/write the allocated memory
3. RELEASE: When no longer needed → garbage collector reclaims
```

### V8's Memory Structure:
```
┌─────────────────────────────────────────┐
│               V8 HEAP                    │
│                                          │
│  ┌────────────────────────────────┐     │
│  │   NEW SPACE (Young Generation) │     │
│  │   - Small (1-8 MB)            │     │
│  │   - Short-lived objects        │     │
│  │   - Scavenge GC (fast, frequent)│    │
│  │   - Semi-space: From + To      │     │
│  └────────────────────────────────┘     │
│                                          │
│  ┌────────────────────────────────┐     │
│  │   OLD SPACE (Old Generation)   │     │
│  │   - Large (hundreds of MB)     │     │
│  │   - Long-lived objects         │     │
│  │   - Mark-Sweep-Compact GC     │     │
│  │     (slow, infrequent)         │     │
│  └────────────────────────────────┘     │
│                                          │
│  ┌────────────────────────────────┐     │
│  │   LARGE OBJECT SPACE           │     │
│  │   - Objects > 256KB            │     │
│  │   - Never moved by GC          │     │
│  └────────────────────────────────┘     │
│                                          │
│  ┌────────────────────────────────┐     │
│  │   CODE SPACE                    │     │
│  │   - JIT compiled machine code   │     │
│  └────────────────────────────────┘     │
└─────────────────────────────────────────┘
```

### Garbage Collection Algorithms:

**Scavenge (Young Generation — fast):**
```
New Space is split into two halves: FROM and TO

1. Objects allocated in FROM space
2. When FROM is full, GC starts:
   - Check which objects are still reachable
   - Copy live objects to TO space
   - Dead objects left in FROM are gone
3. Swap FROM and TO labels
4. Objects that survive 2 scavenges → promoted to Old Space
```

**Mark-Sweep-Compact (Old Generation — slow):**
```
1. MARK: Start from GC roots (global, stack), traverse all references
   - Reachable objects are marked as "alive"
   - Unreachable objects are marked as "dead"

2. SWEEP: Reclaim memory of dead objects
   - Creates holes in memory (fragmentation)

3. COMPACT: Move live objects together
   - Eliminates fragmentation
   - Expensive but necessary periodically
```

### Common Memory Leaks:

```javascript
// LEAK 1: Forgotten event listeners
function setup() {
  const data = loadHugeDataset(); // 50MB of data

  document.addEventListener('click', () => {
    console.log(data); // Closure holds reference to data
    // data can NEVER be garbage collected while listener exists
  });
}
// FIX: Remove listener when no longer needed
// FIX: Use AbortController for cleanup

// LEAK 2: Closures holding unnecessary references
function createHandler() {
  const hugeArray = new Array(1000000).fill('data');
  const name = 'handler';

  return function() {
    // Only uses `name`, but closure captures ENTIRE scope
    // hugeArray is retained in memory!
    console.log(name);
  };
}
// FIX: Restructure so closure only captures what it needs

// LEAK 3: Detached DOM nodes
let detachedNodes = [];
function createAndRemove() {
  const div = document.createElement('div');
  document.body.appendChild(div);
  document.body.removeChild(div);
  detachedNodes.push(div); // Reference kept! DOM node stays in memory
}
// FIX: Don't keep references to removed DOM nodes

// LEAK 4: Forgotten timers
function startPolling() {
  setInterval(() => {
    const data = fetchData(); // Runs forever
    updateUI(data);
  }, 1000);
  // No clearInterval! Even if component is destroyed, timer runs
}
// FIX: Store interval ID, clear on cleanup

// LEAK 5: Global variables
function processData() {
  result = expensiveComputation(); // No let/const → global variable!
  // `result` lives forever on window object
}
// FIX: Always use let/const

// LEAK 6: Map/Set with object keys
const cache = new Map();
function cacheResult(obj, result) {
  cache.set(obj, result); // obj is now strongly referenced by Map
  // Even if nothing else references obj, Map keeps it alive
}
// FIX: Use WeakMap — allows GC when no other references exist
const weakCache = new WeakMap();
```

---

## 🔨 Build Project (4 hours)

### Project: Event Loop Visualizer + Memory Profiler

```
js-engine-demo/
├── event-loop/
│   ├── index.html
│   └── script.js
├── memory/
│   ├── index.html
│   └── script.js
└── v8-optimization/
    ├── index.html
    └── script.js
```

#### Part A: Event Loop Visualizer
```javascript
// Build a visual tool that shows:
// 1. A "Call Stack" display (shows current executing function)
// 2. A "Microtask Queue" display
// 3. A "Macrotask Queue" display
// 4. A "Rendering" indicator
//
// User inputs code snippets and sees the execution order:
//
// Input:
//   console.log('A');
//   setTimeout(() => console.log('B'), 0);
//   Promise.resolve().then(() => console.log('C'));
//   console.log('D');
//
// Visualization:
//   Step 1: [Stack: console.log('A')] → Output: A
//   Step 2: [Stack: setTimeout] → [Macrotask Queue: log('B')]
//   Step 3: [Stack: Promise.then] → [Microtask Queue: log('C')]
//   Step 4: [Stack: console.log('D')] → Output: D
//   Step 5: [Microtask Queue → Stack: log('C')] → Output: C
//   Step 6: [Macrotask Queue → Stack: log('B')] → Output: B
```

#### Part B: Memory Leak Detector
```javascript
// Build a page with buttons that intentionally create memory leaks:
// 1. "Create Event Listener Leak" — adds listeners without removing
// 2. "Create Closure Leak" — closures holding large data
// 3. "Create Detached DOM Leak" — removed DOM nodes still referenced
// 4. "Create Timer Leak" — setIntervals never cleared
//
// Each button has a "Fix" variant that shows the correct approach
//
// Display: Memory usage chart (using performance.memory if available)
// Guide user to use DevTools → Memory tab → Heap Snapshot
```

#### Part C: V8 Optimization Benchmark
```javascript
// Build a benchmark that demonstrates V8 optimization:
//
// Test 1: Monomorphic vs Polymorphic function calls
function addMonomorphic(a, b) { return a + b; }
// Call 10000 times with (number, number) — measure time

function addPolymorphic(a, b) { return a + b; }
// Call with (number, number), (string, string), alternating — measure time

// Test 2: Object shape consistency
// Create 10000 objects with same property order vs random order
// Measure access time

// Test 3: Array type consistency
// Packed SMI array vs holey array vs mixed-type array
// const packed = [1, 2, 3, 4, 5];        // PACKED_SMI — fastest
// const holey = [1, , 3, , 5];           // HOLEY_SMI — slower
// const mixed = [1, 'two', 3, null, 5];  // PACKED_ELEMENTS — slowest

// Display results as a comparison table
```

---

## 🧪 DevTools Memory Profiling Exercises (1 hour)

### Exercise 1: Take a Heap Snapshot
1. Open your memory leak demo page
2. DevTools → Memory tab → "Take heap snapshot"
3. Click "Create Event Listener Leak" 10 times
4. Take another snapshot
5. Compare snapshots: Comparison view shows objects added/deleted
6. Find the leaked objects

### Exercise 2: Allocation Timeline
1. DevTools → Memory tab → "Allocation instrumentation on timeline"
2. Start recording
3. Interact with your page (create objects, trigger code)
4. Stop recording
5. Blue bars = allocated but still alive
6. Gray bars = allocated and collected (no leak)
7. Persistent blue bars = potential leak

### Exercise 3: Performance Monitor
1. DevTools → More tools → Performance Monitor
2. Watch these metrics:
   - JS heap size (should stay stable, not constantly growing)
   - DOM Nodes (should stay stable for a fixed page)
   - Event Listeners (should not grow unboundedly)
3. Interact with your page and watch for growth

---

## ❓ Interview Questions You Must Answer (2 hours)

### Basic:
1. **Explain the JavaScript event loop.**
   - Call stack, microtask queue, macrotask queue, rendering
   - Order of execution: sync → microtasks → render → macrotask

2. **What's the difference between microtasks and macrotasks?**
   - Microtasks: Promise.then, queueMicrotask, MutationObserver
   - Macrotasks: setTimeout, setInterval, setImmediate, I/O, UI rendering
   - ALL microtasks drain before next macrotask

3. **What is the difference between `setTimeout(fn, 0)` and `Promise.resolve().then(fn)`?**
   - setTimeout → macrotask queue
   - Promise.then → microtask queue
   - Promise.then runs first

4. **How does garbage collection work in JavaScript?**
   - Mark-and-sweep algorithm
   - Young generation (scavenge) vs Old generation (mark-sweep-compact)
   - Generational hypothesis: most objects die young

5. **What is a memory leak? Name 4 common causes.**
   - Forgotten event listeners
   - Closures holding large references
   - Detached DOM nodes
   - Forgotten timers/intervals
   - Accidental globals

### Intermediate:
6. **What output does this produce?**
```javascript
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2() {
  console.log('async2');
}

console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

async1();

new Promise(function(resolve) {
  console.log('promise1');
  resolve();
}).then(function() {
  console.log('promise2');
});

console.log('script end');

// Answer:
// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout
```
Explain WHY this is the order.

7. **What is `requestAnimationFrame`? How is it different from `setTimeout`?**
   - rAF syncs to display refresh rate
   - Runs before paint, not after arbitrary delay
   - Pauses when tab is hidden
   - Better for animations (no drift, no wasted frames)

8. **How would you find and fix a memory leak in a web application?**
   - Step 1: Heap snapshot comparison (before and after action)
   - Step 2: Identify retained objects
   - Step 3: Find the retaining path (what's holding the reference)
   - Step 4: Fix: remove listeners, clear timers, null references

9. **What is JIT compilation? How does V8 optimize code?**
   - Ignition (interpreter) → bytecode (fast start)
   - TurboFan (JIT compiler) → machine code (fast execution)
   - Type feedback → speculative optimization
   - Deoptimization when assumptions break

10. **What are hidden classes in V8? How do they affect performance?**
    - V8 assigns shapes (maps) to objects based on property order
    - Objects with same shape share optimized code paths
    - Changing property order/adding dynamically = new hidden class = slower

### Advanced:
11. **Explain `queueMicrotask()` vs `Promise.resolve().then()` vs `MutationObserver`.**
    - All are microtasks
    - queueMicrotask: simplest, no Promise overhead
    - Promise.resolve().then(): creates a Promise object (unnecessary if you just need microtask)
    - MutationObserver: specifically for DOM changes, batches mutations

12. **How does `async/await` work under the hood?**
    - `await` is syntactic sugar for Promise.then
    - Code after `await` goes to microtask queue
    - Async function returns a Promise
    - Explain how it desugars to generators/promises

13. **What is the "Orinoco" garbage collector?**
    - V8's concurrent/parallel GC
    - Mark phase runs on background thread (concurrent)
    - Sweep phase runs in parallel
    - Reduces GC pauses in main thread

14. **How would you optimize JavaScript execution for a performance-critical application?**
    - Keep functions monomorphic (consistent types)
    - Initialize objects with consistent shapes
    - Avoid `delete` operator
    - Use typed arrays for number-heavy computation
    - Avoid creating objects in hot loops (GC pressure)
    - Use Web Workers for CPU-intensive work

15. **What is the difference between `WeakRef`, `WeakMap`, `WeakSet`, and `FinalizationRegistry`?**
    - WeakMap/WeakSet: Keys are weakly held (GC can collect them)
    - WeakRef: Explicitly holds a weak reference to an object
    - FinalizationRegistry: Callback when an object is GC'd
    - Use cases for each

---

## ✅ Week 2 Completion Checklist

- [ ] Read all articles and watched Jake Archibald's event loop talk
- [ ] Can draw V8's compilation pipeline from memory (Parser → Ignition → TurboFan)
- [ ] Can explain the event loop with microtask/macrotask distinction
- [ ] Can predict output of complex async code (async/await + Promises + setTimeout)
- [ ] Understand hidden classes and monomorphic optimization
- [ ] Understand memory heap structure (young generation vs old generation)
- [ ] Can identify and fix 5 types of memory leaks
- [ ] Built event loop visualizer
- [ ] Used DevTools Memory tab to find leaks
- [ ] Can answer all 15 interview questions out loud

---

Next → `03-core-javascript.md`
