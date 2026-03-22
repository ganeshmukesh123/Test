# 🧭 The Staff Engineer's Tradeoff Guide — Part 1

## How to THINK About Tradeoffs + Architecture Decisions

### For: Ganesh Choudhary
### Context: Senior → Staff Frontend Engineer (FAANG-ready)

---

## ⚠️ The Most Important Thing First

**A Senior Engineer picks a solution. A Staff Engineer picks the RIGHT solution
for the CONTEXT and can EXPLAIN WHY to anyone.**

At FAANG, you won't be asked "What's better, SSR or CSR?"
You'll be asked: **"How would you decide between SSR and CSR for THIS product?"**

The answer is NEVER "X is better than Y."
The answer is ALWAYS: **"It depends. Here's my decision framework."**

---

## 🧠 The Universal Tradeoff Framework

Before diving into specific tradeoffs, learn THIS framework.
Apply it to EVERY decision you make.

### Step 1: Define the CONTEXT (most people skip this)

```
Questions to ask BEFORE choosing anything:

TEAM:
- How many engineers? (1, 5, 20, 100?)
- What's their experience level? (junior-heavy? senior-heavy?)
- How many teams will touch this? (1 team vs 5 autonomous teams?)
- What does the team already know? (switching cost is REAL)

PRODUCT:
- What type of product? (content site, dashboard, e-commerce, SaaS?)
- How many users? (100, 10K, 1M, 100M?)
- What's the read/write ratio? (blog = 99% read, editor = 50/50)
- How dynamic is the content? (static blog vs real-time dashboard)
- How important is SEO? (marketing site = critical, admin panel = zero)
- Is offline support needed?

BUSINESS:
- What's the timeline? (2 weeks, 3 months, 1 year?)
- What's the maintenance horizon? (build and forget? maintain for 5 years?)
- What's the cost budget? (infrastructure, developer time, licenses?)
- How fast will requirements change?

TECHNICAL:
- What's the existing tech stack? (migration cost is huge)
- What's the deployment infrastructure? (Vercel? AWS? bare metal?)
- What are the performance requirements? (SLA: <200ms response?)
- What are the compliance requirements? (GDPR, HIPAA, SOC2?)
```

### Step 2: Identify the DIMENSIONS That Matter

```
Not all tradeoff dimensions matter equally for every project.
Rank these for YOUR specific project:

1. PERFORMANCE — Page load speed, interaction responsiveness
2. DEVELOPER EXPERIENCE (DX) — Build speed, debugging, type safety
3. SCALABILITY — Handle more users/data/teams
4. MAINTAINABILITY — Easy to understand, modify, debug over years
5. TIME TO MARKET — How fast can we ship v1?
6. COST — Infrastructure, licensing, developer hours
7. FLEXIBILITY — Easy to change direction later
8. RELIABILITY — Uptime, error handling, graceful degradation
9. SECURITY — Attack surface, compliance
10. ACCESSIBILITY — Support for all users
```

### Step 3: Apply the Decision Matrix

```
For each option, rate it on YOUR top dimensions (1-5):

Example: Choosing rendering strategy for a marketing site
Top dimensions: SEO (critical), Performance (high), Time to Market (high)

| Dimension       | Weight | CSR | SSR | SSG |
|-----------------|--------|-----|-----|-----|
| SEO             | 5      | 1   | 5   | 5   |
| Performance     | 4      | 2   | 4   | 5   |
| Time to Market  | 4      | 5   | 3   | 4   |
| DX              | 3      | 5   | 3   | 4   |
| Cost            | 2      | 5   | 2   | 4   |
|                 |        |     |     |     |
| Weighted Total  |        | 58  | 66  | 82  | ← SSG wins for this context
```

### Step 4: Document the Decision (ADR)

```markdown
# ADR-001: Rendering Strategy for Marketing Site

## Status: Accepted
## Date: 2026-03-21
## Context:
We're building a marketing site with 50 pages. SEO is critical.
Content changes weekly, not in real-time. Team of 3 frontend engineers.

## Decision: Static Site Generation (SSG) with Next.js

## Rationale:
- SEO: SSG serves pre-rendered HTML (perfect for crawlers)
- Performance: Pre-built pages served from CDN (sub-100ms TTFB)
- Cost: No server needed (static hosting on Vercel/CloudFront)

## Alternatives Considered:
- CSR (React SPA): Rejected — poor SEO, slower initial load
- SSR: Rejected — unnecessary server cost for content that changes weekly

## What We're Giving Up:
- Real-time content updates (acceptable: content changes weekly)
- Dynamic per-user content on initial load (we don't need this)
- Must rebuild to update content (acceptable: CI/CD handles this)

## Risks:
- Build time grows with page count (mitigate: ISR for high-page-count sections)
```

---

# 📐 Tradeoff 1: Rendering Strategy

## CSR vs SSR vs SSG vs ISR vs Streaming SSR

### What Each Actually Means

```
CSR (Client-Side Rendering):
  Server sends empty HTML + JS bundle
  Browser downloads JS → executes → renders content

  Timeline:
  [blank page] ────→ [JS downloads] ────→ [JS executes] ────→ [Content visible]
                                                                 ↑ User sees content HERE

SSR (Server-Side Rendering):
  Server runs your components → sends FULL HTML
  Browser shows HTML immediately → downloads JS → "hydrates" (makes interactive)

  Timeline:
  [Full HTML visible] ────→ [JS downloads] ────→ [Hydrates] ────→ [Interactive]
   ↑ User sees content HERE                                        ↑ User can click HERE

SSG (Static Site Generation):
  At BUILD TIME: Server pre-renders all pages to HTML files
  At REQUEST TIME: CDN serves pre-built HTML (no server computation)

  Timeline:
  [Full HTML from CDN] ────→ [JS downloads] ────→ [Hydrates] ────→ [Interactive]
   ↑ Instant! (from CDN)

ISR (Incremental Static Regeneration):
  Like SSG, but pages regenerate in the background after a timeout
  First request: Serve stale page → rebuild in background
  Next request: Serve fresh page

  Timeline: Same as SSG, but content stays fresh without full rebuild

Streaming SSR:
  Server sends HTML in chunks as components render
  Browser shows content progressively (no waiting for full page)

  Timeline:
  [Header HTML] → [Main content HTML] → [Sidebar HTML] → [JS hydrates]
   ↑ Visible immediately   ↑ Appears next      ↑ Last to appear
```

### The Decision Matrix

```
| Factor                | CSR        | SSR           | SSG          | ISR           | Streaming SSR |
|-----------------------|------------|---------------|--------------|---------------|---------------|
| SEO                   | ❌ Poor     | ✅ Excellent   | ✅ Excellent  | ✅ Excellent   | ✅ Excellent    |
| Initial Load (LCP)    | ❌ Slow     | ✅ Fast        | ✅ Fastest    | ✅ Fastest     | ✅ Fast         |
| Time to Interactive    | ❌ Slow     | ⚠️ Medium      | ⚠️ Medium     | ⚠️ Medium      | ✅ Fast         |
| Server Cost            | ✅ None     | ❌ High        | ✅ None (CDN) | ✅ Low         | ❌ High         |
| Content Freshness      | ✅ Always   | ✅ Always      | ❌ Build-time | ⚠️ Near-real   | ✅ Always        |
| Build Time             | ✅ Fast     | ✅ N/A         | ❌ Slow*      | ✅ Fast        | ✅ N/A          |
| Personalization        | ✅ Easy     | ✅ Easy        | ❌ Hard       | ⚠️ Limited     | ✅ Easy          |
| Complexity             | ✅ Simple   | ⚠️ Medium      | ✅ Simple     | ⚠️ Medium      | ❌ Complex       |
| Offline Capability     | ✅ Easy     | ⚠️ Possible    | ✅ Easy       | ✅ Easy        | ⚠️ Possible     |
| Infrastructure         | Static host| Node server   | CDN          | CDN + revalidate| Node server  |

*SSG build time grows with number of pages
```

### When to Use Each — Decision Tree

```
START HERE: Does this page need SEO?
│
├── NO (admin panel, dashboard, internal tool, SPA behind login)
│   └── → CSR (simplest, cheapest, no server needed)
│
├── YES → Is the content the same for all users?
│   │
│   ├── YES → Does content change frequently?
│   │   │
│   │   ├── RARELY (blog, docs, marketing: changes weekly or less)
│   │   │   └── → SSG (fastest, cheapest, best SEO)
│   │   │
│   │   ├── SOMETIMES (e-commerce catalog: changes daily)
│   │   │   └── → ISR (fresh content without full rebuild)
│   │   │
│   │   └── CONSTANTLY (news feed, live data)
│   │       └── → SSR or Streaming SSR
│   │
│   └── NO (personalized content, user-specific data)
│       │
│       ├── Is TTFB critical? (< 200ms SLA)
│       │   └── → Streaming SSR (send shell fast, stream personalized parts)
│       │
│       └── TTFB is flexible
│           └── → SSR (simpler than streaming)
```

### Real-World Examples

```
Netflix.com landing page          → SSG (static marketing, rarely changes)
Netflix.com/browse (logged in)    → CSR (personalized, no SEO needed, behind auth)
Amazon.com product page           → SSR or ISR (SEO critical, millions of pages)
Amazon.com checkout               → CSR (behind auth, no SEO)
Twitter.com/home feed             → CSR + SSR hybrid (SEO for profiles, CSR for feed)
Blog/documentation site           → SSG (content changes on deploy, SEO critical)
Google Docs editor                → CSR (real-time, behind auth, no SEO needed)
E-commerce category pages         → ISR (SEO critical, content changes daily)
Company dashboard (internal)      → CSR (no SEO, behind auth, simplest)
News website article page         → SSR or Streaming SSR (SEO, fresh content, fast TTFB)
```

### Anti-Patterns (Mistakes People Make)

```
❌ Using SSR for an admin dashboard behind login
   → Unnecessary server cost. SEO doesn't matter. CSR is simpler.

❌ Using CSR for a marketing/SEO-critical site
   → Search engines may not index your content properly.

❌ Using SSG for a site with 1 million dynamic pages
   → Build time will be hours. Use ISR instead.

❌ Using SSR when ISR would work
   → Paying for server computation on every request when content
     only changes hourly. ISR serves cached pages from CDN.

❌ Choosing SSR because "it's better" without measuring
   → SSR adds server cost, operational complexity, and a potential
     SPOF (single point of failure). Only use it when you need it.

❌ Not considering hybrid approaches
   → Most real apps use MULTIPLE strategies:
     Marketing pages → SSG
     Product pages → ISR
     User dashboard → CSR
     Search results → SSR
```

---

# 📐 Tradeoff 2: SPA vs MPA (vs Hybrid)

### What They Are

```
SPA (Single Page Application):
  - ONE HTML page, all routing handled by JavaScript
  - Page never fully reloads
  - Frameworks: React SPA, Vue SPA, Angular
  - Example: Gmail, Figma, Spotify Web

MPA (Multi-Page Application):
  - Each route is a SEPARATE HTML page from the server
  - Full page reload on navigation
  - Frameworks: Traditional Rails, Django, PHP, Astro
  - Example: Wikipedia, GitHub (mostly), Amazon

HYBRID (Modern default):
  - Initial page is server-rendered (MPA-like)
  - Subsequent navigations are client-side (SPA-like)
  - Frameworks: Next.js, Nuxt, Remix, SvelteKit
  - Example: Twitter/X, Vercel.com, most modern apps
```

### The Decision Matrix

```
| Factor                   | SPA           | MPA            | Hybrid          |
|--------------------------|---------------|----------------|-----------------|
| Initial Load             | ❌ Slow (JS)   | ✅ Fast (HTML)  | ✅ Fast (SSR)    |
| Navigation Speed         | ✅ Instant     | ❌ Full reload   | ✅ Instant       |
| SEO                      | ❌ Poor        | ✅ Excellent    | ✅ Excellent     |
| Complexity               | ⚠️ Medium      | ✅ Simple       | ❌ High          |
| JS Bundle Size           | ❌ Large       | ✅ Small/page   | ⚠️ Medium        |
| Offline Support          | ✅ Easy        | ❌ Hard         | ⚠️ Possible      |
| State Preservation       | ✅ Easy        | ❌ Lost on nav   | ✅ Easy          |
| Server Requirement       | ❌ No          | ✅ Yes          | ✅ Yes           |
| Back/Forward nav         | ⚠️ Must manage | ✅ Native       | ✅ Native        |
| Memory Usage             | ❌ Grows       | ✅ Fresh/page   | ⚠️ Medium        |
| Deep Linking             | ⚠️ Must manage | ✅ Native       | ✅ Native        |
```

### When to Use Each

```
SPA:
  ✅ App-like experiences (editors, dashboards, tools)
  ✅ Behind authentication (no SEO needed)
  ✅ Rich interactivity (drag-and-drop, real-time collaboration)
  ✅ Offline-capable apps (PWA)
  ✅ When fast navigation between views is critical
  Examples: Google Docs, Figma, Slack, VS Code Web

MPA:
  ✅ Content-heavy sites (blogs, docs, news)
  ✅ SEO is the #1 priority
  ✅ Minimal JavaScript needed
  ✅ Team has backend expertise (Rails, Django, PHP)
  ✅ Maximum simplicity
  Examples: Wikipedia, government sites, documentation sites

HYBRID (recommended for most new projects):
  ✅ Need both SEO and interactivity
  ✅ E-commerce, SaaS, social media
  ✅ Progressive enhancement
  ✅ Best of both worlds
  Examples: Most modern web apps, e-commerce, SaaS products
```

---

# 📐 Tradeoff 3: Monolith vs Micro-Frontend

### What They Are

```
MONOLITH:
  One codebase, one build, one deployment
  All features live together
  ┌─────────────────────────────────────┐
  │         SINGLE APPLICATION          │
  │  ┌─────┐ ┌──────┐ ┌──────────┐    │
  │  │ Auth │ │ Cart │ │ Products │    │
  │  └─────┘ └──────┘ └──────────┘    │
  │  ┌──────────┐ ┌──────────────┐    │
  │  │ Checkout │ │ Admin Panel  │    │
  │  └──────────┘ └──────────────┘    │
  └─────────────────────────────────────┘
  ONE team builds and deploys everything

MODULAR MONOLITH:
  One codebase, but well-organized into modules
  Clear boundaries between features
  Still one build and one deployment
  ┌─────────────────────────────────────┐
  │         SINGLE APPLICATION          │
  │  ┌─────────┐  ┌─────────┐         │
  │  │ Module A │  │ Module B │  ...   │
  │  │ (own dir │  │ (own dir │         │
  │  │  own API)│  │  own API)│         │
  │  └─────────┘  └─────────┘         │
  └─────────────────────────────────────┘
  Multiple teams, shared deployment

MICRO-FRONTEND:
  Multiple codebases, independent builds, independent deployments
  Each team owns a complete vertical slice
  ┌──────────────────────────────────────────┐
  │              SHELL / HOST                 │
  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
  │  │ Team A   │ │ Team B   │ │ Team C   │ │
  │  │ Products │ │ Cart     │ │ Account  │ │
  │  │ (React)  │ │ (React)  │ │ (Vue)    │ │
  │  │ Own CI/CD│ │ Own CI/CD│ │ Own CI/CD│ │
  │  └──────────┘ └──────────┘ └──────────┘ │
  └──────────────────────────────────────────┘
  Each team deploys independently
```

### The Decision Matrix

```
| Factor                     | Monolith    | Modular Monolith | Micro-Frontend   |
|----------------------------|-------------|------------------|------------------|
| Setup Complexity           | ✅ Low       | ⚠️ Medium         | ❌ High           |
| Team Independence          | ❌ Low       | ⚠️ Medium         | ✅ High           |
| Deployment Independence    | ❌ One deploy| ❌ One deploy     | ✅ Independent    |
| Code Sharing               | ✅ Easy      | ✅ Easy           | ❌ Hard           |
| Consistency (UI/UX)        | ✅ Easy      | ✅ Easy           | ❌ Hard           |
| Performance                | ✅ Best      | ✅ Good           | ⚠️ Overhead       |
| Build Time                 | ❌ Grows     | ⚠️ Manageable     | ✅ Per-team       |
| Tech Diversity             | ❌ One stack | ❌ One stack      | ✅ Per-team       |
| Testing                    | ✅ Simple    | ✅ Moderate       | ❌ Complex        |
| Onboarding New Developers  | ✅ Simple    | ⚠️ Moderate       | ⚠️ Per-team       |
| Maintenance Cost           | ✅ Low       | ⚠️ Medium         | ❌ High           |
| Refactoring                | ✅ Easy      | ⚠️ Moderate       | ❌ Hard (cross-MF) |
```

### Decision Tree

```
START: How many frontend engineers?
│
├── 1-10 engineers, 1-2 teams
│   └── → MONOLITH (don't overthink it)
│         Micro-frontends add complexity you don't need.
│         Focus on good architecture within the monolith.
│
├── 10-30 engineers, 2-5 teams
│   │
│   ├── Teams can coordinate on release schedules?
│   │   └── YES → MODULAR MONOLITH (Nx or Turborepo monorepo)
│   │         Best balance of team independence and simplicity.
│   │
│   └── Teams CANNOT coordinate? Different release cadences?
│       └── → Consider MICRO-FRONTEND
│             But try modular monolith first.
│
├── 30+ engineers, 5+ autonomous teams
│   │
│   ├── Teams need technology independence? (different frameworks)
│   │   └── YES → MICRO-FRONTEND (this is the actual use case)
│   │
│   ├── Legacy migration? (strangler fig pattern)
│   │   └── YES → MICRO-FRONTEND (wrap old app, build new features separately)
│   │
│   └── Same tech stack, teams can share build pipeline?
│       └── → MODULAR MONOLITH in MONOREPO
│             Even at this scale, a well-structured monolith often wins.

GOLDEN RULE:
Start with a monolith. Extract to micro-frontends ONLY when you have
a real organizational scaling problem. Premature micro-frontend = pain.
```

### Real-World Examples

```
Company              Engineers    Architecture      Why
─────────────────────────────────────────────────────────────────────
Small startup        3-5         Monolith          No need for complexity
Medium SaaS          10-20       Modular Monolith  Clean boundaries, one deploy
IKEA                 100+        Micro-Frontend    Multiple autonomous teams
Spotify              200+        Micro-Frontend    Squad model, independence
Netflix              100+        Micro-Frontend    Legacy migration needed
Shopify              500+        Modular Monolith  React + shared packages
Amazon               1000+       Micro-Frontend    Hundreds of teams, diverse tech
```

### Anti-Patterns

```
❌ "Let's use micro-frontends because Netflix does"
   → Netflix has 100+ frontend engineers. You have 5. You're not Netflix.

❌ Micro-frontends for a single team
   → All overhead, no benefit. A team doesn't need independence from itself.

❌ Sharing too much state between micro-frontends
   → If micro-frontends need to constantly share state, they shouldn't be separate.
     Micro-frontends should be as independent as possible.

❌ Different versions of React across micro-frontends
   → 3 copies of React = 300KB+ extra JavaScript. Share framework as singleton.

❌ Micro-frontends without a design system
   → UI inconsistency guaranteed. Build shared design system FIRST.

❌ Skipping modular monolith and jumping to micro-frontends
   → If you can't build a well-structured monolith, you can't build
     well-structured micro-frontends. The problem is architecture, not boundaries.
```

---

# 📐 Tradeoff 4: REST vs GraphQL vs tRPC vs gRPC

### The Decision Matrix

```
| Factor                     | REST        | GraphQL      | tRPC         | gRPC          |
|----------------------------|-------------|--------------|--------------|---------------|
| Learning Curve             | ✅ Low       | ❌ High       | ✅ Low        | ⚠️ Medium      |
| Type Safety                | ❌ Manual    | ⚠️ Codegen    | ✅ Automatic  | ✅ Proto-based |
| Over-fetching              | ❌ Common    | ✅ Solved     | ✅ Solved     | ⚠️ Fixed schema|
| Under-fetching             | ❌ Common    | ✅ Solved     | ⚠️ Per-procedure| ⚠️ Per-method |
| Caching                    | ✅ HTTP native| ❌ Complex   | ⚠️ Manual     | ❌ Manual      |
| File Uploads               | ✅ Easy      | ❌ Complex    | ⚠️ Possible   | ✅ Streaming   |
| Real-time (subscriptions)  | ❌ Separate  | ✅ Built-in   | ✅ Built-in   | ✅ Streaming   |
| Browser Support            | ✅ Native    | ✅ (POST)     | ✅ (fetch)    | ❌ Needs proxy |
| Public API                 | ✅ Best      | ✅ Good       | ❌ TS only    | ⚠️ Needs client|
| Multiple Consumers         | ✅ Any       | ✅ Any        | ❌ TS only    | ✅ Any lang    |
| Bundle Size Impact         | ✅ None      | ⚠️ Client lib | ✅ Minimal    | ❌ Proto lib   |
| Tooling/Ecosystem          | ✅ Mature    | ✅ Rich       | ⚠️ Growing    | ✅ Mature      |
| Performance (payload)      | ⚠️ Verbose   | ✅ Minimal    | ✅ Minimal    | ✅ Binary      |
```

### Decision Tree

```
START: Who consumes this API?
│
├── PUBLIC API (third-party developers, mobile apps, different languages)
│   │
│   ├── Simple CRUD operations?
│   │   └── → REST (universally understood, great tooling)
│   │
│   └── Complex data relationships? Consumers need flexibility?
│       └── → GraphQL (let consumers request exactly what they need)
│
├── INTERNAL API (your frontend team only)
│   │
│   ├── Same TypeScript monorepo? (frontend + backend)
│   │   └── → tRPC (best DX, automatic type safety, zero overhead)
│   │
│   ├── Multiple frontend apps, same backend?
│   │   │
│   │   ├── Apps need different data shapes from same entities?
│   │   │   └── → GraphQL (each app queries what it needs)
│   │   │
│   │   └── Apps need similar data?
│   │       └── → REST (simpler, cacheable)
│   │
│   └── Backend is a different language (Go, Java, Python)?
│       │
│       ├── Need high performance / streaming?
│       │   └── → gRPC with REST gateway for browser
│       │
│       └── Standard web app?
│           └── → REST or GraphQL
│
└── MICROSERVICES (service-to-service communication)
    └── → gRPC (binary, fast, streaming, strong typing with protobuf)
        Browser consumers: Add REST/GraphQL gateway in front
```

### When Each Wins (concrete scenarios)

```
REST wins when:
  - Building a public API (widest adoption)
  - Simple CRUD (users, posts, comments)
  - HTTP caching is important (GET requests cacheable by default)
  - Team is new to API design (lowest learning curve)
  - Integrating with third-party services (most use REST)

GraphQL wins when:
  - Mobile apps with bandwidth constraints (request only needed fields)
  - Complex UI with nested relationships (user → posts → comments → author)
  - Multiple consumers needing different data shapes
  - Rapid frontend iteration (change queries without backend changes)
  - Real-time features (subscriptions built-in)

tRPC wins when:
  - Full-stack TypeScript (Next.js, Nuxt + Node backend)
  - Same team owns frontend and backend
  - Monorepo setup
  - Maximum developer velocity (zero API layer boilerplate)
  - Internal tools (not public-facing APIs)

gRPC wins when:
  - Microservice-to-microservice communication
  - Need binary protocol (high performance)
  - Need streaming (bi-directional)
  - Polyglot backends (Go, Java, Python — protobuf generates clients)
  - NOT directly consumed by browsers (needs a gateway)
```

### Anti-Patterns

```
❌ GraphQL for a simple CRUD app with 5 endpoints
   → Massive over-engineering. REST is simpler and cacheable.

❌ tRPC for a public API consumed by mobile apps and third parties
   → tRPC requires TypeScript. External consumers can't use it.

❌ REST when frontend needs 5 round trips to render one page
   → This is the N+1 problem. GraphQL solves it with one query.

❌ GraphQL without a caching strategy
   → All queries are POST → HTTP cache doesn't work → you need
     Apollo Client/urql cache → complexity grows fast.

❌ Choosing GraphQL because "Facebook uses it"
   → Facebook has thousands of frontend engineers working on one API.
     Your team of 5 doesn't have this problem.

❌ Building REST API without OpenAPI/Swagger spec
   → No auto-generated types → no type safety → bugs in production.
     If you use REST, ALWAYS have an OpenAPI spec.
```

---

# 📐 Tradeoff 5: State Management

### Options Landscape

```
NO LIBRARY (React built-ins):
  useState        → Component-local state
  useReducer      → Complex local state with actions
  useContext      → Share state across components (no prop drilling)
  URL state       → State encoded in URL (query params, path)

  When: Small-medium apps, simple state, 1-3 developers

LIGHTWEIGHT LIBRARIES:
  Zustand         → Simple global store, minimal API (React)
  Jotai           → Atomic state (bottom-up, like Recoil but simpler)
  Pinia           → Vue's official store (you know this)
  Nanostores      → Framework-agnostic atomic stores

  When: Medium apps, need global state but Redux feels heavy

FULL SOLUTIONS:
  Redux Toolkit   → Predictable, devtools, middleware, RTK Query
  MobX            → Observable-based, automatic tracking
  XState          → State machines, finite states, visualizer

  When: Large apps, complex state logic, multiple teams

SERVER STATE LIBRARIES:
  TanStack Query  → Server state caching, refetching, sync
  SWR             → Similar to TanStack Query, lighter
  Apollo Client   → GraphQL-specific cache + state

  When: Most of your "state" is really server data
```

### The Critical Insight Most People Miss

```
╔═══════════════════════════════════════════════════════════════╗
║  80% of what people put in "global state" is SERVER STATE.  ║
║  Handle server state with TanStack Query / SWR.             ║
║  You'll need very little client state after that.           ║
╚═══════════════════════════════════════════════════════════════╝

What is CLIENT state? (truly belongs in a store)
  - UI state: sidebar open/closed, modal visibility, active tab
  - Form state: current input values (before submission)
  - Theme: dark/light mode
  - Auth: current user (after login)

What is SERVER state? (use TanStack Query / SWR)
  - User profile data
  - Product catalog
  - Comments, posts
  - Any data that lives in a database
  - Anything fetched from an API

What is URL state? (use the URL)
  - Current page/route
  - Search filters
  - Pagination
  - Sort order
  - Selected tab (if it should survive refresh/sharing)
```

### Decision Tree

```
START: What kind of state?
│
├── Server data (fetched from API)?
│   └── → TanStack Query or SWR
│         Handles: caching, refetching, optimistic updates, loading/error states
│         You don't need Redux/Zustand for this.
│
├── URL-representable? (filters, pagination, search, sort)
│   └── → URL state (useSearchParams, router query)
│         Benefits: shareable links, browser back/forward works, SSR-friendly
│
├── Component-local? (only this component needs it)
│   └── → useState or useReducer
│         Don't put everything in global state.
│
├── Shared between a few nearby components?
│   └── → Lift state up to common parent, or useContext
│
└── Truly global client state? (theme, auth, UI state)
    │
    ├── Simple? (< 5 values)
    │   └── → useContext + useReducer (no library needed)
    │
    ├── Medium complexity?
    │   └── → Zustand (React) or Pinia (Vue)
    │         Minimal boilerplate, excellent DX
    │
    └── Complex? (many teams, complex transitions, time-travel debugging needed)
        │
        ├── Predictable state transitions needed?
        │   └── → Redux Toolkit
        │
        └── Complex state machines? (wizard flows, multi-step processes)
            └── → XState
```

### Comparison Matrix

```
| Factor                | Context+Reducer | Zustand   | Redux TK  | TanStack Query |
|-----------------------|-----------------|-----------|-----------|----------------|
| Bundle Size           | 0 KB            | ~1 KB     | ~11 KB    | ~13 KB         |
| Boilerplate           | Low             | Minimal   | Low (RTK) | Minimal        |
| Learning Curve        | Low             | Low       | Medium    | Medium         |
| DevTools              | React DevTools  | Basic     | Excellent | Excellent      |
| Middleware             | Manual          | Yes       | Yes       | N/A            |
| Async handling         | Manual          | Manual    | RTK Query | Built-in       |
| Server state focus     | No              | No        | RTK Query | YES            |
| TypeScript support     | Good            | Excellent | Excellent | Excellent      |
| Persistence            | Manual          | Plugin    | Plugin    | Built-in cache |
| Best for               | Simple apps     | Most apps | Large apps| Server data    |
```

### Anti-Patterns

```
❌ Putting server data in Redux
   → You'll manually handle loading, error, caching, refetching, stale data.
     TanStack Query does all of this automatically.

❌ Global state for everything
   → Form input values don't need to be global.
     Modal open/close doesn't need to be global.
     Keep state as LOCAL as possible.

❌ Not using URL state for shareable UI state
   → Filters, pagination, sort order should be in the URL.
     Users expect: copy URL → paste → see same view.
     Browser back button should work for filter changes.

❌ Multiple state management libraries in one app
   → Pick ONE for client state. ONE for server state. Done.
     Don't use Redux + Zustand + Context in the same app.

❌ Choosing Redux for a 3-page app
   → Over-engineering. useState + useContext is plenty.

❌ No state management strategy (just vibes)
   → Document: "What goes where?"
     Server data → TanStack Query
     URL state → URL
     Local UI → useState
     Global UI → Zustand
     Write this down. Share with team.
```

---

# 📐 Tradeoff 6: CSS Approach

### Options

```
VANILLA CSS:             Plain .css files, class names
CSS MODULES:             Scoped class names, .module.css files
TAILWIND CSS:            Utility-first, classes in HTML
CSS-IN-JS (Runtime):     styled-components, Emotion — JS generates CSS
CSS-IN-JS (Zero-runtime): Vanilla Extract, Panda CSS, StyleX — CSS at build time
COMPONENT LIBRARIES:     MUI, Chakra, Ant Design — pre-styled components
HEADLESS UI + Tailwind:  Radix + Tailwind, Headless UI + Tailwind
```

### The Decision Matrix

```
| Factor                 | CSS Modules | Tailwind    | styled-comp | Vanilla Extract | MUI/Chakra |
|------------------------|-------------|-------------|-------------|-----------------|------------|
| Performance            | ✅ Best      | ✅ Great     | ❌ Runtime   | ✅ Great         | ⚠️ Heavy    |
| Bundle Size            | ✅ Minimal   | ✅ Small*    | ⚠️ Grows     | ✅ Small         | ❌ Large    |
| DX                     | ⚠️ Manual    | ✅ Fast      | ✅ Colocation| ⚠️ Verbose       | ✅ Fast      |
| Type Safety            | ❌ No        | ❌ No        | ⚠️ Limited   | ✅ Full          | ⚠️ Limited   |
| SSR Compatibility      | ✅ Perfect   | ✅ Perfect   | ⚠️ Complex   | ✅ Perfect       | ⚠️ Complex   |
| Design Tokens          | ✅ CSS vars  | ✅ Config    | ⚠️ JS theme  | ✅ Built-in      | ✅ Theme     |
| Learning Curve         | ✅ Low       | ⚠️ Medium    | ✅ Low       | ⚠️ Medium        | ✅ Low       |
| Community              | ✅ Huge      | ✅ Huge      | ⚠️ Declining | ⚠️ Growing       | ✅ Large     |
| Micro-frontend safe    | ✅ Yes       | ⚠️ Config    | ❌ Conflicts | ✅ Yes           | ❌ Conflicts |

*Tailwind: PurgeCSS removes unused utilities → typically 5-15KB
```

### Decision Tree

```
START: What type of project?
│
├── Marketing site / Landing pages
│   └── → Tailwind CSS (fast development, responsive utilities)
│
├── Design system / Component library
│   │
│   ├── Need runtime theme switching?
│   │   └── → CSS custom properties + CSS Modules or Vanilla Extract
│   │
│   └── Static theme?
│       └── → Vanilla Extract (type-safe, zero-runtime, SSR-safe)
│
├── Large application (many teams)
│   │
│   ├── Need scoping guarantee? (micro-frontends)
│   │   └── → CSS Modules or Vanilla Extract (guaranteed no conflicts)
│   │
│   └── Single team, need speed?
│       └── → Tailwind CSS (utility classes = fast, consistent)
│
├── Application with heavy theming?
│   └── → CSS custom properties + any approach
│         CSS vars = runtime theme switching without JS overhead
│
└── Rapid prototyping / MVP?
    └── → Component library (MUI, Chakra) or Tailwind + Headless UI
          Fastest time to market
```

### The 2026 Recommendation

```
For most new projects in 2026:

1. TAILWIND CSS + HEADLESS UI COMPONENTS
   → Fastest development, smallest bundle, SSR-safe, no runtime overhead
   → Headless components (Radix, Headless UI) for accessibility
   → This is what Vercel, Stripe, and most modern companies use

2. CSS MODULES
   → If you hate utility classes
   → If you need guaranteed style isolation (micro-frontends)
   → Simple, zero-runtime, SSR-safe

3. VANILLA EXTRACT
   → If you need type-safe CSS
   → If you're building a design system
   → Zero-runtime, SSR-safe, TypeScript-first

AVOID in 2026:
   → Runtime CSS-in-JS (styled-components, Emotion)
     React core team recommends against runtime CSS-in-JS
     SSR complications, performance overhead, React 19 compatibility issues
```

---

*Continued in Part 2 → `09-tradeoff-guide-part2.md`*
