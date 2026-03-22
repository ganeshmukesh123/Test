# 🎯 Staff Frontend Engineer — 12-Month Growth Plan

## Current: Senior Frontend Engineer (Vue, 7 YOE, Product/Startup)
## Target: Staff Frontend Engineer (FAANG-ready)
## Time: 10 hours/week | Reading + Building Projects
## Created: March 2026

---

## ⚠️ Honest Assessment First

I must be real with you. Based on your answers, here's what I see:

| Area | Current Level | Required for Staff @ FAANG |
|------|--------------|---------------------------|
| Framework depth | Vue only | React (mandatory) + one more |
| System design | No experience | Must lead architecture decisions |
| Micro-frontends | Zero | Must evaluate, not necessarily implement |
| Monorepo/tooling | Zero | Must architect build systems |
| Performance | No experience | Must optimize at scale |
| CI/CD | No understanding | Must design pipelines |
| RFC/ADR writing | Never written | Must write weekly |
| AI integration | Zero | Must build AI-powered features |
| Tradeoff decisions | None documented | Must teach others tradeoffs |
| Cross-team influence | Not started | Core Staff expectation |

**The gap is significant but closable in 12 months with focused, disciplined effort.**

**Critical truth:** 6 months is NOT realistic for your current state. Plan for 12 months minimum.
FAANG Staff Frontend = top ~10% of engineers. You need depth, not just awareness.

---

## 🧱 The Plan: 4 Phases

```
Phase 1: Foundation Reset (Month 1-3)     — "Learn to think differently"
Phase 2: Architecture Deep Dive (Month 4-6) — "Design systems at scale"
Phase 3: Advanced Patterns (Month 7-9)     — "Build Staff-level projects"
Phase 4: Staff Behaviors (Month 10-12)     — "Influence and interview prep"
```

---

## 📅 Phase 1: Foundation Reset (Month 1–3)
### Goal: Break out of Vue-only mindset, learn React, understand rendering models

### 🔑 Why This Phase Matters
FAANG (Meta, Google, Amazon, Apple, Netflix) heavily uses React. You CANNOT target
Staff Frontend at FAANG without React proficiency. Vue knowledge transfers well,
but you must be hands-on with React.

---

### Month 1: React Fundamentals + Rendering Models (40 hours)

#### Week 1-2: React Core (20 hours)

**Read (8 hours):**
- [ ] React official docs (react.dev) — read EVERY page of "Learn" section
- [ ] "Thinking in React" — understand component model vs Vue's Options/Composition API
- [ ] Read: React mental model differences from Vue
  - No two-way binding by default (vs v-model)
  - JSX vs templates
  - Hooks vs Composition API
  - No directive system

**Build (12 hours):**
- [ ] Project: Rebuild a feature you've already built in Vue → in React
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

#### Week 3-4: Rendering Models — CSR vs SSR vs SSG vs ISR (20 hours)

**Read (10 hours):**
- [ ] "Patterns.dev" — Rendering Patterns section (FREE) — https://www.patterns.dev/
- [ ] Next.js docs: "Rendering" section (App Router)
- [ ] Nuxt 3 docs: "Rendering" section (you'll understand faster coming from Vue)
- [ ] Read: "When should you use SSR?" by Alex Russell
- [ ] Read: Google's web.dev — Core Web Vitals documentation

**Build (10 hours):**
- [ ] Project: Build the SAME simple blog app in 3 ways:
  1. Pure CSR (Vite + React)
  2. SSR (Next.js App Router)
  3. SSG (Next.js with static export)
  - Measure and document: Time to First Byte, First Contentful Paint, Largest Contentful Paint
  - Use Chrome DevTools → Performance tab + Lighthouse
  - Write a 1-page comparison document (this is your FIRST tradeoff document)

**Tradeoff Document Template (use for all comparisons going forward):**
```markdown
# Decision: [Title]
## Context: What problem are we solving?
## Options Evaluated:
### Option A: [Name]
- How it works:
- Pros:
- Cons:
- Best when:
### Option B: [Name]
- (same structure)
## Recommendation: [Which one and WHY]
## What we're giving up: [Be explicit about tradeoffs]
```

---

### Month 2: State Management + TypeScript Depth (40 hours)

#### Week 5-6: State Management Patterns (20 hours)

**Read (8 hours):**
- [ ] Pinia docs (you know Vuex, understand Pinia deeply)
- [ ] Zustand docs + source code (it's tiny, ~500 lines — READ THE SOURCE)
- [ ] Redux Toolkit docs (RTK) — you'll see this in FAANG interviews
- [ ] React Query / TanStack Query docs — server state vs client state
- [ ] Read: "Application State Management with React" by Kent C. Dodds
- [ ] Read: "You Might Not Need Redux" by Dan Abramov

**Build (12 hours):**
- [ ] Project: E-commerce cart system with SAME business logic, 3 state solutions:
  1. React Context + useReducer (no library)
  2. Zustand (minimal library)
  3. Redux Toolkit + RTK Query (full solution)
  - Requirements: Add to cart, remove, quantity update, persist to localStorage, optimistic updates
  - Write tradeoff document: "Which state management for which scale?"

**Key Tradeoffs to Understand:**
```
| Criteria        | Context+Reducer | Zustand    | Redux Toolkit |
|-----------------|-----------------|------------|---------------|
| Bundle size     | 0 KB            | ~1 KB      | ~11 KB        |
| Boilerplate     | Medium          | Low        | Medium (RTK)  |
| DevTools        | React DevTools  | Basic      | Excellent     |
| Server state    | Manual          | Manual     | RTK Query     |
| Learning curve  | Low             | Low        | Medium        |
| Team scale      | Small           | Small-Med  | Large         |
| Best for        | Simple apps     | Most apps  | Complex/large |
```

#### Week 7-8: TypeScript for Architecture (20 hours)

**Read (10 hours):**
- [ ] "Total TypeScript" by Matt Pocock — Essential TypeScript section (book/website)
- [ ] TypeScript docs: Utility Types, Generics, Conditional Types, Mapped Types
- [ ] Read: "Effective TypeScript" by Dan Vanderkam — Chapters 3, 4, 5, 7
- [ ] Study type-level programming patterns used in libraries:
  - How does React.FC work?
  - How does Zustand infer types?
  - How does tRPC achieve end-to-end type safety?

**Build (10 hours):**
- [ ] Project: Build a type-safe API client layer
  - Generic fetch wrapper with type inference
  - API route definitions as TypeScript types
  - Auto-generated types from OpenAPI spec (use openapi-typescript)
  - Error handling with discriminated unions
  - This becomes a reusable pattern in all future projects

---

### Month 3: Build Systems + Monorepo Fundamentals (40 hours)

#### Week 9-10: Build Tools Deep Dive (20 hours)

**Read (8 hours):**
- [ ] "How does a bundler work?" — write your own mini-bundler (tutorial by Ronen Amiel)
- [ ] Vite docs: "Why Vite" + architecture overview
- [ ] Webpack docs: Concepts section (you must understand Webpack for micro-frontends later)
- [ ] esbuild docs: understand why it's fast (Go-based, no AST transforms)
- [ ] Read: "Module Federation" concept docs on webpack.js.org
- [ ] Read: Turbopack architecture overview

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
  - Shared TypeScript config
  - Shared ESLint config
  - Build pipeline: lint → type-check → test → build
  - Understand: task dependencies, caching, parallel execution

#### Week 11-12: CI/CD for Frontend (20 hours)

**Read (8 hours):**
- [ ] GitHub Actions docs: Workflow syntax, caching, matrix builds
- [ ] Read: "Frontend CI/CD Best Practices" (Vercel blog)
- [ ] Read: "Trunk Based Development" (trunkbaseddevelopment.com)
- [ ] Understand deployment strategies: Blue-green, Canary, Feature flags
- [ ] Read: LaunchDarkly docs — understand feature flag architecture

**Build (12 hours):**
- [ ] Project: Add CI/CD to your monorepo
  - GitHub Actions workflow:
    - PR checks: lint, type-check, unit tests, build
    - Affected-only builds (only rebuild what changed) — use Turborepo's `--filter`
    - Preview deployments for PRs (Vercel or Netlify)
    - Production deployment on merge to main
  - Add: Bundle size tracking (bundlesize or size-limit)
  - Add: Lighthouse CI for performance regression
  - Write tradeoff document: "CI/CD pipeline design decisions"

---

### 📊 Phase 1 Checkpoint — What You Should Have:
- [ ] React proficiency (can build features without looking up basic APIs)
- [ ] Understanding of CSR/SSR/SSG/ISR with measured tradeoffs
- [ ] 3 state management approaches compared with real code
- [ ] TypeScript at an architectural level (generics, utility types)
- [ ] A working monorepo with shared packages
- [ ] A CI/CD pipeline you designed
- [ ] 3 tradeoff documents written
- [ ] ~120 hours invested

---

## 📅 Phase 2: Architecture Deep Dive (Month 4–6)
### Goal: System design skills, micro-frontends, design systems, performance

---

### Month 4: Frontend System Design (40 hours)

#### Week 13-14: System Design Fundamentals (20 hours)

**Read (10 hours):**
- [ ] "Frontend System Design" — GreatFrontEnd (https://www.greatfrontend.com/system-design)
- [ ] "System Design for Frontend Engineers" by Evgenii Ray (book)
- [ ] Read: "Design Docs at Google" (how Google writes design documents)
- [ ] Read: "Scaling Frontend Architecture" series by Malte Ubl (ex-Google)
- [ ] Study these real-world architecture case studies:
  - How Netflix builds frontend (Netflix Tech Blog)
  - How Meta builds frontend (React + Relay architecture)
  - How Shopify moved to React from Rails
  - How Airbnb built their design system

**Practice (10 hours):**
- [ ] Design these systems (write full design documents, do NOT code):
  1. Design a real-time collaborative document editor (like Google Docs)
     - Component architecture
     - State management strategy
     - WebSocket vs polling
     - Conflict resolution approach
     - Performance at 100 concurrent editors
  2. Design a social media feed (like Twitter/X)
     - Infinite scroll implementation
     - Image lazy loading strategy
     - Caching strategy
     - Optimistic updates
     - Offline support approach
  3. Design an analytics dashboard (like Grafana)
     - Large dataset rendering
     - Chart library selection rationale
     - Real-time data updates
     - Filter/query state management
     - URL-driven state

**Design Document Structure to Follow:**
```markdown
# System Design: [Name]
## 1. Requirements
### Functional Requirements
### Non-Functional Requirements (performance, scale, a11y)
## 2. High-Level Architecture
### Component Tree (diagram)
### Data Flow (diagram)
## 3. API Design
### Endpoints needed
### Request/Response shapes
## 4. State Management
### Client state vs Server state
### Caching strategy
## 5. Core Components
### Component API design (props, events)
## 6. Performance Considerations
### Bundle size budget
### Rendering optimization
### Network optimization
## 7. Tradeoffs Made
### What we chose and why
### What we rejected and why
## 8. Future Considerations
```

#### Week 15-16: Design System Architecture (20 hours)

**Read (10 hours):**
- [ ] "Design Systems" by Alla Kholmatova (book)
- [ ] Radix UI source code — study the "headless UI" pattern
- [ ] Shadcn/ui architecture — study the "copy-paste" distribution model
- [ ] Material UI architecture — study the "full library" approach
- [ ] Chakra UI — study the "style props" approach
- [ ] Read: "The spectrum of design system approaches" (compare all 4 models above)
- [ ] Read: "Design Tokens" specification (W3C)

**Build (10 hours):**
- [ ] Project: Build a mini design system in your monorepo
  - 5 components: Button, Input, Select, Modal, Toast
  - Requirements:
    - TypeScript strict mode
    - Design tokens (colors, spacing, typography) as CSS custom properties
    - Compound component pattern (e.g., `<Select><Select.Option>`)
    - Polymorphic `as` prop (Button can render as `<a>` or `<button>`)
    - Full keyboard accessibility (WAI-ARIA)
    - Storybook for documentation
    - Unit tests with Testing Library
  - Write ADR: "Design System Architecture Decisions"
    - Why headless vs styled vs copy-paste?
    - Why CSS custom properties vs CSS-in-JS vs Tailwind?
    - How to version and distribute?

---

### Month 5: Micro-Frontend Architecture (40 hours)

#### Week 17-18: Micro-Frontend Theory + Module Federation (20 hours)

**Read (10 hours):**
- [ ] "Building Micro-Frontends" by Luca Mezzalira (book — the definitive guide)
- [ ] "Micro Frontends in Action" by Michael Geers (book)
- [ ] Webpack Module Federation docs + examples
- [ ] Read: "Micro-frontends at scale" — DAZN engineering blog
- [ ] Read: "Micro-frontends at IKEA" case study
- [ ] Read: "When NOT to use micro-frontends" (Martin Fowler's blog)
- [ ] Study: Module Federation architecture diagram — understand runtime dependency sharing

**Key Concepts to Master:**
```
1. Composition Patterns:
   - Build-time composition (npm packages)
   - Server-side composition (SSI, ESI)
   - Client-side composition (Module Federation, import maps)
   - Edge-side composition (Edge workers)

2. Communication Patterns:
   - Custom Events
   - Shared state (careful!)
   - URL/Route-based
   - Pub/Sub

3. Shared Dependencies:
   - Singleton packages (React must be shared)
   - Version mismatch handling
   - Fallback strategies

4. Anti-Patterns:
   - Sharing too much state
   - Nano-frontends (too granular)
   - Coupled deployment
   - Shared mutable state
```

**Build (10 hours):**
- [ ] Project: Module Federation proof-of-concept
  ```
  micro-frontend-demo/
  ├── shell/              (Host app — layout, routing, auth)
  ├── team-products/      (Remote — product listing, PDP)
  ├── team-cart/           (Remote — cart, checkout)
  └── shared/
      ├── design-system/   (Shared UI components)
      └── auth/            (Shared auth context)
  ```
  - Shell app loads remote apps at runtime
  - Each remote is independently deployable
  - Shared React and design system as singletons
  - Routing: shell owns top-level routes, remotes own sub-routes
  - Communication via Custom Events (NOT shared state)

#### Week 19-20: Alternative Approaches + Decision Framework (20 hours)

**Read (8 hours):**
- [ ] Single-SPA documentation + architecture
- [ ] Import Maps specification + SystemJS
- [ ] Native Federation (framework-agnostic Module Federation)
- [ ] Read: "Micro-frontends with Nx" — Nrwl blog
- [ ] Read: "Why we moved away from micro-frontends" — various post-mortems
- [ ] Read: "Modular Monolith" as an alternative pattern

**Build (12 hours):**
- [ ] Write a comprehensive tradeoff document:
  ```markdown
  # Micro-Frontend Decision Framework

  ## When to use micro-frontends:
  - Multiple autonomous teams (>3 teams, >15 engineers)
  - Different release cadences needed
  - Teams want technology independence
  - Legacy migration (strangler fig pattern)

  ## When NOT to use micro-frontends:
  - Single team
  - Small application
  - Consistent tech stack
  - Tight coupling between features

  ## Approach Comparison:
  | Criteria              | Module Fed | Single-SPA | npm packages | iframes |
  |-----------------------|-----------|------------|-------------|---------|
  | Runtime independence   | ✅         | ✅          | ❌           | ✅       |
  | Shared dependencies    | ✅         | ⚠️          | ✅           | ❌       |
  | Style isolation        | ❌         | ❌          | ❌           | ✅       |
  | Performance            | Good      | Good       | Best        | Worst   |
  | Complexity             | High      | High       | Low         | Low     |
  | Independent deploy     | ✅         | ✅          | ⚠️           | ✅       |
  ```
- [ ] Project: Rebuild one remote app as a Nuxt (Vue) app integrated with the React shell
  - This demonstrates the "technology-agnostic" benefit of micro-frontends
  - You'll hit real problems: CSS conflicts, shared state, routing

---

### Month 6: Performance Engineering (40 hours)

#### Week 21-22: Performance Fundamentals (20 hours)

**Read (10 hours):**
- [ ] web.dev "Performance" section — ALL articles
- [ ] "Web Performance in Action" by Jeremy Wagner (book)
- [ ] Read: "The Cost of JavaScript" by Addy Osmani
- [ ] Read: Chrome DevTools Performance documentation
- [ ] Study: Core Web Vitals in depth
  - LCP (Largest Contentful Paint) — what affects it, how to optimize
  - INP (Interaction to Next Paint) — replaces FID, what affects it
  - CLS (Cumulative Layout Shift) — common causes, fixes
- [ ] Read: "React Performance" by Ivan Akulov
- [ ] Read: "Virtual DOM is pure overhead" by Svelte creator Rich Harris

**Build (10 hours):**
- [ ] Project: Performance audit and optimization of your monorepo apps
  - Set up performance monitoring:
    - Lighthouse CI in CI/CD pipeline
    - Bundle analyzer (webpack-bundle-analyzer or vite-bundle-analyzer)
    - Real User Monitoring setup (web-vitals library)
  - Implement optimizations:
    - Code splitting with React.lazy + Suspense
    - Route-based code splitting
    - Image optimization (next/image or custom with srcset, WebP/AVIF)
    - Font optimization (font-display: swap, preload, subset)
    - Tree shaking verification
  - Document before/after metrics

#### Week 23-24: Advanced Performance Patterns (20 hours)

**Read (8 hours):**
- [ ] "Virtualization" — study react-window and react-virtuoso source code
- [ ] "Web Workers for computation offloading" — Comlink library
- [ ] "Service Workers and caching strategies" — Workbox documentation
- [ ] Read: "Streaming SSR" — how Next.js/React 19 streams HTML
- [ ] Read: "Progressive Hydration" and "Islands Architecture" (Astro's approach)
- [ ] Read: "Resumability" — Qwik framework's approach (know it exists, understand the tradeoff)

**Build (12 hours):**
- [ ] Project: Build a data-heavy dashboard that handles:
  - 10,000+ row table with virtualization
  - Real-time chart updates via WebSocket
  - Offline-capable with Service Worker caching
  - Web Worker for data transformation
  - Measure: Memory usage, frame rate, INP score
  - Write tradeoff document: "Performance optimization strategies and when to use each"

---

### 📊 Phase 2 Checkpoint — What You Should Have:
- [ ] 3 frontend system design documents (editor, feed, dashboard)
- [ ] A working design system in your monorepo
- [ ] A working micro-frontend proof-of-concept
- [ ] A micro-frontend decision framework document
- [ ] Performance optimization experience with measured results
- [ ] ~240 total hours invested
- [ ] 8+ tradeoff/design documents written

---

## 📅 Phase 3: Advanced Patterns + AI (Month 7–9)
### Goal: AI integration, advanced patterns, real-world project

---

### Month 7: AI for Frontend Engineers (40 hours)

#### Week 25-26: AI Fundamentals + Using AI Tools (20 hours)

**Read (10 hours):**
- [ ] "What are LLMs?" — understand tokens, context windows, temperature, streaming
- [ ] OpenAI API docs: Chat Completions, Streaming, Function Calling
- [ ] Anthropic API docs: Messages API, Streaming, Tool Use
- [ ] Vercel AI SDK documentation (the standard for AI + frontend)
- [ ] Read: "Building LLM-powered applications" (Vercel blog)
- [ ] Read: "Prompt Engineering Guide" — basic patterns
- [ ] Understand: RAG (Retrieval-Augmented Generation) at a high level

**Build (10 hours):**
- [ ] Project: AI-powered chat interface
  - React + Next.js + Vercel AI SDK
  - Features:
    - Streaming responses (show tokens as they arrive)
    - Markdown rendering in responses
    - Code syntax highlighting in responses
    - Conversation history management
    - Token usage display
    - Error handling and retry logic
    - Loading states and abort capability
  - This is a FAANG-relevant project — every company is building AI features

#### Week 27-28: AI-Powered Features + Architecture (20 hours)

**Read (8 hours):**
- [ ] Study: How Cursor IDE integrates AI (you're using it now!)
- [ ] Study: How GitHub Copilot works architecturally
- [ ] Read: "AI UX patterns" — loading states, streaming, error handling for AI
- [ ] Read: "Function Calling / Tool Use" — structured output from LLMs
- [ ] Read: "Edge computing for AI" — why inference at the edge matters

**Build (12 hours):**
- [ ] Project: AI-powered code review tool (add to your monorepo)
  - Upload a code file or paste code
  - AI analyzes: performance issues, accessibility issues, best practices
  - Streaming response with structured sections
  - Uses Function Calling to return structured data (not just text)
  - Implement: Caching (same code = same review), Rate limiting UI, Cost estimation display
  - Write ADR: "AI Integration Architecture Decisions"
    - API key management (server-side only, NEVER client-side)
    - Streaming vs. batch responses
    - Cost management strategies
    - Error handling patterns
    - Caching strategies for AI responses

---

### Month 8: Testing at Scale + Accessibility (40 hours)

#### Week 29-30: Testing Architecture (20 hours)

**Read (8 hours):**
- [ ] "Testing Trophy" by Kent C. Dodds — understand the philosophy
- [ ] Vitest docs (modern test runner, works with Vite)
- [ ] Testing Library philosophy — "test like a user"
- [ ] Playwright docs — E2E testing
- [ ] Read: "Testing micro-frontends" — integration testing strategies
- [ ] Read: "Visual Regression Testing" — Chromatic, Percy, or Playwright screenshots

**Build (12 hours):**
- [ ] Add comprehensive testing to your monorepo:
  - Unit tests: Vitest + Testing Library for components
  - Integration tests: Test component interactions and API calls
  - E2E tests: Playwright for critical user flows
  - Visual regression: Playwright screenshot comparison
  - Test in CI/CD: Run tests on PRs, block merge on failure
  - Coverage thresholds: Configure minimum coverage
  - Write ADR: "Testing Strategy"
    - What to test at each level
    - How much coverage is enough (hint: 100% is NOT the answer)
    - Testing shared packages vs apps
    - Mocking strategies

#### Week 31-32: Accessibility as Architecture (20 hours)

**Read (8 hours):**
- [ ] WAI-ARIA Authoring Practices 1.2 — study ALL component patterns
- [ ] "Inclusive Components" by Heydon Pickering (book)
- [ ] axe-core documentation — automated accessibility testing
- [ ] Read: "Accessibility at scale" — how large companies enforce a11y
- [ ] WCAG 2.2 AA guidelines — understand success criteria

**Build (12 hours):**
- [ ] Audit and fix your design system for accessibility:
  - Every component must pass axe-core automated checks
  - Keyboard navigation testing for all interactive components
  - Screen reader testing (use VoiceOver on Mac)
  - Focus management in Modal component
  - Announce Toast notifications to screen readers
  - Add axe-core to your CI/CD pipeline (fail builds on a11y violations)
  - Write document: "Accessibility Standards for our Design System"

---

### Month 9: Capstone Project (40 hours)

#### Week 33-36: Build a Complete Staff-Level Project (40 hours)

**This is your portfolio piece. It demonstrates every skill from the plan.**

**Project: Open-Source Micro-Frontend Starter Kit**

```
staff-project/
├── apps/
│   ├── shell/                 (Next.js host — routing, auth, layout)
│   ├── dashboard/             (React remote — analytics dashboard)
│   ├── ai-assistant/          (React remote — AI chat interface)
│   └── docs/                  (Documentation site — Astro or similar)
├── packages/
│   ├── design-system/         (Shared components with Storybook)
│   ├── ai-sdk/                (Shared AI integration utilities)
│   ├── testing-utils/         (Shared test helpers)
│   ├── config/                (Shared configs)
│   └── types/                 (Shared TypeScript types)
├── .github/
│   └── workflows/             (CI/CD pipelines)
├── docs/
│   ├── adrs/                  (Architecture Decision Records)
│   ├── system-design.md       (Overall system design document)
│   └── runbook.md             (How to develop, deploy, debug)
├── turbo.json
└── package.json
```

**Requirements:**
- [ ] Module Federation between shell and remotes
- [ ] Design system with 10+ accessible components
- [ ] AI chat feature with streaming responses
- [ ] Dashboard with virtualized data table + real-time charts
- [ ] Full CI/CD pipeline with preview deployments
- [ ] Performance budgets enforced in CI
- [ ] Accessibility checks in CI
- [ ] Bundle size tracking
- [ ] 5+ Architecture Decision Records
- [ ] System design document
- [ ] TypeScript strict mode throughout
- [ ] Comprehensive testing at all levels

---

### 📊 Phase 3 Checkpoint — What You Should Have:
- [ ] AI integration experience with real projects
- [ ] Complete testing strategy implemented
- [ ] Accessibility expertise with automated enforcement
- [ ] A capstone project that demonstrates Staff-level architecture skills
- [ ] ~360 total hours invested
- [ ] 12+ architecture/tradeoff documents

---

## 📅 Phase 4: Staff Behaviors + Interview Prep (Month 10–12)
### Goal: Think, communicate, and influence like a Staff Engineer

---

### Month 10: Communication + Influence (40 hours)

#### Week 37-38: Technical Writing (20 hours)

**Read (8 hours):**
- [ ] "Staff Engineer" by Will Larson (book — REQUIRED reading)
- [ ] "The Staff Engineer's Path" by Tanya Reilly (book — REQUIRED reading)
- [ ] "An Elegant Puzzle" by Will Larson (skim for tech leadership insights)
- [ ] Read: "How to write an RFC" — examples from Rust, React, and Ember communities
- [ ] Read: "Writing Technical Design Docs" by Malte Ubl

**Practice (12 hours):**
- [ ] Write 3 RFCs for your current company (even if not asked to):
  1. RFC: "Migrating from [current build tool] to Vite"
  2. RFC: "Introducing a shared design system"
  3. RFC: "Frontend testing strategy"
  - Follow this structure:
    ```markdown
    # RFC: [Title]
    ## Status: Draft
    ## Author: [You]
    ## Date: [Date]
    ## Summary (2-3 sentences)
    ## Motivation (Why now? What problem?)
    ## Detailed Design (How it works)
    ## Alternatives Considered (What else did you evaluate?)
    ## Migration Strategy (How do we get there from here?)
    ## Risks and Mitigations
    ## Open Questions
    ```
  - Share with your team for feedback (this IS Staff behavior)

#### Week 39-40: Mentorship + Cross-Team Influence (20 hours)

**Actions (these are not coding tasks — they are Staff Engineer behaviors):**
- [ ] Start a "Frontend Architecture" discussion group at your company (biweekly, 30 min)
- [ ] Present one technical topic per month to your team:
  - Month 10: "How micro-frontends work and when to use them"
  - Month 11: "Frontend performance optimization strategies"
  - Month 12: "AI integration patterns for frontend"
- [ ] Code review with architectural feedback (not just style nits):
  - "Have you considered the performance impact of this approach?"
  - "This component API doesn't compose well because..."
  - "Let's document this decision in an ADR because..."
- [ ] Mentor 1-2 junior/mid engineers — pair program weekly
- [ ] Start writing technical blog posts (even 1-2 posts helps):
  - "What I learned building a micro-frontend from scratch"
  - "Performance patterns that actually matter"

---

### Month 11: Frontend System Design Interview Prep (40 hours)

#### Week 41-42: System Design Practice (20 hours)

**Study (8 hours):**
- [ ] GreatFrontEnd System Design questions — practice ALL of them
- [ ] "Frontend Interview Handbook" — system design section
- [ ] Watch: "Frontend System Design" mock interviews on YouTube
- [ ] Study: How to draw architecture diagrams (use Excalidraw)

**Practice (12 hours):**
- [ ] Practice these system design problems (45 min each, timed):
  1. Design a Google Maps-like application
  2. Design a Figma-like collaborative whiteboard
  3. Design a Spotify-like music player
  4. Design a Gmail-like email client
  5. Design a Notion-like block editor
  6. Design an auto-complete/search component at scale
  7. Design a news feed with infinite scroll
  8. Design a real-time polling/voting system
  - For each: Write component tree, state management, API design, performance strategy
  - Time yourself — in interviews you get 35-45 minutes

#### Week 43-44: Coding Interview Prep (20 hours)

**This is specifically for FAANG frontend interviews:**
- [ ] JavaScript fundamentals (closures, prototypes, event loop, promises)
- [ ] DOM manipulation without frameworks
- [ ] Implement common utilities: debounce, throttle, deepClone, EventEmitter
- [ ] Implement UI components from scratch: Autocomplete, Modal, Carousel, Virtual List
- [ ] LeetCode Medium problems — focus on:
  - Arrays/Strings (most common in frontend rounds)
  - Trees (DOM is a tree)
  - BFS/DFS (common in UI problems)
  - Dynamic Programming (less common but asked at Staff level)
- [ ] Practice on: GreatFrontEnd, LeetCode, Frontend Masters

---

### Month 12: Mock Interviews + Final Preparation (40 hours)

#### Week 45-46: Mock Interviews (20 hours)

- [ ] Do 4-6 mock interviews with peers or paid platforms:
  - 2 system design rounds
  - 2 coding rounds (JavaScript/React)
  - 1 behavioral round (Staff-level leadership questions)
  - 1 architecture/deep dive round
- [ ] Platforms: Pramp (free), interviewing.io, or find peers on Blind/Discord
- [ ] After each mock: write down what went well, what didn't, improve

#### Week 47-48: Polish + Apply (20 hours)

- [ ] Update your resume for Staff level:
  - Lead with IMPACT, not tasks
  - Bad: "Built a design system using React"
  - Good: "Architected a shared design system adopted by 4 teams (15 engineers),
    reducing UI development time by 40% and eliminating 200+ duplicate components"
- [ ] Update LinkedIn with architecture-focused content
- [ ] Polish your capstone project — make the README excellent
- [ ] Publish 1-2 blog posts about your architectural learnings
- [ ] Start applying to FAANG companies for Staff Frontend roles
- [ ] Target companies: Meta, Google, Amazon, Apple, Netflix, Stripe, Vercel, Shopify

---

## 📊 Final Checkpoint — What You Should Have After 12 Months:

### Portfolio Artifacts:
- [ ] 1 capstone project (micro-frontend + AI + design system)
- [ ] 15+ Architecture Decision Records / Tradeoff documents
- [ ] 3 RFCs written (ideally shared with your team)
- [ ] 3 system design practice documents
- [ ] 1-2 published blog posts

### Technical Skills:
- [ ] React proficiency (on par with Vue)
- [ ] Next.js (SSR, SSG, ISR, App Router)
- [ ] TypeScript at an architectural level
- [ ] Micro-frontend architecture (Module Federation + alternatives)
- [ ] Monorepo management (Turborepo)
- [ ] Design system architecture and implementation
- [ ] Frontend performance optimization with measurable results
- [ ] AI integration (streaming, function calling, UX patterns)
- [ ] CI/CD pipeline design
- [ ] Testing architecture at all levels
- [ ] Accessibility as a first-class concern

### Staff Engineer Behaviors:
- [ ] Technical writing (RFCs, ADRs, design docs)
- [ ] Cross-team influence and communication
- [ ] Mentorship of junior/mid engineers
- [ ] Architectural decision-making with documented tradeoffs
- [ ] Tech talks / presentations

---

## 📚 Complete Reading List (Priority Order)

### Books (MUST READ):
1. "Staff Engineer" — Will Larson
2. "The Staff Engineer's Path" — Tanya Reilly
3. "Building Micro-Frontends" — Luca Mezzalira
4. "Effective TypeScript" — Dan Vanderkam
5. "Design Systems" — Alla Kholmatova

### Books (RECOMMENDED):
6. "Web Performance in Action" — Jeremy Wagner
7. "Micro Frontends in Action" — Michael Geers
8. "Inclusive Components" — Heydon Pickering

### Online Resources:
- patterns.dev (design patterns + rendering patterns)
- web.dev (performance + Core Web Vitals)
- react.dev (React official docs)
- greatfrontend.com (system design + interview prep)
- totalTypeScript.com (advanced TypeScript)

---

## ⏰ Weekly Schedule Template (10 hours/week)

```
Monday:     2 hours  — Reading (book/article)
Wednesday:  3 hours  — Building (hands-on project)
Friday:     3 hours  — Building (hands-on project)
Saturday:   2 hours  — Writing (tradeoff doc, ADR, or blog post)
```

---

## 🚨 Non-Negotiable Rules

1. **WRITE every week.** A Staff Engineer who can't write design docs isn't a Staff Engineer.
2. **BUILD every week.** Reading without building is useless.
3. **DOCUMENT tradeoffs.** Every technical decision you make — write down WHY.
4. **SHARE your work.** Present to your team. Write blog posts. Get feedback.
5. **Don't skip React.** FAANG requires it. No negotiation.
6. **Don't shortcut the capstone.** It's your proof of Staff-level capability.
7. **Talk to your manager.** Tell them you want to grow toward Staff. Get their input.

---

## 🎯 Success Metrics

After 12 months, you should be able to:

1. ✅ Design a frontend system from scratch in 45 minutes on a whiteboard
2. ✅ Write an RFC that your team can follow and implement
3. ✅ Evaluate micro-frontend vs monolith for any given team/product
4. ✅ Optimize a web app from 40 → 90+ Lighthouse score
5. ✅ Build an AI-powered feature with streaming and proper error handling
6. ✅ Set up a monorepo with shared packages, CI/CD, and preview deployments
7. ✅ Articulate tradeoffs between any two technical approaches with data
8. ✅ Pass a FAANG Staff Frontend interview loop

---

*"The difference between a Senior and a Staff Engineer is not what they can build —
it's what they can decide, communicate, and enable others to build."*

Good luck. This will be the hardest and most rewarding year of your career. 🚀
