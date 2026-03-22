# 🌐 Web Fundamentals Deep Dive — Master Plan

## Goal: Staff-level web platform knowledge + FAANG interview readiness
## Duration: 8 weeks (separate from 12-month Staff plan)
## Time: 10 hours/week | Total: ~80 hours
## Style: Reading + Building +  Practice

---

## ⚠️ How to Use This Plan

- This is a **standalone deep dive** — do this BEFORE or in PARALLEL with your 12-month plan
- Each part is a **self-contained module** with reading, building, and interview practice
- Every part ends with **"Interview Questions You Must Answer"** — if you can't answer them, re-study
- Build the mini-projects — reading without doing is wasted time

---

## 📂 Parts Overview

| Part | File | Topic | Time | Week |
|------|------|-------|------|------|
| 1 | `01-how-browser-works.md` | Browser rendering pipeline, parsing, paint, composite | 10h | Week 1 |
| 2 | `02-javascript-engine.md` | V8 internals, event loop, memory, garbage collection | 10h | Week 2 |
| 3 | `03-core-javascript.md` | Closures, prototypes, `this`, promises, generators, proxies | 10h | Week 3 |
| 4 | `04-networking.md` | HTTP/1.1 vs 2 vs 3, DNS, TCP, TLS, CDN, caching headers | 10h | Week 4 |
| 5 | `05-dom-and-css.md` | DOM APIs, CSS internals, layout algorithms, stacking context | 10h | Week 5 |
| 6 | `06-web-platform-apis.md` | Service Workers, Web Workers, WebSockets, Streams API | 10h | Week 6 |
| 7 | `07-security-and-storage.md` | CORS, CSP, XSS, CSRF, cookies, IndexedDB, Cache API | 10h | Week 7 |
| 8 | `08-performance.md` | Core Web Vitals, critical rendering path, resource loading | 10h | Week 8 |

---

## 🔗 How Parts Connect

```
Week 1: Browser Rendering ──→ You understand WHAT the browser does
Week 2: JS Engine ──────────→ You understand HOW JavaScript executes
Week 3: Core JavaScript ────→ You master the LANGUAGE deeply
Week 4: Networking ─────────→ You understand how DATA reaches the browser
Week 5: DOM + CSS ──────────→ You understand the PLATFORM APIs
Week 6: Web Platform APIs ──→ You understand ADVANCED browser capabilities
Week 7: Security + Storage ─→ You understand PROTECTION and PERSISTENCE
Week 8: Performance ────────→ You tie EVERYTHING together for optimization
```

Each week builds on the previous. Do NOT skip ahead.

---

## 📊 Self-Assessment Tracker

After each part, rate yourself honestly:

| Part | Before (1-5) | After (1-5) | Can answer all interview Qs? |
|------|-------------|-------------|------------------------------|
| 1. Browser Rendering | | | ☐ Yes ☐ No |
| 2. JS Engine | | | ☐ Yes ☐ No |
| 3. Core JavaScript | | | ☐ Yes ☐ No |
| 4. Networking | | | ☐ Yes ☐ No |
| 5. DOM + CSS | | | ☐ Yes ☐ No |
| 6. Web Platform APIs | | | ☐ Yes ☐ No |
| 7. Security + Storage | | | ☐ Yes ☐ No |
| 8. Performance | | | ☐ Yes ☐ No |

---

## ⏰ Weekly Schedule (10 hours)

```
Monday:     2h — Reading (articles, specs, docs)
Tuesday:    1h — Reading (continue + take notes)
Wednesday:  3h — Building (mini-project for that week)
Thursday:   2h — Building (finish + experiment)
Saturday:   2h — Interview practice (answer questions, explain aloud)
```

---

## 📚 Core Resources (Used Across All Parts)

### Websites:
- web.dev (Google's web platform docs)
- developer.mozilla.org (MDN — the reference)
- javascript.info (best JS tutorial)
- whatwg.org (HTML/DOM specifications)

### Books:
- "JavaScript: The Definitive Guide" — David Flanagan (reference)
- "High Performance Browser Networking" — Ilya Grigorik (FREE online: hpbn.co)
- "Web Performance in Action" — Jeremy Wagner

### Tools You'll Use:
- Chrome DevTools (Performance, Network, Application, Memory tabs)
- Wireshark (networking, optional but powerful)
- about:blank + vanilla JS (no frameworks — raw web platform)

---

Start with `01-how-browser-works.md` →
