# 🧭 The Staff Engineer's Tradeoff Guide — Part 2

## More Architecture Decisions + Framework Choices

---

# 📐 Tradeoff 7: Monorepo vs Polyrepo

### What They Are

```
POLYREPO (Multiple Repositories):
  repo-1: frontend-app
  repo-2: design-system
  repo-3: shared-utils
  repo-4: backend-api
  Each has its own git history, CI/CD, versioning

MONOREPO (Single Repository):
  one-repo/
  ├── apps/
  │   ├── frontend-app/
  │   ├── admin-app/
  │   └── docs-site/
  ├── packages/
  │   ├── design-system/
  │   ├── shared-utils/
  │   └── config/
  └── package.json
  ALL code in one repo, shared tooling, atomic changes
```

### The Decision Matrix

```
| Factor                     | Polyrepo        | Monorepo           |
|----------------------------|-----------------|--------------------|
| Atomic cross-project changes| ❌ Multiple PRs  | ✅ Single PR        |
| Code sharing               | ❌ Publish + install| ✅ Direct import |
| Dependency consistency     | ❌ Version drift | ✅ One version      |
| CI/CD complexity           | ✅ Simple per repo| ⚠️ Needs smart builds|
| Git history clarity        | ✅ Clean per project| ❌ Noisy          |
| Repository size            | ✅ Small         | ❌ Can grow large   |
| Access control             | ✅ Per-repo      | ⚠️ Needs CODEOWNERS |
| Onboarding                 | ✅ Clone what you need| ⚠️ Clone everything|
| Tooling overhead           | ✅ Standard git  | ⚠️ Need Nx/Turborepo|
| Refactoring across projects| ❌ Very hard     | ✅ Easy             |
| Build caching              | ❌ Per-repo      | ✅ Shared (Turbo/Nx)|
| Testing cross-project      | ❌ Integration CI| ✅ Run all tests    |
```

### Decision Tree

```
START: How many shared packages/libraries between projects?
│
├── 0-1 shared packages (projects are independent)
│   └── → POLYREPO (no benefit to monorepo)
│
├── 2-5 shared packages
│   │
│   ├── Team is < 10 engineers?
│   │   └── → MONOREPO (easier code sharing, atomic changes)
│   │
│   └── Teams are in different orgs / companies?
│       └── → POLYREPO (publish packages to npm registry)
│
├── 5+ shared packages, frequent cross-project changes
│   └── → MONOREPO (managing 5+ repos with version coordination is hell)
│
└── Migrating from polyrepo?
    └── → Start hybrid: Monorepo for new work + shared packages
          Gradually migrate existing repos when touching them

MONOREPO TOOLS:
  Turborepo  → Simpler, faster setup, Vercel ecosystem
  Nx         → More features, plugins for many frameworks, steeper learning curve

  Choose Turborepo if: Starting fresh, want simplicity
  Choose Nx if: Need framework-specific generators, larger team
```

### Anti-Patterns

```
❌ Monorepo without build caching
   → Without Turborepo/Nx caching, CI builds everything every time → slow.

❌ Monorepo with tight coupling between packages
   → Monorepo doesn't mean "one big app." Packages should have clean APIs.
   → If every change requires changing 5 packages, your boundaries are wrong.

❌ Polyrepo with "shared" npm packages that change weekly
   → Every change: bump version, publish, update consumers, test, repeat.
   → This is the #1 reason teams switch to monorepo.

❌ Using a monorepo but deploying everything together
   → The whole point is independent deployability. Use affected-only builds.
```

---

# 📐 Tradeoff 8: Testing Strategy

### The Testing Trophy (not Pyramid)

```
Traditional Testing Pyramid:        Testing Trophy (recommended):
         /  E2E  \                        /  E2E  \
        /─────────\                      /─────────\
       / Integration\                   / Integration\    ← MOST value here
      /───────────────\                /───────────────\
     /    Unit Tests    \             /  Unit (focused)  \
    /─────────────────────\          /───────────────────────\
   /    Static Analysis     \       /    Static Analysis       \
  ───────────────────────────      ─────────────────────────────

Key insight: Integration tests give the MOST confidence per effort.
Unit tests are cheap but test implementation, not behavior.
E2E tests are valuable but slow and flaky.
```

### The Decision Matrix

```
| Test Type      | Speed     | Confidence | Maintenance | What It Tests            |
|----------------|-----------|------------|-------------|--------------------------|
| Static (TS/ESLint) | ✅ Instant | ⚠️ Low      | ✅ Zero      | Types, syntax, patterns  |
| Unit           | ✅ Fast    | ⚠️ Low-Med  | ✅ Low       | Pure logic, utilities    |
| Integration    | ⚠️ Medium  | ✅ High     | ⚠️ Medium    | Components + interactions|
| E2E            | ❌ Slow    | ✅ Highest  | ❌ High      | Full user flows          |
| Visual Regress.| ⚠️ Medium  | ✅ High     | ⚠️ Medium    | UI appearance changes    |
```

### What to Test at Each Level

```
STATIC ANALYSIS (TypeScript + ESLint):
  → Catches: type errors, unused variables, import errors
  → Cost: Nearly zero (runs in IDE)
  → ALWAYS have this. No excuse not to.

UNIT TESTS:
  → Test: Pure functions, utilities, hooks (no DOM)
  → Don't test: Component rendering (that's integration)
  → Don't test: Implementation details (internal state)
  → Examples:
    ✅ formatCurrency(1234.5) → "$1,234.50"
    ✅ calculateTotal(items) → 150
    ✅ debounce function behavior
    ❌ "component sets state to X" (implementation detail)
    ❌ "function calls Y internally" (implementation detail)

INTEGRATION TESTS (this is where you invest most):
  → Test: Components rendering + user interactions + API responses
  → Tool: Testing Library + Vitest/Jest (test like a user)
  → Examples:
    ✅ "User types in search → sees results → clicks result → navigates"
    ✅ "User fills form → submits → sees success message"
    ✅ "Component shows loading state → then data → handles error"
    ❌ "Component renders 3 divs and 2 spans" (too coupled to markup)

E2E TESTS:
  → Test: Critical user flows through the REAL app
  → Tool: Playwright (recommended) or Cypress
  → Scope: Only the most important flows (5-15 tests)
  → Examples:
    ✅ Login → Browse → Add to Cart → Checkout → Confirmation
    ✅ Sign Up → Email Verification → First-time Setup
    ✅ Search → Filter → Sort → View Product
    ❌ Every possible edge case (too slow, too flaky)

VISUAL REGRESSION:
  → Test: UI hasn't changed unexpectedly
  → Tool: Playwright screenshots, Chromatic, Percy
  → Scope: Design system components + key pages
```

### How Much Testing Is Enough?

```
╔═══════════════════════════════════════════════════════════════╗
║  100% code coverage is NOT the goal.                         ║
║  100% confidence in critical paths IS the goal.              ║
╚═══════════════════════════════════════════════════════════════╝

Recommended coverage targets:
  Shared packages (design system, utils): 80-90% unit
  Features/pages: Focus on integration tests (no coverage target)
  Critical user flows: 100% E2E coverage

Time allocation:
  Static analysis: 0% ongoing effort (set up once, runs automatically)
  Unit tests: 20% of test effort
  Integration tests: 50% of test effort  ← MOST effort here
  E2E tests: 20% of test effort
  Visual regression: 10% of test effort

In CI/CD:
  PR check: Static + Unit + Integration (fast, blocking)
  Merge to main: E2E + Visual Regression (slow, blocking)
  Nightly: Full E2E suite (comprehensive, alerting)
```

---

# 📐 Tradeoff 9: Build vs Buy (Libraries vs Custom)

### The Framework

```
For every library/tool decision, ask:

1. DOES IT SOLVE A REAL PROBLEM?
   "Is this a problem we actually have, or might have someday?"
   → Only add dependencies for CURRENT problems, not imagined future ones.

2. WHAT'S THE TOTAL COST OF OWNERSHIP?
   Build cost = Development time + Testing + Documentation + Maintenance
   Buy cost = Bundle size + Learning curve + Dependency risk + Upgrade cost

3. IS IT A CORE DIFFERENTIATOR?
   → If this feature is what makes your product special → BUILD
   → If this is commodity functionality → BUY

4. HOW STABLE IS THE LIBRARY?
   → Check: GitHub stars, last commit, issue response time, downloads/week
   → Check: Who maintains it? (solo dev vs company vs foundation)
   → Check: License (MIT = safe, GPL = careful, custom = read carefully)
```

### Decision Matrix by Category

```
ALWAYS BUY (don't build these):
┌─────────────────────────────────────────────────────────────┐
│ Category           │ Recommended Library                    │
│───────────────────────────────────────────────────────────── │
│ Date handling      │ date-fns (tree-shakeable, immutable)   │
│ Form validation    │ Zod (schema validation, TypeScript)    │
│ HTTP client        │ fetch (built-in) + wrapper             │
│ Routing            │ Framework router (Next/Nuxt/React Router)│
│ Testing            │ Vitest + Testing Library + Playwright  │
│ Linting            │ ESLint + Prettier                      │
│ Animation          │ Framer Motion or CSS (depends on need) │
│ Charts             │ Recharts, Visx, or Chart.js            │
│ Rich text editor   │ Tiptap, Plate, or Lexical              │
│ Auth               │ Clerk, Auth0, NextAuth (don't roll your own)│
│ Payments           │ Stripe (always)                        │
└─────────────────────────────────────────────────────────────┘

EVALUATE CAREFULLY (could go either way):
┌─────────────────────────────────────────────────────────────┐
│ Category           │ Build if...        │ Buy if...         │
│─────────────────────────────────────────────────────────────│
│ Design system      │ Unique brand,      │ Speed > brand,    │
│                    │ many teams         │ small team        │
│                    │                    │ (use shadcn/ui)   │
│ State management   │ Simple needs       │ Complex needs     │
│                    │ (context/reducer)  │ (Zustand/Redux)   │
│ Data fetching      │ Very simple app    │ Most apps         │
│                    │ (raw fetch)        │ (TanStack Query)  │
│ i18n               │ < 3 languages      │ 3+ languages      │
│                    │ (simple JSON)      │ (i18next)         │
│ Image optimization │ Simple img tags    │ At scale          │
│                    │ (srcset/loading)   │ (next/image,CDN)  │
└─────────────────────────────────────────────────────────────┘

USUALLY BUILD (custom is better):
┌─────────────────────────────────────────────────────────────┐
│ Category                │ Why build?                        │
│─────────────────────────────────────────────────────────────│
│ Fetch wrapper/API layer │ Your auth, error handling, retry  │
│ Layout components       │ Specific to your app's navigation │
│ Feature flags UI        │ Specific to your flag system      │
│ Analytics integration   │ Specific to your tracking needs   │
│ Business logic          │ ALWAYS custom — this IS your app  │
│ Domain-specific components │ These are your product         │
└─────────────────────────────────────────────────────────────┘
```

### Bundle Size Awareness

```
Before adding ANY library, check its size:
  → https://bundlephobia.com/

Common offenders:
  moment.js:          300 KB → Replace with: date-fns (tree-shake) or dayjs (2KB)
  lodash:              70 KB → Replace with: lodash-es (tree-shake) or native JS
  axios:               13 KB → Replace with: fetch (built-in, 0KB)
  Material UI:        300 KB+→ Consider: Tailwind + Headless components
  Chart.js:            60 KB → OK if you need charts, consider Recharts

Rule of thumb:
  If a library adds > 20KB to your bundle, you need a good reason.
  If a library adds > 50KB, you need a VERY good reason.
  If you can write it in < 50 lines, don't add a dependency.
```

### Anti-Patterns

```
❌ "Let's build our own auth system"
   → Authentication is incredibly complex (CSRF, session management,
     OAuth, MFA, password hashing). Use Auth0, Clerk, or NextAuth.
     One security bug in your custom auth = user data breach.

❌ "Let's build our own rich text editor"
   → Text editors are the hardest frontend problem. ContentEditable is
     a nightmare. Use Tiptap, Lexical, or Plate. Companies have spent
     years building these.

❌ Adding a library for something JavaScript can do natively
   → classnames (2KB) when you can use template literals
   → uuid when crypto.randomUUID() exists
   → is-number (yes, this npm package exists) when typeof x === 'number'

❌ Not evaluating library maintenance risk
   → Check: Is it maintained by one person? (bus factor = 1)
   → Check: When was the last release? (> 1 year = risk)
   → Check: How many open issues with no response? (abandoned?)

❌ "We'll replace it later if needed"
   → Migration cost is ALWAYS higher than you think.
   → Choose carefully upfront. Migrations are never prioritized.
```

---

# 📐 Tradeoff 10: Framework Selection (React vs Vue vs Angular vs Others)

### The Honest Matrix

```
| Factor                 | React      | Vue 3      | Angular    | Svelte/Kit  | Solid      |
|------------------------|------------|------------|------------|-------------|------------|
| FAANG Adoption         | ✅ Dominant | ⚠️ Some     | ⚠️ Google   | ❌ Rare      | ❌ Rare     |
| Job Market (2026)      | ✅ Largest  | ✅ Strong   | ⚠️ Declining| ⚠️ Growing   | ❌ Small    |
| Learning Curve         | ⚠️ Medium   | ✅ Low      | ❌ Steep    | ✅ Low       | ⚠️ Medium   |
| Ecosystem              | ✅ Largest  | ✅ Large    | ✅ Full     | ⚠️ Growing   | ⚠️ Small    |
| Performance (runtime)  | ⚠️ VDOM     | ⚠️ VDOM     | ⚠️ VDOM     | ✅ No VDOM   | ✅ No VDOM  |
| Bundle Size            | ⚠️ 40KB     | ✅ 33KB     | ❌ 65KB     | ✅ ~2KB      | ✅ ~7KB     |
| TypeScript Support     | ✅ Excellent | ✅ Excellent | ✅ Built-in | ✅ Built-in  | ✅ Good     |
| SSR/Meta-framework     | Next.js    | Nuxt 3     | Angular SSR| SvelteKit   | SolidStart |
| Corporate Backing      | Meta       | Independent| Google     | Vercel      | Independent|
| Component Model        | Functions  | SFC+Comp   | Classes    | Components  | Functions  |
| State Management       | External   | Built-in   | Built-in   | Built-in    | Signals    |
| Hiring Pool            | ✅ Largest  | ⚠️ Moderate | ⚠️ Moderate | ❌ Small     | ❌ Tiny     |
```

### Decision Tree

```
START: What's the primary constraint?
│
├── TARGETING FAANG?
│   └── → REACT (non-negotiable for most FAANG companies)
│         Meta = React, Netflix = React, Airbnb = React, Uber = React
│         Google = Angular internally, but React for some teams
│         Amazon = React for most consumer-facing
│
├── EXISTING TEAM EXPERTISE?
│   │
│   ├── Team knows Vue deeply (like you, Ganesh)
│   │   │
│   │   ├── Building internal tools / staying at current company?
│   │   │   └── → VUE (leverage existing expertise)
│   │   │
│   │   └── Need React for career growth / FAANG target?
│   │       └── → LEARN REACT (Vue knowledge transfers well)
│   │             Vue Composition API ≈ React Hooks
│   │             Vue SFC ≈ React Components
│   │             Pinia ≈ Zustand
│   │             Nuxt ≈ Next.js
│   │
│   └── Team knows Angular
│       └── → ANGULAR if enterprise / staying
│           → REACT if moving to product companies
│
├── NEW PROJECT, GREENFIELD?
│   │
│   ├── Need largest ecosystem + hiring pool?
│   │   └── → REACT
│   │
│   ├── Want best DX + fastest onboarding?
│   │   └── → VUE 3 or SVELTE
│   │
│   ├── Enterprise + need everything included (CLI, testing, forms)?
│   │   └── → ANGULAR (opinionated, batteries-included)
│   │
│   └── Want best performance + smallest bundle?
│       └── → SVELTE or SOLID
│
└── SPECIFIC FOR YOU (Ganesh):
    You know Vue. Your target is FAANG Staff Frontend.
    → LEARN REACT. It's in your 12-month plan (Month 1).
    → Keep Vue as your secondary framework.
    → Understanding BOTH gives you architectural perspective
      that single-framework engineers lack.
    → In Staff interviews, knowing multiple frameworks shows depth.
```

### The Hard Truth About Frameworks

```
╔═══════════════════════════════════════════════════════════════╗
║  At Staff level, the framework matters LESS than you think.  ║
║                                                               ║
║  What matters MORE:                                           ║
║  - Understanding the web platform (your 8-week fundamentals) ║
║  - System design skills                                       ║
║  - Architecture decisions and tradeoffs                       ║
║  - Performance optimization                                   ║
║  - Communication and influence                                ║
║                                                               ║
║  A Staff Engineer who deeply understands React can learn Vue  ║
║  in 2 weeks. The reverse is also true.                        ║
║                                                               ║
║  Frameworks change every 3-5 years.                           ║
║  Web fundamentals haven't changed in 20 years.                ║
╚═══════════════════════════════════════════════════════════════╝
```

---

# 📐 Tradeoff 11: Design System Approach

```
| Approach             | Examples          | Pros                        | Cons                        |
|----------------------|-------------------|-----------------------------|-----------------------------|
| Full Library         | MUI, Chakra, Ant  | Fast start, comprehensive   | Hard to customize, large    |
| Headless + Style     | Radix + Tailwind  | Full control, accessible    | More work to style          |
| Copy-Paste           | shadcn/ui         | Own the code, customizable  | Manual updates              |
| Custom from Scratch  | Your own          | Perfect fit, full control   | Massive effort, a11y hard   |
```

### Decision Tree

```
START: Do you have a dedicated design system team?
│
├── NO (most companies < 50 engineers)
│   │
│   ├── Need to ship fast? (startup, MVP)
│   │   └── → shadcn/ui (copy-paste, own the code, Tailwind-based)
│   │         or MUI / Chakra (full library, instant components)
│   │
│   └── Need unique brand? (consumer product)
│       └── → Radix + Tailwind (headless a11y + custom styles)
│             You get accessible behavior. You control appearance.
│
├── YES (design system team exists)
│   │
│   ├── Serving multiple frameworks? (React + Vue + Mobile)
│   │   └── → Web Components or Headless (framework-agnostic)
│   │
│   └── Single framework?
│       └── → Custom components built on Radix primitives
│             Maximum control, accessibility included
│
└── MIGRATING from existing UI library?
    └── → Incremental: Wrap old components, replace one by one
          Don't big-bang migrate — it never works.
```

---

# 📐 Tradeoff 12: TypeScript Strictness

```
| Level              | Config                        | Catches           | Cost          |
|--------------------|-------------------------------|--------------------|----|
| Off                | No TypeScript                 | Nothing            | None          |
| Loose              | strict: false                 | Basic type errors  | Low           |
| Standard           | strict: true                  | Most type errors   | Medium        |
| Maximum            | strict + noUncheckedIndexed   | Edge cases         | High          |
|                    | Access + exactOptionalProperty|                    |               |
|                    | Types + noPropertyAccessFromIndex |                |               |

RECOMMENDATION: strict: true for ALL new projects.
  → Catches bugs at compile time, not runtime.
  → IntelliSense is dramatically better.
  → Refactoring is safe (compiler finds broken references).
  → The "cost" is upfront; the savings are forever.

  If migrating existing JS codebase:
  → Enable strict incrementally (per-file with // @ts-strict)
  → Or use 'any' as temporary escape hatch, track and reduce over time
```

---

# 📐 Tradeoff 13: Authentication Architecture (SPA)

```
| Approach                    | Security    | Complexity  | UX          |
|-----------------------------|-------------|-------------|-------------|
| Session Cookie (traditional)| ✅ High      | ✅ Simple    | ✅ Seamless  |
| JWT in localStorage         | ❌ XSS risk  | ✅ Simple    | ✅ Seamless  |
| JWT in memory + refresh cookie| ✅ High    | ⚠️ Medium    | ✅ Seamless  |
| OAuth + BFF (Backend for Frontend) | ✅ Highest | ❌ Complex | ✅ Seamless |

RECOMMENDATION for SPAs:
  → Access token: Store in JavaScript memory (variable)
  → Refresh token: HttpOnly, Secure, SameSite=Strict cookie
  → On page reload: Call /auth/refresh to get new access token
  → This is the industry-standard secure SPA auth pattern.

  For most apps: Use Auth0, Clerk, or NextAuth (don't build custom auth).
```

---

# 📐 Tradeoff 14: Deployment Strategy

```
| Strategy          | Downtime  | Rollback    | Risk         | Complexity    |
|-------------------|-----------|-------------|--------------|---------------|
| Big Bang          | ❌ Yes     | ❌ Hard      | ❌ High       | ✅ Simple      |
| Blue-Green        | ✅ Zero    | ✅ Instant   | ⚠️ Medium     | ⚠️ Medium      |
| Canary            | ✅ Zero    | ✅ Fast      | ✅ Low        | ❌ Complex     |
| Feature Flags     | ✅ Zero    | ✅ Instant   | ✅ Lowest     | ⚠️ Medium      |
| Rolling           | ✅ Zero    | ⚠️ Gradual   | ⚠️ Medium     | ⚠️ Medium      |

RECOMMENDATION:
  Small team / startup → Feature flags (LaunchDarkly, Unleash, or custom)
  → Deploy code to production with flag OFF
  → Enable flag for internal users → test
  → Roll out to 5% → 25% → 50% → 100%
  → If broken: turn off flag instantly (no rollback needed)

  This is what FAANG companies do. You should learn this pattern.
```

---

# 📐 Tradeoff 15: Performance vs Developer Experience

```
This is the META-TRADEOFF that affects every other decision.

EXAMPLES:
  TypeScript vs JavaScript
    → TS: Slower to write initially, faster to maintain
    → JS: Faster to start, more runtime bugs
    → VERDICT: TypeScript wins for any project > 2 weeks

  SSR vs CSR
    → SSR: Better performance, more infrastructure complexity
    → CSR: Simpler development, worse initial load
    → VERDICT: Depends on SEO needs (see Tradeoff 1)

  Micro-frontend vs Monolith
    → MF: Team independence, more infrastructure
    → Monolith: Simple, but coupling grows
    → VERDICT: Start monolith, extract when needed (see Tradeoff 3)

  Custom components vs UI library
    → Custom: Perfect fit, slow to build
    → Library: Fast start, hard to customize
    → VERDICT: Use headless primitives + custom styles

THE RULE OF THUMB:
  Short-term project (< 3 months): Optimize for DX (ship fast)
  Long-term project (> 1 year): Optimize for maintainability and performance

  At FAANG scale: Performance and maintainability ALWAYS win.
  You can never go back and add performance. You CAN always improve DX with tooling.
```

---

# 🎯 The Staff Engineer's Tradeoff Cheat Sheet

## Quick Reference: "What Should I Use?"

```
Rendering:     Marketing site → SSG | Dashboard → CSR | E-commerce → ISR | Dynamic → SSR
Architecture:  < 10 devs → Monolith | 10-30 → Modular Monolith | 30+ autonomous → Micro-FE
API:           Same TS repo → tRPC | Complex UI → GraphQL | Simple/Public → REST
State:         Server data → TanStack Query | Simple client → Zustand | Complex → Redux TK
CSS:           Most apps → Tailwind | Design system → Vanilla Extract | Quick → CSS Modules
Repo:          Shared packages → Monorepo | Independent projects → Polyrepo
Testing:       Static + Integration heavy + Selective E2E (Testing Trophy)
Libraries:     Auth → Buy | Editor → Buy | Business logic → Build | Utilities → Evaluate
Framework:     FAANG target → React | Current company → Vue | Enterprise → Angular
Design System: Fast → shadcn/ui | Custom brand → Radix + Tailwind | Enterprise → Build
TypeScript:    ALWAYS strict:true
Auth:          Access token in memory + Refresh in HttpOnly cookie
Deployment:    Feature flags + Canary rollout
```

## The 5 Rules of Tradeoff Thinking

```
RULE 1: "It depends" is always the correct starting point.
        But "it depends" is never the complete answer.
        Follow up with: "Here's my decision framework..."

RULE 2: The BEST solution is the one your team can MAINTAIN.
        A perfect architecture that nobody understands is worse
        than a good architecture that everyone can contribute to.

RULE 3: Optimize for the constraint that ACTUALLY matters.
        If SEO doesn't matter, don't add SSR complexity.
        If you have 3 engineers, don't build micro-frontends.

RULE 4: Document EVERY significant decision.
        Write an ADR. Explain what you chose, why, and what you gave up.
        Your future self (and your team) will thank you.

RULE 5: Decisions are reversible — but some are more expensive to reverse.
        Framework choice: Very expensive to reverse (2-year migration)
        CSS approach: Moderately expensive (gradual migration possible)
        State management: Low-medium (can swap with adapter pattern)
        Component library: Medium (new components in new lib, migrate old ones)

        For expensive decisions: Take more time upfront.
        For cheap decisions: Pick one and move fast.
```

---

## 📝 Practice Exercise: Apply Tradeoffs to a Real Scenario

### Scenario: You're the Staff Frontend Engineer at a Series B startup.

```
Context:
- E-commerce platform (clothes, 50K products)
- 12 frontend engineers, 3 teams
- Currently: Vue 2 monolith, REST API, Vuex, vanilla CSS
- Problems: Slow build times (15 min), inconsistent UI, SEO is poor
- Budget: 6 months to modernize

YOUR TASK: Make these decisions with documented tradeoffs:
1. Rendering strategy for product pages? (SEO critical)
2. Rendering strategy for user dashboard? (behind auth)
3. Vue 2 → Vue 3 or Vue 2 → React?
4. Monolith → Modular monolith or Micro-frontend?
5. Vuex → what?
6. CSS approach?
7. Monorepo or polyrepo?
8. Testing strategy?
9. Design system approach?
10. Deployment strategy?

Write an ADR for each decision.
This is EXACTLY the kind of exercise FAANG asks in Staff interviews.
```

---

*This tradeoff guide + your 8-week web fundamentals + 12-month Staff plan
= comprehensive preparation for Staff Frontend Engineer at FAANG.*

*Good luck, Ganesh. The difference between Senior and Staff is not knowing
the answers — it's knowing HOW TO DECIDE.* 🎯
