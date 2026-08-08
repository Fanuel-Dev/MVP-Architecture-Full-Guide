# 🚀 MVP Architecture: The Full Guide

<p align="center">
  <img src="assets/mvp-architecture-diagram.svg" alt="MVP Architecture Diagram" width="100%"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/status-active-brightgreen" alt="status"/>
  <img src="https://img.shields.io/badge/license-MIT-blue" alt="license"/>
  <img src="https://img.shields.io/badge/PRs-welcome-orange" alt="PRs welcome"/>
  <img src="https://img.shields.io/badge/made%20with-%E2%9D%A4-red" alt="made with love"/>
</p>

<p align="center">
  A practical, no-fluff guide to designing and shipping <b>Minimum Viable Product</b> architectures —
  for web apps, mobile apps, marketplaces, SaaS, and AI products.
</p>

---

## 📑 Table of Contents

- [Why This Guide](#-why-this-guide)
- [Core Principles](#-core-principles)
- [Typical MVP Stack](#-typical-mvp-stack-layer-by-layer)
- [Architecture Overview](#-architecture-overview)
- [Step-by-Step Build Process](#-step-by-step-build-process)
- [What to Deliberately Skip](#-what-to-deliberately-skip)
- [Stack by Product Type](#-stack-by-product-type)
  - [Web App](#1-web-app-mvp)
  - [Mobile App](#2-mobile-app-mvp)
  - [Marketplace](#3-marketplace-mvp-two-sided-buyers--sellers)
  - [SaaS](#4-saas-mvp)
  - [AI Product](#5-ai-product-mvp)
- [Quick Comparison Table](#-quick-comparison-table)
- [When to Evolve the Architecture](#-when-to-evolve-the-architecture)
- [Common Mistakes](#-common-mistakes)
- [Contributing](#-contributing)
- [License](#-license)

---

## 💡 Why This Guide

Most MVPs don't fail because of bad architecture — they fail because teams **over-build** before they've validated anything. This guide exists to help you ship the smallest, most honest version of your product without painting yourself into a technical corner.

> Your architecture should serve the hypothesis, not "best practices" for a company you don't have yet.

---

## 🧭 Core Principles

| Principle | Why It Matters |
|---|---|
| **Optimize for iteration speed, not scale** | You don't need Kubernetes for 50 users |
| **Monolith first** | Easier to build, debug, and change than microservices |
| **Use boring, proven technology** | Save creative risk for the product, not the infra |
| **Buy, don't build** | Auth, payments, email, hosting — use managed services |
| **Make it easy to throw away** | Don't over-abstract for a future that may not come |

---

## 🏗 Typical MVP Stack (Layer by Layer)

| Layer | Recommended Choices |
|---|---|
| **Frontend (Web)** | React/Next.js, Vue/Nuxt, or server-rendered templates |
| **Frontend (Mobile)** | React Native or Flutter (native only if performance is core) |
| **Backend** | Node/Express, Django, Rails, or FastAPI — one monolith |
| **API Style** | REST by default; GraphQL only with a concrete reason |
| **Database** | PostgreSQL or MySQL |
| **Caching/Queues** | Redis — only once there's a real need |
| **Auth** | Auth0, Clerk, Supabase Auth, Firebase Auth |
| **Hosting** | Vercel, Render, Railway, Fly.io, Heroku |
| **File Storage** | S3 or equivalent object storage |
| **Payments** | Stripe (never build your own) |

---

## 🗺 Architecture Overview

```
[Client: Web/Mobile]
        |
        v
[Load Balancer / CDN]   <-- often handled by your PaaS
        |
        v
[Monolithic API Server] --- [Auth Provider]
        |
        +----> [PostgreSQL DB]
        |
        +----> [Object Storage (S3)]
        |
        +----> [Third-party APIs: payments, email, etc.]
```

No message queues, no microservices, no service mesh — unless your hypothesis specifically requires them (e.g., real-time collaboration needs WebSockets/Pusher/Ably).

---

## 🔨 Step-by-Step Build Process

1. **Write down the core user flow** (2–5 screens/steps max) that proves the hypothesis
2. **Design the data model** — only the tables/entities needed for that flow
3. **Pick the stack** based on team familiarity first, ecosystem maturity second
4. **Stand up auth and hosting** before writing feature code
5. **Build the thinnest vertical slice** — one full flow end-to-end, even if ugly
6. **Instrument analytics from day one** (PostHog, Mixpanel, or simple logging)
7. **Deploy continuously** — even a simple `git push` to your PaaS
8. **Set a hard "ugly is fine" rule** for anything outside the core flow

---

## ⛔ What to Deliberately Skip

- ❌ Microservices
- ❌ Kubernetes / container orchestration
- ❌ Custom auth/session systems
- ❌ Elaborate CI/CD pipelines
- ❌ Horizontal scaling infrastructure
- ❌ Complex caching layers
- ❌ Multi-region deployment
- ❌ 100% automated test coverage (test core flows, not everything)

---

## 📦 Stack by Product Type

### 1. Web App MVP

**Core need:** Fast iteration, SEO (maybe), simple deployment

- **Frontend:** Next.js / Nuxt / SvelteKit
- **Backend:** Same framework's API routes, or separate Node/Django/Rails API
- **DB:** PostgreSQL
- **Auth:** Clerk / Auth0 / Supabase Auth
- **Hosting:** Vercel / Netlify / Render
- **Storage:** S3 or Cloudflare R2

```
[Browser] -> [Next.js Frontend + API routes] -> [PostgreSQL]
                        |
                        +--> [Auth Provider]
                        +--> [S3/R2 for files]
```

**Priorities:** page load speed, mobile-responsive UI, clean core flow.
**Skip:** SSR everywhere, Redux-style state management until you actually need it.

---

### 2. Mobile App MVP

**Core need:** Fast cross-platform build, offline tolerance, app store approval cycle

- **Frontend:** React Native (Expo) or Flutter
- **Backend:** Shared REST API (reuse with web if applicable)
- **DB:** PostgreSQL
- **Auth:** Firebase Auth / Supabase Auth
- **Push:** Firebase Cloud Messaging / OneSignal
- **Hosting:** Render/Fly.io backend + Expo EAS for builds

```
[iOS/Android App] -> [REST API] -> [PostgreSQL]
        |                  |
        |                  +--> [Auth Provider]
        +--> [Push Notification Service]
```

**Priorities:** offline-first UX, smooth onboarding, analytics from day one.
**Skip:** native modules, custom animations, platform-specific features unless core.

---

### 3. Marketplace MVP (two-sided: buyers + sellers)

**Core need:** Matching, trust/reputation, split payments/escrow

- **Frontend:** Next.js (web) and/or React Native
- **Backend:** Monolith modeled around buyer/seller roles from the start
- **DB:** PostgreSQL — listings, transactions, reviews as first-class entities
- **Payments:** Stripe Connect (built for marketplace splits/escrow)
- **Search:** Postgres full-text search → Algolia/Meilisearch once you scale
- **Auth:** Role-based access (buyer/seller/admin)

```
[Buyer App] --\                    /-- [Seller Dashboard]
               v                  v
            [Monolithic API]
               |        |       |
               v        v       v
         [PostgreSQL] [Stripe Connect] [Search Index]
```

**Priorities:** trust signals (reviews, verification), clear transaction state machine, notifications.
**Skip:** recommendation engines, dynamic pricing, complex logistics until volume justifies it.

---

### 4. SaaS MVP

**Core need:** Multi-tenancy, subscription billing, converting onboarding

- **Frontend:** Next.js / React
- **Backend:** Monolith with tenant isolation from day one (`tenant_id`/`org_id`)
- **DB:** PostgreSQL with row-level tenant scoping
- **Billing:** Stripe Billing (subscriptions, trials, invoices)
- **Auth:** Clerk / WorkOS / Auth0 (WorkOS if enterprise SSO is coming)
- **Hosting:** Render / Fly.io / Vercel

```
[Web App] -> [API] -> [PostgreSQL: tenant-scoped tables]
                |
                +--> [Stripe Billing]
                +--> [Auth/Org Provider]
                +--> [Email: Resend/Postmark for transactional]
```

**Priorities:** clean signup → trial → paid conversion, usage tracking, transactional email.
**Skip:** enterprise SSO, custom roles/permissions, white-labeling — until a paying customer asks.

---

### 5. AI Product MVP

**Core need:** Reliable model orchestration, cost/latency control, non-deterministic output handling

- **Frontend:** Next.js / React with streaming UI (SSE or Vercel AI SDK)
- **Backend:** Thin orchestration layer over LLM provider calls
- **Model layer:** Anthropic/OpenAI API directly; self-host only if cost/data residency force it
- **Vector DB (if RAG):** Pinecone / Supabase pgvector / Chroma
- **DB:** PostgreSQL for app data (conversations, users, usage)
- **Auth:** Standard provider
- **Observability:** Log every prompt/response pair

```
[Client: Web/Mobile] -> [API Layer] -> [LLM Provider API]
                              |               |
                              v               v
                       [PostgreSQL]   [Vector DB (if RAG)]
                              |
                              +--> [Logging/Eval pipeline]
```

**Priorities:** prompt versioning, cost monitoring, graceful fallback, feedback loop (👍/👎).
**Skip:** fine-tuning your own model, complex agent frameworks, multi-model routing — until proven necessary.

---

## 📊 Quick Comparison Table

| Product Type | Unique Complexity | Non-Negotiable Service |
|---|---|---|
| 🌐 Web App | SEO / performance | Hosting + DB |
| 📱 Mobile App | App store cycle, offline support | Push notifications |
| 🛒 Marketplace | Two-sided trust, split payments | Stripe Connect |
| 💼 SaaS | Multi-tenancy, billing | Stripe Billing |
| 🤖 AI Product | Non-determinism, cost/latency | LLM provider + logging |

---

## 📈 When to Evolve the Architecture

Treat these as real signals, not premature optimization:

- ✅ Real users are hitting **actual** performance limits (not hypothetical ones)
- ✅ A specific service is bottlenecking releases for the rest of the team
- ✅ Compliance/security requirements force isolation (e.g., health or financial data)
- ✅ You've validated the hypothesis and are now scaling a proven product

---

## ⚠️ Common Mistakes

- Building for scale you don't have yet
- Over-abstracting code for flexibility you don't need
- Choosing tech based on resume-building instead of team skill
- Skipping analytics, so you can't tell if the MVP succeeded
- No deployment pipeline, so shipping updates becomes painful
- Ignoring basic security "because it's just an MVP" — this gets expensive later

---

## 🤝 Contributing

Contributions are welcome! If you have real-world MVP architecture lessons, feel free to:

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/my-addition`)
3. Commit your changes (`git commit -m 'Add some insight'`)
4. Push to the branch (`git push origin feature/my-addition`)
5. Open a Pull Request

---

## 📄 License

This guide is released under the [MIT License](LICENSE) — use it, share it, adapt it for your team.

---

<p align="center">
  ⭐ If this guide helped you ship faster, consider starring the repo!
</p>
