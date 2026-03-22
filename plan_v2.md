# Staff Frontend Engineer — 12-Month Growth Plan (v2 — Architect Reviewed)

## Current: Senior Frontend Engineer (Vue, 7 YOE, Product/Startup)
## Target: Staff Frontend Engineer (FAANG-ready)
## Time: 10 hours/week structured + 3 hours/week DSA (ongoing)
## Created: March 2026 | Revised: March 2026
## Changes from v1: Browser internals, data architecture, production ownership, AI architecture depth, continuous DSA, RADIO framework, reduced micro-frontends, deep capstone

---

## Honest Assessment

| Area | Current Level | Required for Staff @ FAANG |
|------|--------------|---------------------------|
| Framework depth | Vue only | React (mandatory) + one more |
| Browser internals | Surface level | Must reason from first principles |
| System design | No experience | Must lead architecture decisions |
| Data architecture | Not explored | Must design caching, real-time, offline strategies |
| Micro-frontends | Zero | Must evaluate tradeoffs (not necessarily implement) |
| Monorepo/tooling | Zero | Must architect build systems |
| Performance | No experience | Must optimize at scale with business metrics |
| CI/CD | No understanding | Must design pipelines |
| Production ownership | No experience | Must own observability, incidents, rollout strategy |
| RFC/ADR writing | Never written | Must write weekly |
| AI integration | Zero | Must architect AI-powered products, not just consume APIs |
| DSA | Not practicing | Must solve Medium-Hard in 25 min consistently |
| Cross-team influence | Not started | Core Staff expectation |

**The gap is significant but closable in 12 months with focused, disciplined effort.**

**Critical truth:** FAANG Staff Frontend = top ~10% of engineers. You need depth, not awareness.

---

## The Plan: 4 Phases

```
Phase 1: Foundations (Month 1-3)              — "Learn to think from first principles"
Phase 2: Architecture Deep Dive (Month 4-6)   — "Design systems at scale"
Phase 3: AI Era + Depth (Month 7-9)           — "Build Staff-level depth, not breadth"
Phase 4: Staff Behaviors + Interview (Month 10-12) — "Influence, decide, and prove it"
```

---

## DSA: Continuous Track (Runs Parallel to All Phases)

**This is non-negotiable. Start Day 1. Never stop.**

```
Month 1-6:    2 problems/week  (build the habit)           → ~48 problems
Month 7-9:    3 problems/week  (increasing difficulty)      → ~36 problems
Month 10-12:  5 problems/week  (interview intensity)        → ~60 problems
                                                    Total:    ~144 problems
```

**Weekly DSA schedule: 3 hours/week (separate from 10-hour plan)**

```
Tuesday:   45 min — 1 problem (solve + review optimal solution)
Thursday:  45 min — 1 problem (solve + review optimal solution)
Sunday:    90 min — review week's problems + 1 new problem (Month 7+)
```

**Focus areas (frontend-relevant patterns):**

| Pattern | Why it's frontend-relevant | Example problems |
|---------|---------------------------|-----------------|
| Tree traversal (BFS/DFS) | DOM is a tree. Component trees. Virtual DOM diffing. | Serialize/deserialize DOM, lowest common ancestor, invert tree |
| Hash maps | Caching, deduplication, event systems | LRU Cache, groupBy, two sum, event emitter |
| Sliding window | Virtualized lists, infinite scroll, throttling | Max in sliding window, minimum window substring |
| Stack | Undo/redo, bracket matching, expression parsing | Browser history (back/forward), valid parentheses, HTML tag parser |
| Graphs | Dependency resolution, module bundling, routing | Task scheduler, course schedule, topological sort |
| Trie | Autocomplete, search suggestions, prefix matching | Implement autocomplete, word search II |
| Dynamic Programming | Layout computation, text wrapping, optimal chunking | Word break, edit distance, coin change |
| Queue / BFS | Event loop simulation, level-order rendering | Implement task queue, serialize binary tree |

**Resources:**
- LeetCode: Focus on "Top Interview Questions" Medium + Hard
- GreatFrontEnd: Frontend-specific coding challenges
- NeetCode.io: Curated problem lists by pattern
- Implement from scratch every month: debounce, throttle, deepClone, EventEmitter, Promise.all, JSON.parse, virtual DOM diff, LRU cache

---

## Phase 1: Foundations (Month 1-3)
### Goal: React proficiency, browser-level understanding, build systems, and the mental shift from "framework user" to "platform engineer"

---

### Month 1: React + Browser Internals (40 hours)

#### Week 1-2: React Core (20 hours)

**Read (8 hours):**
- [ ] React official docs (react.dev) — read EVERY page of "Learn" section
- [ ] "Thinking in React" — understand component model vs Vue's Options/Composition API
- [ ] React mental model differences from Vue:
  - No two-way binding by default (vs v-model)
  - JSX vs templates
  - Hooks vs Composition API
  - No directive system
  - One-way data flow as a hard constraint

**Build (12 hours):**
- [ ] Rebuild a feature you've already built in Vue → in React
  - Pick something real from your work (dashboard, form, data table)
  - Use: React 19, TypeScript, Vite
  - Implement: useState, useEffect, useRef, useContext, useMemo, useCallback
  - Do NOT use any UI library — raw React only
  - Goal: Feel the differences in your fingers, not just your head

**Key Comparisons to Internalize:**
```
Vue                          →  React
ref() / reactive()           →  useState()
watch() / watchEffect()      →  useEffect()
computed()                    →  useMemo()
provide/inject               →  useContext()
v-model                      →  controlled components (value + onChange)
<template>                   →  JSX
defineProps/defineEmits       →  props + callback functions
onMounted/onUnmounted        →  useEffect with cleanup
```

#### Week 3-4: Browser Internals — The Foundation FAANG Expects (20 hours)

**Why this matters:** Every performance question, every rendering decision, every architecture choice traces back to how the browser works. Candidates who understand the platform answer from first principles. Candidates who only know frameworks memorize patterns and break under pressure.

**Read (10 hours):**
- [ ] Critical Rendering Path deep dive:
  - HTML parsing → DOM construction
  - CSS parsing → CSSOM construction
  - DOM + CSSOM → Render Tree
  - Layout (reflow) → Paint → Composite
  - How `transform` and `opacity` skip layout/paint (GPU compositing)
  - What triggers reflow vs repaint (and why this matters for React performance)
- [ ] Event Loop internals:
  - Call stack, microtask queue (Promise, queueMicrotask), macrotask queue (setTimeout, setInterval)
  - requestAnimationFrame timing (before paint, after style/layout)
  - requestIdleCallback (when the browser is idle)
  - Why React batches state updates (and how React 18+ automatic batching relates to the event loop)
- [ ] Memory model:
  - Heap vs stack allocation in JS
  - Garbage collection (mark-and-sweep, generational GC)
  - Common memory leak patterns in SPAs: detached DOM trees, forgotten event listeners, closures holding references, uncleared timers/intervals
  - WeakRef, WeakMap, FinalizationRegistry — when and why
- [ ] Networking fundamentals:
  - HTTP/2 multiplexing vs HTTP/1.1 connection limits
  - HTTP/3 QUIC — why it matters for mobile users
  - Resource hints: preload, prefetch, preconnect, dns-prefetch, modulepreload
  - 103 Early Hints — server pushing resource hints before the response
  - Service Worker lifecycle and fetch interception
- [ ] Security model:
  - Same-Origin Policy — what it actually enforces
  - CORS — how preflight works, why credentials change behavior
  - Content Security Policy (CSP) — how to architect it for a large app
  - XSS vectors and mitigation (input sanitization, CSP, trusted types)
  - CSRF — token-based vs SameSite cookie approach
  - iframe sandboxing — when and how to embed third-party content safely

**Resources:**
- web.dev "How browsers work" (Tali Garsiel)
- "Inside look at modern web browser" (4-part Google series)
- Jake Archibald: "In The Loop" (JSConf talk — event loop visualization)
- MDN: "Critical Rendering Path"
- MDN: "Memory Management"

**Build (10 hours):**
- [ ] Performance Lab — prove your understanding with measurements:
  - Write a page that deliberately triggers layout thrashing. Measure in Chrome DevTools Performance tab. Fix it. Document before/after.
  - Write code that creates a memory leak (detached DOM + closure). Find it with DevTools Memory tab (heap snapshot). Fix it. Document.
  - Build a demo showing microtask vs macrotask ordering. Predict the output before running.
  - Measure HTTP/2 multiplexing vs HTTP/1.1: load 20 images and compare waterfall charts.
  - Write a CSP policy for a sample app. Verify XSS is blocked.
- [ ] Write tradeoff document #1: "Browser rendering pipeline and what it means for frontend architecture decisions"

---

### Month 2: State Management + Practical TypeScript (40 hours)

#### Week 5-6: State Management Patterns (20 hours)

**Read (8 hours):**
- [ ] Zustand docs + source code (it's ~500 lines — READ THE SOURCE)
- [ ] Redux Toolkit docs (RTK) — you'll see this in FAANG interviews
- [ ] TanStack Query docs — server state vs client state (this distinction is critical)
- [ ] "Application State Management with React" by Kent C. Dodds
- [ ] "You Might Not Need Redux" by Dan Abramov
- [ ] Pinia docs (deepen what you know — understand the architecture, not just the API)

**Build (12 hours):**
- [ ] Project: E-commerce cart system with SAME business logic, 3 state solutions:
  1. React Context + useReducer (no library)
  2. Zustand (minimal library)
  3. Redux Toolkit + RTK Query (full solution with server state)
  - Requirements: Add to cart, remove, quantity update, persist to localStorage, optimistic updates, error rollback
  - For each: measure bundle size, count re-renders with React DevTools Profiler, measure time-to-interactive
  - Write tradeoff document #2: "Client state vs server state — when to use what, and at what team scale"

**Key Insight to Internalize:**
```
The most important state management decision is NOT "which library."
It's "what is server state vs client state?"

Server state: data that lives on the server (user profile, products, orders)
  → Use TanStack Query / SWR / RTK Query
  → Cache, invalidate, refetch. Don't duplicate in a store.

Client state: data that lives only in the browser (UI state, form state, local preferences)
  → Use useState / Zustand / Context
  → Keep it minimal. Most "state management problems" disappear
    when you stop putting server data in client stores.
```

#### Week 7-8: TypeScript for Real Architecture (20 hours)

**Not academic type gymnastics. Practical TypeScript that Staff engineers use daily.**

**Read (10 hours):**
- [ ] "Effective TypeScript" by Dan Vanderkam — Chapters 3, 4, 5, 7
- [ ] "Total TypeScript" by Matt Pocock — Essential TypeScript section
- [ ] Study real-world patterns:
  - How TanStack Query infers types from query functions
  - How tRPC achieves end-to-end type safety
  - How Zod combines runtime validation with static types

**Build (10 hours):**
- [ ] Project: Type-safe API layer + domain modeling
  - Generic fetch wrapper with type inference from endpoint definitions
  - API route definitions as TypeScript types (type-safe routing)
  - Auto-generated types from OpenAPI spec (use openapi-typescript)
  - Error handling with discriminated unions (not `any` or `unknown` casts)
  - Domain models: use branded types for IDs (`UserId` vs `ProductId` — never mix them)
  - Runtime validation with Zod, static types derived from schemas
  - Write tradeoff document #3: "TypeScript strictness levels — when to enforce what, and migration strategies for existing JS codebases"

**What matters at Staff level:**
```
NOT this (academic):              THIS (practical):
─────────────────────             ──────────────────
Conditional mapped types          Strict API contracts between teams
Template literal types            Type-safe event systems across modules
Recursive generic magic           Domain modeling with discriminated unions
How React.FC works internally     Shared type packages across 10+ consumers
                                  Migration strategy: JS → TS at scale
                                  Type-safe feature flags and config
                                  Runtime validation that matches static types
```

---

### Month 3: Build Systems + CI/CD + Production Basics (40 hours)

#### Week 9-10: Build Tools + Monorepo (20 hours)

**Read (8 hours):**
- [ ] "How does a bundler work?" — Ronen Amiel's mini-bundler tutorial
- [ ] Vite docs: "Why Vite" + architecture (dev server = esbuild, prod = Rollup)
- [ ] Webpack docs: Concepts section (mandatory for Module Federation understanding later)
- [ ] esbuild architecture: why it's fast (Go-based, parallel, minimal AST transforms)
- [ ] Turbopack architecture overview
- [ ] Turborepo docs: task dependencies, caching, remote caching, `--filter`

**Build (12 hours):**
- [ ] Project: Set up a Turborepo monorepo from scratch
  ```
  my-monorepo/
  ├── apps/
  │   ├── web/          (Next.js app)
  │   ├── docs/         (Documentation site)
  │   └── admin/        (React admin panel)
  ├── packages/
  │   ├── ui/           (Shared component library)
  │   ├── config/       (Shared ESLint, TS config)
  │   ├── utils/        (Shared utility functions)
  │   └── types/        (Shared TypeScript types)
  ├── turbo.json
  └── package.json
  ```
  - Shared UI package consumed by all 3 apps
  - Build pipeline: lint → type-check → test → build (with proper task dependencies)
  - Understand caching: when does Turborepo skip work? Verify with `--dry`
  - **The hard problem:** simulate a breaking change in `packages/ui`. How do you find all affected apps? How do you enforce that breaking changes go through a review process?
  - Write tradeoff document #4: "Monorepo vs polyrepo — when each makes sense, and the real operational cost of shared packages"

#### Week 11-12: CI/CD + Production Foundations (20 hours)

**Read (8 hours):**
- [ ] GitHub Actions docs: Workflow syntax, caching, matrix builds
- [ ] "Trunk Based Development" (trunkbaseddevelopment.com)
- [ ] Deployment strategies: Blue-green, Canary, percentage rollouts, feature flags
- [ ] LaunchDarkly architecture docs — understand how feature flags work at scale
- [ ] Sentry docs — how error monitoring and source maps work
- [ ] Read: "Monitoring and Observability for Frontend" — an introduction

**Build (12 hours):**
- [ ] Project: CI/CD pipeline + basic production tooling for your monorepo
  - GitHub Actions workflow:
    - PR checks: lint, type-check, unit tests, build (affected-only via `--filter`)
    - Preview deployments for PRs (Vercel or Netlify)
    - Production deployment on merge to main
    - Bundle size tracking (size-limit — fail PR if bundle grows beyond budget)
    - Lighthouse CI for performance regression
  - Production foundations:
    - Add Sentry error tracking to one app (understand source maps upload)
    - Add web-vitals library — log Core Web Vitals to console (later you'll send to analytics)
    - Add a basic feature flag system (even a simple JSON config — understand the pattern)
  - Write tradeoff document #5: "Deployment strategies — canary vs blue-green vs feature flags, and when to use each"

---

### Phase 1 Checkpoint — What You Should Have:
- [ ] React proficiency (can build features without looking up basic APIs)
- [ ] Browser internals knowledge (can explain rendering pipeline, event loop, memory model from memory)
- [ ] Understanding of CSR/SSR/SSG with measured tradeoffs
- [ ] 3 state management approaches compared with real code and real measurements
- [ ] TypeScript at a practical architecture level (type-safe APIs, domain modeling)
- [ ] A working monorepo with shared packages
- [ ] A CI/CD pipeline with bundle tracking and preview deploys
- [ ] Basic production tooling (error monitoring, web vitals, feature flags)
- [ ] 5 tradeoff documents written
- [ ] ~120 structured hours invested + ~24 DSA hours (~48 problems solved)

---

## Phase 2: Architecture Deep Dive (Month 4-6)
### Goal: System design mastery, data architecture, design systems, performance engineering

---

### Month 4: Frontend System Design (40 hours)

#### Week 13-14: System Design Fundamentals + RADIO Framework (20 hours)

**Read (10 hours):**
- [ ] "Frontend System Design" — GreatFrontEnd (https://www.greatfrontend.com/system-design)
- [ ] "System Design for Frontend Engineers" by Evgenii Ray
- [ ] "Design Docs at Google" — how Google writes design documents
- [ ] "Scaling Frontend Architecture" series by Malte Ubl (ex-Google)
- [ ] Case studies:
  - How Netflix builds frontend (Netflix Tech Blog)
  - How Meta builds frontend (React + Relay architecture)
  - How Airbnb moved to React and built their design system
  - How Shopify's frontend architecture evolved

**Master the RADIO Framework:**
```
Every system design answer follows this structure:

R — Requirements Exploration (5 min)
    Don't jump to solutions. Ask:
    ├── Who are the users? How many concurrent?
    ├── What devices/browsers must we support?
    ├── What's the latency budget? (p50, p95, p99)
    ├── What's the data shape and volume?
    ├── What are the offline requirements?
    ├── What's the team structure? (this drives architecture choices)
    └── What's the MVP vs future scope?

A — Architecture / High-Level Design (10 min)
    ├── Component tree (draw it)
    ├── Data flow direction (draw it)
    ├── Rendering strategy choice (CSR/SSR/SSG and WHY)
    └── Key boundaries (what's a page, what's a component, what's shared)

D — Data Model (10 min) ← WHERE MOST CANDIDATES FAIL
    ├── Client-side data model (entity shapes, normalized vs nested)
    ├── API contract (REST? GraphQL? WebSocket? WHY?)
    ├── Caching strategy (what's cached, how long, invalidation triggers)
    ├── Optimistic vs pessimistic updates (which actions, why)
    └── Server state vs client state separation

I — Interface Definition (5 min)
    ├── Props, events, slots for key components
    ├── How do components compose?
    ├── What's the public API boundary?
    └── How does state flow between components?

O — Optimizations & Deep Dives (10 min)
    ├── Performance (bundle splitting, virtualization, lazy loading)
    ├── Accessibility (keyboard, screen readers, focus management)
    ├── Error handling & edge cases (offline, slow network, stale data)
    ├── Monitoring & observability (how do you know it's working?)
    └── Internationalization (if applicable at scale)
```

**Practice (10 hours):**
- [ ] Design these systems (write full design documents using RADIO, do NOT code):
  1. **Real-time collaborative document editor (like Google Docs)**
     - CRDT vs OT for conflict resolution
     - Cursor presence and awareness
     - WebSocket connection management and reconnection
     - Performance at 100 concurrent editors
  2. **Social media feed (like Twitter/X)**
     - Infinite scroll with virtualization
     - Optimistic like/retweet with rollback
     - Real-time new post indicators
     - Image lazy loading and placeholder strategy
     - Caching and pagination (cursor-based vs offset)
  3. **Analytics dashboard (like Grafana)**
     - Large dataset rendering with virtualization
     - Real-time chart updates via WebSocket/SSE
     - URL-driven state (shareable dashboard configs)
     - Date range picker and filter architecture
     - Data aggregation: server-side vs client-side

#### Week 15-16: Design System Architecture (20 hours)

**Read (10 hours):**
- [ ] "Design Systems" by Alla Kholmatova (book)
- [ ] Radix UI source code — study the "headless UI" pattern
- [ ] Shadcn/ui architecture — study the "copy-paste" distribution model
- [ ] Material UI architecture — study the "full library" approach
- [ ] W3C "Design Tokens" specification
- [ ] Read: How companies solve the REAL design system problems:
  - Versioning strategy (semver vs lockstep)
  - Adoption (how do you get 8 teams to actually use it?)
  - Theming (white-labeling for enterprise customers)
  - Breaking changes across multiple consumers

**Build (10 hours):**
- [ ] Project: Build a mini design system in your monorepo
  - 5 components: Button, Input, Select, Modal, Toast
  - Requirements:
    - TypeScript strict mode
    - Design tokens as CSS custom properties (map to a token system, not raw hex)
    - Compound component pattern (`<Select><Select.Option>`)
    - Polymorphic `as` prop (Button renders as `<a>` or `<button>`)
    - Full keyboard accessibility (WAI-ARIA)
    - Storybook for documentation
    - Unit tests with Testing Library
  - **Architectural decisions to document:**
    - Controlled vs uncontrolled: which components, why?
    - Headless vs styled: why this choice for this team size?
    - CSS approach: custom properties vs CSS-in-JS vs Tailwind? Why?
    - Versioning: how would you handle a breaking Button API change when 4 apps consume it?
    - Adoption: how would you migrate an existing app to this design system incrementally?
  - Write ADR #6: "Design System Architecture Decisions"

---

### Month 5: Data Architecture + Micro-Frontends Evaluation (40 hours)

**This is the most underrated area of frontend architecture. Staff engineers at FAANG spend more time on data layer decisions than component architecture.**

#### Week 17-18: Data Fetching Architecture (20 hours)

**Read (10 hours):**
- [ ] TanStack Query docs (deep dive): cache invalidation, optimistic updates, prefetching, infinite queries, mutation lifecycle, offline support
- [ ] Relay documentation (Meta's data fetching framework): understand fragment colocation, data masking, normalized cache
- [ ] Apollo Client architecture: normalized cache, cache policies, local state management
- [ ] Read: "GraphQL at scale" — how Meta uses Relay, how Shopify uses Apollo
- [ ] Read: "REST vs GraphQL" — not which is "better" but when each is appropriate
- [ ] tRPC docs — end-to-end type safety without code generation
- [ ] Read: "Backend-for-Frontend (BFF) pattern" — when to add an API aggregation layer

**Build (10 hours):**
- [ ] Project: Data layer comparison
  - Take your e-commerce app from Month 2 and add:
    1. TanStack Query integration (REST backend)
       - Implement: query invalidation on mutation, optimistic cart updates, prefetch on hover, stale-while-revalidate
    2. GraphQL + Apollo Client version (same UI, different data layer)
       - Implement: normalized cache, fragment colocation, optimistic responses
    3. Real-time price updates via Server-Sent Events (SSE)
       - Implement: reconnection strategy, stale data indicators, merge with cached data
  - Measure: network requests, cache hit rates, time-to-data after navigation
  - Write tradeoff document #7: "REST + TanStack Query vs GraphQL + Apollo vs tRPC — decision framework based on team size, API ownership, and data complexity"

#### Week 19-20: Real-Time + Offline Architecture & Micro-Frontends Evaluation (20 hours)

**Real-Time & Offline (10 hours):**

**Read (5 hours):**
- [ ] WebSocket architecture: connection management, heartbeat, reconnection with exponential backoff
- [ ] Server-Sent Events vs WebSocket vs Long Polling — when each is appropriate
- [ ] IndexedDB for offline storage: Dexie.js as a practical wrapper
- [ ] Read: "Offline-first web applications" — sync strategies and conflict resolution
- [ ] Read: CRDTs at a conceptual level (Conflict-free Replicated Data Types) — understand what they solve, not the math

**Build (5 hours):**
- [ ] Add to your project: offline-capable cart with sync
  - Save cart to IndexedDB when offline
  - Queue mutations (add/remove/update) while offline
  - Sync queued mutations when back online
  - Handle conflicts (item out of stock while user was offline)
  - Show sync status to user (synced / pending / conflict)
  - Write brief tradeoff note: "Offline strategy — last-write-wins vs conflict resolution UI"

**Micro-Frontends Evaluation (10 hours):**

**Read (6 hours):**
- [ ] "Building Micro-Frontends" by Luca Mezzalira — read for concepts, not to implement everything
- [ ] Webpack Module Federation docs + architecture diagram
- [ ] "When NOT to use micro-frontends" (Martin Fowler's blog)
- [ ] Read: Post-mortems from companies that moved away from MFE
- [ ] Read: "Modular Monolith" as the alternative pattern
- [ ] Study the real-world landscape:
  ```
  Company     | Approach              | Why
  ────────────┼───────────────────────┼──────────────────────────
  Meta        | Monorepo, not MFE     | Single team culture, shared infra
  Google      | Monorepo, not MFE     | Single massive repo (Piper)
  Amazon      | MFE (some teams)      | Extreme team autonomy required
  Netflix     | Moved away from MFE   | Coordination cost exceeded benefit
  Shopify     | Modular monolith      | Strong module boundaries, single deploy
  IKEA        | MFE with Module Fed   | Legacy migration, many vendors
  ```

**Build (4 hours):**
- [ ] Write a comprehensive decision framework (NOT a demo app):
  ```
  # Micro-Frontend Decision Framework

  ## Use micro-frontends when ALL of these are true:
  - Multiple autonomous teams (>3 teams, >15 engineers)
  - Different release cadences are genuinely needed
  - Legacy migration (strangler fig pattern)
  - Teams need different tech stacks (rare, but real)

  ## Use modular monolith when:
  - Same tech stack across teams
  - Shared design system and patterns
  - < 15 frontend engineers
  - Deployment coordination is acceptable

  ## Composition approaches ranked by complexity:
  1. npm packages (build-time) — simplest, best performance
  2. Module Federation (runtime) — independent deploy, shared deps
  3. Single-SPA (runtime) — framework-agnostic, higher complexity
  4. iframes — complete isolation, worst performance, last resort

  ## The question to ask: "Is the organizational problem bad enough
     to justify the technical complexity of micro-frontends?"
     Usually the answer is: modular monolith with strong boundaries.
  ```
- [ ] Write tradeoff document #8: "Micro-frontends vs Modular Monolith — a decision framework for different team structures"

---

### Month 6: Performance Engineering (40 hours)

#### Week 21-22: Performance Measurement + Optimization (20 hours)

**Read (10 hours):**
- [ ] web.dev "Performance" section — ALL articles on Core Web Vitals
- [ ] "The Cost of JavaScript" by Addy Osmani
- [ ] Chrome DevTools Performance documentation (deep dive, not tutorial level)
- [ ] Core Web Vitals in depth:
  - LCP: what affects it, critical rendering path connection, server vs client factors
  - INP: replaces FID, measures full interaction responsiveness, event handlers + rendering time
  - CLS: layout shift causes, font-display impact, dynamic content injection patterns
- [ ] "React Performance" by Ivan Akulov
- [ ] Read: "Virtual DOM is pure overhead" by Rich Harris — understand the tradeoff, not the clickbait
- [ ] **The business case for performance:**
  - Study: Google/Amazon/Walmart studies linking LCP to conversion rates
  - Understand: how to present performance work to non-technical stakeholders
  - "LCP improved by 200ms → conversion up by X%" is how Staff engineers justify performance work

**Build (10 hours):**
- [ ] Project: Performance audit and optimization of your monorepo apps
  - Set up performance monitoring stack:
    - Lighthouse CI in pipeline (fail builds on regression)
    - Bundle analyzer (vite-bundle-analyzer)
    - web-vitals library sending to a dashboard (or console for now)
    - Performance budgets: define max bundle size per route
  - Implement optimizations:
    - Code splitting with React.lazy + Suspense (route-based)
    - Image optimization (next/image or manual with srcset, WebP/AVIF, loading="lazy")
    - Font optimization (font-display: swap, preload critical font, subset)
    - Tree shaking verification (import analysis, side-effect-free marking)
  - **Document before/after metrics** (this is what you show in interviews)

#### Week 23-24: Advanced Performance Patterns (20 hours)

**Read (8 hours):**
- [ ] Virtualization: study react-window and react-virtuoso source code
- [ ] Web Workers: Comlink library for easy communication
- [ ] Service Workers: Workbox documentation, caching strategies (cache-first, network-first, stale-while-revalidate)
- [ ] Streaming SSR: how React 19 + Next.js stream HTML to the browser
- [ ] Islands Architecture: Astro's approach (partial hydration)
- [ ] Read: "Resumability" — Qwik framework's approach (understand the concept)
- [ ] RUM vs Synthetic monitoring — why Lighthouse scores don't tell the full story

**Build (12 hours):**
- [ ] Project: Data-heavy dashboard with advanced performance patterns
  - 10,000+ row table with virtualization (react-virtuoso or react-window)
  - Real-time chart updates via WebSocket
  - Web Worker for data transformation (sort/filter/aggregate on background thread)
  - Service Worker for offline caching of dashboard configuration
  - Measure: memory usage (DevTools heap snapshot), frame rate (DevTools FPS meter), INP score
  - Performance budget enforcement: automated alerts when thresholds are crossed
  - Write tradeoff document #9: "Performance optimization priority framework — where to invest effort for maximum user impact"

---

### Phase 2 Checkpoint — What You Should Have:
- [ ] 3 frontend system design documents (editor, feed, dashboard) using RADIO framework
- [ ] A working design system with documented architectural decisions
- [ ] Deep understanding of data fetching patterns (TanStack Query, GraphQL, real-time, offline)
- [ ] A micro-frontend decision framework (not just a demo)
- [ ] Performance optimization experience with measured before/after results
- [ ] Understanding of the business case for performance
- [ ] ~240 structured hours invested + ~48 DSA hours (~96 problems solved)
- [ ] 9 tradeoff/design documents written

---

## Phase 3: AI Era + Depth (Month 7-9)
### Goal: AI as an architectural concern, production ownership, testing architecture, and a deep capstone

---

### Month 7: AI Architecture for Frontend (40 hours)

**AI is not "add a chatbot." In 2026, AI is an architectural concern that affects data flow, UX patterns, cost structure, and infrastructure decisions.**

#### Week 25-26: AI Fundamentals + AI-Native UX (20 hours)

**Read (10 hours):**
- [ ] LLM fundamentals: tokens, context windows, temperature, top-p, streaming, function calling
- [ ] OpenAI API docs: Chat Completions, Streaming, Function Calling, Structured Outputs
- [ ] Anthropic API docs: Messages API, Streaming, Tool Use
- [ ] Vercel AI SDK documentation (the standard for AI + frontend in 2026)
- [ ] Read: "AI UX patterns beyond chat":
  - Inline suggestions (Copilot-style, autocomplete-on-steroids)
  - AI-assisted forms (smart defaults, validation suggestions, field auto-fill)
  - Semantic search UIs (query understanding, AI-ranked results, citation rendering)
  - Human-in-the-loop workflows (AI proposes, human approves/edits)
  - Progressive disclosure of AI confidence (when to show uncertainty)
  - Graceful degradation (what happens when AI is slow, down, or wrong)
- [ ] Read: "Prompt Engineering Guide" — basic patterns and anti-patterns
- [ ] Study: How GitHub Copilot, Cursor, and Notion AI architect their UX

**Build (10 hours):**
- [ ] Project: AI-powered chat interface (the baseline)
  - React + Next.js + Vercel AI SDK
  - Features:
    - Streaming responses (show tokens as they arrive)
    - Markdown rendering with code syntax highlighting
    - Conversation history management (context window awareness)
    - Token usage display and cost estimation
    - Error handling, retry logic, abort capability
    - Loading states (skeleton vs spinner vs progressive content)
  - Then go beyond chat:
    - Add: inline AI suggestions in a text input (autocomplete-style, not chat)
    - Add: AI-powered search with citation rendering
  - This demonstrates you understand AI UX is more than a chatbot

#### Week 27-28: AI Architecture + System Design (20 hours)

**Read (8 hours):**
- [ ] AI architecture decisions Staff engineers make:
  - Where does inference run? Cloud API vs Edge (Cloudflare Workers AI) vs Browser (WebLLM, ONNX Runtime Web)
  - Latency budget: streaming start time (time to first token), token rate, total response time
  - Cost architecture: token budgets per user/session, caching strategies, model tiering (GPT-4 for complex, GPT-3.5 for simple)
  - Prompt management: versioning prompts, A/B testing prompts, evaluating prompt quality
  - Context window management: what goes in, what gets truncated, summarization strategies
  - Safety and guardrails: content filtering, PII detection, hallucination mitigation in UI
- [ ] Study: RAG (Retrieval-Augmented Generation) from the frontend perspective
  - How to render search results with AI-generated summaries
  - Citation UI patterns (link to source, highlight relevant passage)
  - Chunking display (showing which documents the AI used)
- [ ] Read: "Function Calling / Tool Use" — structured output from LLMs for UI rendering
- [ ] Read: Edge inference — running models at the edge for latency and cost optimization

**Build (12 hours):**
- [ ] Project: AI-powered code review tool (add to your monorepo)
  - Upload/paste code → AI analyzes for performance, accessibility, and best practice issues
  - Streaming response with structured sections (uses Function Calling, not free text)
  - Caching: same code hash → same review (don't re-call the API)
  - Rate limiting UI + cost estimation display
  - Prompt versioning: store prompts as versioned configs, easy to A/B test
  - Quality evaluation: track user feedback (helpful / not helpful) on AI responses
  - Write ADR #10: "AI Integration Architecture"
    - API key management (server-side only, never client-side)
    - Streaming vs batch responses: when each is appropriate
    - Cost management: caching, model tiering, token budgets
    - Error handling: timeout, rate limit, content filter, model unavailable
    - Evaluation: how do we know the AI feature is actually good?

**AI-Era System Design Practice (write design docs, don't code):**
- [ ] Design an AI-powered search experience (like Perplexity)
  - Source retrieval → AI synthesis → citation rendering
  - Streaming answer with progressive source loading
  - Follow-up questions with context retention
- [ ] Design a copilot sidebar for a SaaS product (like Notion AI)
  - Context awareness (what page/section is the user on?)
  - Multi-action capability (summarize, rewrite, translate, generate)
  - Inline insertion vs sidebar interaction
  - Undo/rollback after AI modification

---

### Month 8: Production Ownership + Testing + Accessibility (40 hours)

#### Week 29-30: Production Ownership — What Happens After Code Ships (20 hours)

**This is the single biggest gap most Senior-to-Staff candidates have. Staff engineers own production reliability.**

**Read (8 hours):**
- [ ] Sentry docs (deep dive): error grouping, release tracking, source maps, session replay
- [ ] Datadog RUM (or equivalent): real user monitoring architecture
- [ ] Read: "Frontend Observability" — what to monitor, what to alert on, what to log
- [ ] Read: "Feature Flags at Scale" — LaunchDarkly / Statsig architecture
  - Percentage rollouts, user segmentation, kill switches
  - How to design experiments (A/B testing) at the frontend layer
  - Flag evaluation: client-side vs server-side vs edge
- [ ] Read: "SLOs and SLIs for Frontend"
  - Example: "p95 LCP < 2.5s for 99.9% of users"
  - How to define, measure, and alert on frontend SLOs
- [ ] Read: "Incident Response for Frontend"
  - How to diagnose production issues with source maps
  - Session replay for bug reproduction
  - War room process: what's the frontend engineer's role?
  - Postmortem writing: blameless, actionable, preventive

**Build (12 hours):**
- [ ] Add production infrastructure to your monorepo app:
  - **Error monitoring:** Sentry integration with:
    - Custom error boundaries (catch errors per route/feature, not just globally)
    - Source map upload in CI/CD (errors show original code, not minified)
    - Release tracking (correlate errors with deployments)
    - User context (know which user hit the error)
  - **Feature flags:** Implement a feature flag system:
    - Server-evaluated flags for SSR pages
    - Client SDK for dynamic UI toggles
    - Percentage rollout capability
    - Kill switch for emergency disable
    - Integrate with CI: different flags for preview vs production
  - **Monitoring dashboard:** Set up basic observability:
    - Core Web Vitals tracking (web-vitals → analytics)
    - Error rate per route
    - Feature flag adoption metrics
    - Deployment success/rollback tracking
  - **Incident simulation:** Deliberately break something in production (staging). Practice:
    - Detect via monitoring
    - Diagnose via error tracking + source maps
    - Mitigate via feature flag kill switch
    - Write a mini postmortem
  - Write ADR #11: "Production Observability Strategy"

#### Week 31-32: Testing Architecture + Accessibility (20 hours)

**Testing (10 hours):**

**Read (4 hours):**
- [ ] "Testing Trophy" by Kent C. Dodds
- [ ] Vitest docs + Testing Library philosophy ("test like a user")
- [ ] Playwright docs — E2E testing
- [ ] Read: "Visual Regression Testing" — Chromatic, Percy, or Playwright screenshots
- [ ] Read: "Testing in production" — canary analysis, synthetic monitoring

**Build (6 hours):**
- [ ] Add testing architecture to your monorepo:
  - Unit tests: Vitest + Testing Library for design system components
  - Integration tests: test component interactions with mocked API (MSW)
  - E2E tests: Playwright for 3 critical user flows
  - Visual regression: Playwright screenshot comparison for design system
  - Test in CI: run on PRs, block merge on failure
  - Coverage thresholds: set minimum but DON'T chase 100%
  - Write brief ADR section: "Testing Strategy — what to test at each level, and why 100% coverage is not the goal"

**Accessibility (10 hours):**

**Read (4 hours):**
- [ ] WAI-ARIA Authoring Practices 1.2 — study component patterns (dialog, menu, tabs, combobox)
- [ ] "Inclusive Components" by Heydon Pickering
- [ ] WCAG 2.2 AA guidelines — understand success criteria
- [ ] Read: "Accessibility at scale" — how FAANG companies enforce a11y

**Build (6 hours):**
- [ ] Audit and fix your design system for accessibility:
  - axe-core automated checks on every component
  - Keyboard navigation testing (Tab, Escape, Arrow keys, Enter)
  - Screen reader testing with VoiceOver (Mac)
  - Focus management: Modal traps focus, restores on close
  - Toast: announced to screen readers via aria-live
  - Add axe-core to CI (fail builds on a11y violations)
  - Write document: "Accessibility Standards for Our Design System"

---

### Month 9: Capstone Project — Go Deep, Not Wide (40 hours)

#### Week 33-36: Build ONE Staff-Level Project With Depth (40 hours)

**Choose ONE of these capstone tracks. Going deep on one problem demonstrates Staff-level thinking. Going wide across everything demonstrates you followed a plan.**

---

**Option A: Real-Time Collaborative Dashboard**
**(Recommended if you want to show data architecture + performance depth)**

```
capstone-dashboard/
├── apps/
│   ├── web/                    (Next.js — the dashboard application)
│   └── docs/                   (Architecture documentation site)
├── packages/
│   ├── ui/                     (Design system components)
│   ├── data-layer/             (TanStack Query + WebSocket + offline sync)
│   ├── chart-engine/           (Virtualized chart rendering)
│   └── config/                 (Shared configs)
├── infrastructure/
│   ├── .github/workflows/      (CI/CD)
│   └── monitoring/             (Sentry config, web-vitals setup)
├── docs/
│   ├── system-design.md        (Full RADIO system design document)
│   ├── adrs/                   (Architecture Decision Records)
│   └── performance-report.md   (Before/after optimization metrics)
└── turbo.json
```

**Requirements:**
- [ ] Full system design document using RADIO framework
- [ ] Real-time data via WebSocket with reconnection and stale indicators
- [ ] 10,000+ row virtualized table with sort/filter/search
- [ ] Chart rendering with real-time updates (no layout shift)
- [ ] Offline support: cache dashboard config, queue filter changes
- [ ] Optimistic updates with rollback on server rejection
- [ ] Performance: LCP < 1.5s, INP < 100ms, CLS < 0.05 (measured and documented)
- [ ] Accessible: keyboard navigable, screen reader tested
- [ ] Error monitoring via Sentry with custom error boundaries per widget
- [ ] Feature flags: toggle between chart types, new widget rollout
- [ ] CI/CD: preview deploys, Lighthouse CI, bundle budget, a11y checks
- [ ] 5+ Architecture Decision Records explaining WHY, not just WHAT

---

**Option B: AI-Powered Developer Tool**
**(Recommended if you want to show AI architecture depth)**

```
capstone-ai-tool/
├── apps/
│   ├── web/                    (Next.js — the AI tool)
│   └── docs/                   (Architecture documentation site)
├── packages/
│   ├── ui/                     (Design system components)
│   ├── ai-sdk/                 (AI integration: streaming, caching, prompt management)
│   ├── editor/                 (Code editor with AI inline suggestions)
│   └── config/                 (Shared configs)
├── infrastructure/
│   ├── .github/workflows/      (CI/CD)
│   └── monitoring/             (Sentry, web-vitals, AI quality metrics)
├── docs/
│   ├── system-design.md        (Full RADIO system design document)
│   ├── adrs/                   (Architecture Decision Records)
│   └── ai-evaluation.md        (How we measure AI quality)
└── turbo.json
```

**Requirements:**
- [ ] Full system design document using RADIO framework
- [ ] Code editor with AI inline suggestions (not just a chat box)
- [ ] Streaming responses with structured output (Function Calling)
- [ ] Prompt versioning system (versioned configs, easy to swap)
- [ ] Caching layer: identical inputs → cached results (save cost + latency)
- [ ] Cost tracking: token usage per session, budget enforcement
- [ ] Quality evaluation: user feedback collection, quality dashboard
- [ ] Graceful degradation: AI slow → show cached/fallback, AI down → degrade to manual
- [ ] Error monitoring with AI-specific error types (timeout, rate limit, content filter)
- [ ] Feature flags: A/B test different prompts/models
- [ ] CI/CD: preview deploys, bundle budget, a11y checks
- [ ] 5+ Architecture Decision Records

---

### Phase 3 Checkpoint — What You Should Have:
- [ ] AI integration at an architectural level (not just API consumption)
- [ ] AI-era system design documents (AI search, copilot sidebar)
- [ ] Production ownership experience (monitoring, feature flags, incident response)
- [ ] Complete testing strategy implemented
- [ ] Accessibility expertise with automated enforcement
- [ ] A capstone project that goes DEEP on one problem, demonstrating Staff-level thinking
- [ ] ~360 structured hours invested + ~72 DSA hours (~132 problems solved)
- [ ] 12+ architecture/tradeoff documents

---

## Phase 4: Staff Behaviors + Interview Prep (Month 10-12)
### Goal: Think, communicate, and influence like a Staff Engineer. Prove it in interviews.

---

### Month 10: Communication + Influence (40 hours)

#### Week 37-38: Technical Writing + Staff Mindset (20 hours)

**Read (8 hours):**
- [ ] "Staff Engineer" by Will Larson (REQUIRED — the foundational book)
- [ ] "The Staff Engineer's Path" by Tanya Reilly (REQUIRED — the practical guide)
- [ ] "An Elegant Puzzle" by Will Larson (skim for tech leadership insights)
- [ ] Read: "How to write an RFC" — examples from Rust, React, and Ember communities
- [ ] Read: "Writing Technical Design Docs" by Malte Ubl

**Practice (12 hours):**
- [ ] Write 3 RFCs for your current company (even if not asked to — THIS IS STAFF BEHAVIOR):
  1. RFC: "Migrating from [current build tool] to Vite"
  2. RFC: "Introducing a shared design system"
  3. RFC: "Frontend observability and monitoring strategy"
  - Follow this structure:
    ```
    # RFC: [Title]
    ## Status: Draft
    ## Author: [You]
    ## Date: [Date]
    ## Summary (2-3 sentences — what and why)
    ## Motivation (Why now? What problem? What's the cost of doing nothing?)
    ## Detailed Design (How it works — architecture, data flow, component design)
    ## Alternatives Considered (What else did you evaluate? Why not those?)
    ## Migration Strategy (How do we get there from here? What's the sequence?)
    ## Risks and Mitigations (What could go wrong? How do we handle it?)
    ## Open Questions (What do we need input on?)
    ```
  - **Share with your team for feedback.** Getting feedback on RFCs is a core Staff activity.

**The Staff Mindset Shift:**
```
Senior thinks:          Staff thinks:
─────────────────       ─────────────────────────────────
"How do I build this?"  "Should we build this at all?"
"What's the best tech?" "What's the best tech for THIS team?"
"I'll implement it."    "I'll scope it so 3 people can implement it."
"Here's my PR."         "Here's my design doc. Let's align before coding."
"I finished the task."  "I finished the task and documented why we made each decision."
"This code is slow."    "This page has 2.8s LCP affecting 40% of users.
                         Here's the fix, projected impact, and rollout plan."
```

#### Week 39-40: Mentorship + Cross-Team Influence (20 hours)

**Actions (these are behaviors, not coding tasks):**
- [ ] Start a "Frontend Architecture" discussion group at your company (biweekly, 30 min)
  - Month 10: "How micro-frontends work and when (not) to use them"
  - Month 11: "Frontend performance — what actually matters and how to measure it"
  - Month 12: "AI integration patterns — what we should build and what we shouldn't"
- [ ] Code review with architectural feedback (not just style nits):
  - "Have you considered the performance impact of this approach at 10x our current data?"
  - "This component API doesn't compose well because... here's an alternative"
  - "Let's document this decision in an ADR because future engineers will ask why"
  - "This is the right solution for now, but let's add a TODO for when we hit [threshold]"
- [ ] Mentor 1-2 junior/mid engineers — pair program weekly
- [ ] Start writing technical blog posts (1-2 is enough):
  - "What I learned building a real-time dashboard from scratch"
  - "AI architecture patterns for frontend engineers"
- [ ] **Practice navigating ambiguity**: take a vague problem at work ("our frontend is slow" / "we need to modernize") and turn it into a scoped, sequenced plan. This is THE defining Staff skill.

---

### Month 11: System Design Interview Prep (40 hours)

#### Week 41-42: System Design Deep Practice (20 hours)

**Study (8 hours):**
- [ ] GreatFrontEnd system design questions — practice ALL of them
- [ ] Watch: "Frontend System Design" mock interviews on YouTube (study the framework, not memorize answers)
- [ ] Practice drawing architecture diagrams (Excalidraw)
- [ ] Review your RADIO framework notes and all previous design documents

**Practice (12 hours) — 45 min each, strictly timed:**

Classic problems (still asked):
- [ ] Design a Gmail-like email client
- [ ] Design a Google Maps-like application
- [ ] Design a Spotify-like music player
- [ ] Design a Figma-like collaborative whiteboard
- [ ] Design a Notion-like block editor
- [ ] Design an auto-complete/search component at scale
- [ ] Design a news feed with infinite scroll
- [ ] Design a real-time polling/voting system

Modern problems (increasingly asked in 2026):
- [ ] Design an AI chatbot with tool use and multi-turn context (like ChatGPT)
- [ ] Design an AI-powered search experience (like Perplexity)
- [ ] Design a video conferencing UI (like Zoom/Google Meet)
- [ ] Design a feature flag management dashboard
- [ ] Design a notification system (multi-channel: in-app, push, email)
- [ ] Design a drag-and-drop dashboard builder
- [ ] Design an e-commerce product page at scale (like Amazon)

**For each: write component tree, data model, API design, state management, performance strategy, and monitoring approach. Time yourself — in interviews you get 35-45 minutes.**

#### Week 43-44: Coding Interview Prep — Frontend Specific (20 hours)

**JavaScript fundamentals (tested at every FAANG):**
- [ ] Closures, prototypes, `this` binding, event loop
- [ ] Promises: implement Promise.all, Promise.race, Promise.allSettled from scratch
- [ ] Implement: debounce, throttle, deepClone, EventEmitter, curry, memoize
- [ ] DOM manipulation without frameworks (createElement, event delegation, virtual scroll)

**Build UI components from scratch (common Staff-level question format):**
- [ ] Autocomplete with debounced search, keyboard navigation, highlighting
- [ ] Modal with focus trapping, escape to close, portal rendering
- [ ] Virtual scrolling list (render only visible items)
- [ ] Drag and drop list reordering
- [ ] Infinite scroll with intersection observer
- [ ] Carousel with touch/swipe support

**LeetCode (continue 5 problems/week from DSA track):**
- Focus: Medium-Hard problems from these categories:
  - Trees and graphs (DOM-related thinking)
  - Hash maps (caching, indexing)
  - Sliding window (streaming data)
  - Stacks (undo/redo, parsing)
  - Dynamic programming (layout, optimization)

---

### Month 12: Mock Interviews + Final Preparation (40 hours)

#### Week 45-46: Mock Interviews (20 hours)

- [ ] Do 6-8 mock interviews:
  - 2 system design rounds (use RADIO, practice drawing live)
  - 2 coding rounds (JavaScript/React, build UI components from scratch)
  - 2 behavioral rounds (Staff-level: "Tell me about a time you influenced a technical decision across teams")
  - 1 architecture deep dive (deep dive on a system you've built — prepare your capstone story)
  - 1 AI/modern tech round (prepare to discuss AI architecture decisions)
- [ ] Platforms: Pramp (free), interviewing.io, or find peers on Blind/Discord
- [ ] After each mock: write down what went well, what didn't, specific improvements

**Behavioral prep for Staff level:**
```
Prepare stories (STAR format) for:

1. "Tell me about a technical decision you made that affected multiple teams."
2. "Describe a time you said NO to a popular technical choice. Why?"
3. "How did you handle disagreement on a technical approach?"
4. "Tell me about a time you identified a problem nobody asked you to solve."
5. "How do you decide between building vs buying?"
6. "Describe how you mentored someone and what they achieved."
7. "Tell me about a production incident you led resolution for."
8. "How do you prioritize technical debt vs features?"
```

#### Week 47-48: Polish + Apply (20 hours)

- [ ] Update your resume for Staff level:
  - Lead with IMPACT, not tasks
  - Bad: "Built a design system using React"
  - Good: "Architected a shared design system adopted by 4 teams (15 engineers), reducing UI development time by 40% and eliminating 200+ duplicate components"
  - Bad: "Added AI chatbot feature"
  - Good: "Designed AI integration architecture serving 50K daily queries with streaming responses, prompt caching reducing API costs by 60%, and graceful degradation achieving 99.9% availability"
- [ ] Update LinkedIn with architecture-focused content
- [ ] Polish your capstone project:
  - README must be excellent (problem statement, architecture diagram, decisions made, results)
  - System design doc must be comprehensive
  - ADRs must be readable by someone outside your context
- [ ] Publish 1-2 blog posts about your architectural learnings
- [ ] Start applying:
  - Tier 1: Meta, Google, Apple, Netflix, Stripe
  - Tier 2: Vercel, Shopify, Airbnb, Uber, DoorDash
  - Tier 3: Well-funded growth-stage startups (Staff at a 200-person startup is legit)

---

## Final Checkpoint — What You Should Have After 12 Months:

### Portfolio Artifacts:
- [ ] 1 deep capstone project (demonstrates architectural depth, not breadth)
- [ ] 15+ Architecture Decision Records / Tradeoff documents
- [ ] 3 RFCs written and shared with your team
- [ ] 5+ system design practice documents (classic + AI-era)
- [ ] 2+ AI-era system design documents
- [ ] 1-2 published blog posts
- [ ] Performance optimization report with measured before/after metrics
- [ ] Production observability documentation

### Technical Skills:
- [ ] React proficiency (on par with Vue)
- [ ] Browser internals (rendering pipeline, event loop, memory model, security, networking)
- [ ] Next.js (SSR, SSG, ISR, App Router, streaming)
- [ ] TypeScript at a practical architecture level (type-safe APIs, domain modeling, migration strategies)
- [ ] Data architecture (TanStack Query, GraphQL, real-time, offline-first, caching strategies)
- [ ] Micro-frontend evaluation capability (decision framework, not just implementation)
- [ ] Design system architecture (implementation + versioning + adoption strategy)
- [ ] Frontend performance optimization with business-impact metrics
- [ ] AI architecture (streaming, prompt management, cost optimization, evaluation, edge inference)
- [ ] Production ownership (monitoring, feature flags, incident response, SLOs)
- [ ] CI/CD pipeline design with performance budgets and a11y enforcement
- [ ] Testing architecture at all levels (unit, integration, E2E, visual regression)
- [ ] Accessibility as a first-class architectural concern
- [ ] ~144 DSA problems solved (ready for FAANG coding rounds)

### Staff Engineer Behaviors:
- [ ] Technical writing (RFCs, ADRs, design docs — can write one in your sleep)
- [ ] Cross-team influence and communication (led discussions, presented to teams)
- [ ] Mentorship of junior/mid engineers (ongoing relationships)
- [ ] Architectural decision-making with documented tradeoffs
- [ ] Navigating ambiguity (turning vague problems into scoped plans)
- [ ] Production ownership mindset (you own it until it works in production)

---

## Complete Reading List (Priority Order)

### Books (MUST READ — Non-negotiable):
1. "Staff Engineer" — Will Larson
2. "The Staff Engineer's Path" — Tanya Reilly
3. "Effective TypeScript" — Dan Vanderkam
4. "Design Systems" — Alla Kholmatova

### Books (HIGH VALUE):
5. "Building Micro-Frontends" — Luca Mezzalira (read for concepts, not to implement everything)
6. "Web Performance in Action" — Jeremy Wagner
7. "Inclusive Components" — Heydon Pickering
8. "System Design for Frontend Engineers" — Evgenii Ray

### Books (SKIM / REFERENCE):
9. "An Elegant Puzzle" — Will Larson
10. "Micro Frontends in Action" — Michael Geers

### Online Resources:
- patterns.dev (design patterns + rendering patterns)
- web.dev (performance + Core Web Vitals + browser fundamentals)
- react.dev (React official docs)
- greatfrontend.com (system design + interview prep)
- totalTypeScript.com (practical TypeScript)
- neetcode.io (DSA problem curation by pattern)

---

## Weekly Schedule Template (13 hours/week total)

```
Monday:      2 hours  — Reading (book/article/docs)
Tuesday:     45 min   — DSA practice (1 problem)
Wednesday:   3 hours  — Building (hands-on project)
Thursday:    45 min   — DSA practice (1 problem)
Friday:      3 hours  — Building (hands-on project)
Saturday:    2 hours  — Writing (tradeoff doc, ADR, RFC, or blog post)
Sunday:      90 min   — DSA review + extra problem (Month 7+)
```

---

## Non-Negotiable Rules

1. **DSA every week from Day 1.** Not negotiable. Not "I'll start later." 2 problems/week minimum.
2. **WRITE every week.** A Staff Engineer who can't write design docs isn't a Staff Engineer.
3. **BUILD every week.** Reading without building is useless. Measure what you build.
4. **DOCUMENT tradeoffs.** Every technical decision — write down WHY and WHAT YOU GAVE UP.
5. **SHARE your work.** Present to your team. Write blog posts. Get feedback. Staff = influence.
6. **OWN production.** If you've never debugged a production issue with source maps and session replay, you're not ready for Staff.
7. **Go deep on your capstone.** It's proof of Staff-level thinking. Depth > breadth.
8. **Don't skip browser internals.** FAANG interviewers test first-principles reasoning. You can't fake it.
9. **Talk to your manager.** Tell them you want Staff. Get their input. Align your growth with company needs.

---

## Success Metrics

After 12 months, you should be able to:

1. Design a frontend system from scratch in 45 minutes using the RADIO framework
2. Write an RFC that your team can follow and implement without asking you questions
3. Evaluate micro-frontend vs monolith for any given team/product with data
4. Optimize a web app from 40 → 90+ Lighthouse score and explain the business impact
5. Build an AI-powered feature with streaming, caching, cost management, and graceful degradation
6. Set up a monorepo with shared packages, CI/CD, preview deployments, and production monitoring
7. Articulate tradeoffs between any two technical approaches with data and business context
8. Diagnose a production frontend issue using error monitoring, source maps, and session replay
9. Solve a LeetCode Medium in 25 minutes consistently
10. Pass a FAANG Staff Frontend interview loop

---

## What Changed from v1 → v2 (Summary)

| Area | v1 | v2 | Why |
|------|----|----|-----|
| DSA | 20 hours in Month 11-12 | 144 problems across 12 months | FAANG coding rounds require consistent practice, not last-minute cramming |
| Browser internals | Not covered | Full module in Month 1 (rendering, event loop, memory, networking, security) | FAANG tests first-principles reasoning, not framework knowledge |
| TypeScript | Academic (conditional types, mapped types) | Practical (domain modeling, Zod, branded types, migration strategies) | Staff engineers need practical type safety, not type gymnastics |
| Micro-frontends | Full month (40 hours) | 2 weeks (10 hours) — decision framework only | Most FAANG companies don't use MFE. Know when to use it, don't spend a month building it |
| Data architecture | Not covered | Full module (TanStack Query, GraphQL, Relay, real-time, offline, BFF) | Staff engineers spend more time on data layer than component architecture |
| AI | API consumer level (chat + streaming) | Architectural level (UX patterns, cost, evaluation, prompt management, edge inference) | In 2026, AI is an architectural concern, not a feature bolt-on |
| Production ownership | Not covered | Full module (Sentry, feature flags, SLOs, incident response, postmortems) | Staff engineers own production. This was the biggest gap in v1 |
| System design | Generic template | RADIO framework + AI-era problems | FAANG uses structured frameworks. Modern problems include AI-native design |
| Capstone | Wide (everything demo) | Deep (choose ONE track, go deep) | Depth demonstrates Staff thinking. Breadth demonstrates following a plan |
| Interview prep | 4-6 mocks, 8 design problems | 6-8 mocks, 15+ design problems, 8 behavioral questions, STAR format | Staff interviews are harder. More practice = better outcomes |
| Schedule | 10 hours/week | 13 hours/week (10 structured + 3 DSA) | DSA requires dedicated time outside the main plan |

---

*"The difference between a Senior and a Staff Engineer is not what they can build —
it's what they can decide, communicate, and enable others to build.
And then own it in production."*

This will be the hardest and most rewarding year of your career. Every hour counts. Start today.
