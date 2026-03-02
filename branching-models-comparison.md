# Git Branching Models — Comparison Summary

A reference guide comparing the four major Git branching strategies: **Git Flow**, **GitHub Flow**, **Trunk-Based Development (TBD)**, and **GitLab Flow**.

---

## At a Glance

| Dimension | Git Flow | GitHub Flow | Trunk-Based Dev | GitLab Flow |
|-----------|---------|------------|----------------|------------|
| **Complexity** | High | Very Low | Low–Medium | Medium |
| **Long-lived branches** | `main`, `develop` | `main` only | `main` only | `main`, `staging`, `production` |
| **Release cadence** | Scheduled / periodic | Continuous (any time) | Continuous (multiple/day) | Continuous or scheduled |
| **Supports multi-environment natively** | ❌ No (requires convention) | ❌ No (requires convention) | ❌ No (uses flags/gates) | ✅ Yes (env branches) |
| **Feature flags required** | No | Sometimes | ✅ Yes (mandatory) | Sometimes |
| **Hotfix strategy** | Dedicated `hotfix/*` branch | Same as feature branch | Commit to trunk + flag | `hotfix/*` → cherry-pick |
| **Team size sweet spot** | Medium–Large (5–50+) | Small–Medium (1–20) | Any (with discipline) | Medium–Large (5–50+) |
| **Best for** | Versioned releases | SaaS / fast delivery | CI/CD native teams | Multi-env structured teams |
| **CI/CD maturity needed** | Medium | Medium | High | Medium–High |

---

## Branch Structure Comparison

| Branch | Git Flow | GitHub Flow | Trunk-Based Dev | GitLab Flow |
|--------|---------|------------|----------------|------------|
| `main` | Production-ready code | Single deployable branch | Trunk (always green) | Development integration |
| `develop` | ✅ Integration branch | ❌ | ❌ | ❌ |
| `staging` / `pre-prod` | ❌ (by convention only) | ❌ (by convention only) | ❌ (gate in pipeline) | ✅ Long-lived env branch |
| `production` | ❌ (`main` = prod) | ❌ (`main` = prod) | ❌ (`main` = prod) | ✅ Long-lived env branch |
| `feature/*` | ✅ (from `develop`) | ✅ (from `main`) | ✅ (max 1–2 days) | ✅ (from `main`) |
| `release/*` | ✅ Stabilisation branch | ❌ | ❌ (optional, brief) | ✅ (release variant) |
| `hotfix/*` | ✅ From `main` | ❌ (just a feature branch) | ❌ (commit to trunk) | ✅ From `production` |

---

## Environment Deployment Mapping

### Git Flow

| Branch | Environment |
|--------|------------|
| `feature/*` | **Dev** |
| `develop` | **QA** |
| `release/*` | **UAT** |
| `main` / tag | **PROD** |
| `hotfix/*` | **PROD** (expedited) |

### GitHub Flow

| Branch / Event | Environment |
|---------------|------------|
| `feature/*` (PR open) | **Dev** (per-PR preview) |
| `feature/*` (PR review) | **QA** (optional deploy to staging) |
| `main` (post-merge) | **UAT** (automatic) |
| `main` (approved) | **PROD** (manual gate) |

### Trunk-Based Development

| Branch / Event | Environment |
|---------------|------------|
| `main` (every commit) | **Dev** (auto) |
| `main` (CI green) | **QA** (auto) |
| `main` (RC tag / gate) | **UAT** (gated) |
| `main` (approved) | **PROD** (approval + canary) |

### GitLab Flow

| Branch | Environment |
|--------|------------|
| `feature/*` | **Dev** |
| `main` | **QA** |
| `staging` | **UAT** |
| `production` | **PROD** |

---

## Pros & Cons Summary

### Git Flow

| ✅ Pros | ❌ Cons |
|--------|--------|
| Structured, predictable releases | Complex branch hierarchy |
| Great for versioned/scheduled releases | Slow delivery; not CD-friendly |
| Parallel feature development | Large, painful merges on long-lived branches |
| Hotfixes isolated safely | Overkill for small teams |
| Excellent audit trail | `develop` can become a bottleneck |

### GitHub Flow

| ✅ Pros | ❌ Cons |
|--------|--------|
| Extremely simple | `main` breakage = prod breakage |
| Native to CI/CD pipelines | No release management out-of-the-box |
| Fast iteration cycles | Hotfixes indistinguishable from features |
| Per-PR preview environments | Not suited for multiple concurrent versions |
| Low cognitive overhead | Multi-env compliance needs extra tooling |

### Trunk-Based Development

| ✅ Pros | ❌ Cons |
|--------|--------|
| True CI — daily integration | Requires feature flags (adds complexity) |
| No long merge conflicts | Demands high CI/CD maturity |
| Always-deployable trunk | Discipline required from all developers |
| Fastest feedback loops | Feature flag debt accumulates |
| Scales to massive teams | Difficult for mobile (app store cycles) |

### GitLab Flow

| ✅ Pros | ❌ Cons |
|--------|--------|
| Clear environment promotion path | Branch proliferation (env branches + features) |
| Native multi-environment support | Cherry-pick risk in release variant |
| Complete audit trail per environment | Environment branches can lag/diverge |
| Balances simplicity and structure | Upstream-only rule requires discipline |
| Flexible (env or release variant) | Full value requires CI/CD maturity |

---

## Which Model for Which Application?

| Application Type | Recommended Model | Runner-Up | Avoid |
|-----------------|------------------|-----------|-------|
| **Enterprise Web App** (large team, scheduled releases) | **Git Flow** | GitLab Flow | TBD (unless mature CI) |
| **SaaS Web App** (fast-moving, CD) | **GitHub Flow** / **TBD** | GitLab Flow | Git Flow |
| **Mobile App** (iOS / Android) | **Git Flow** | GitLab Flow (release variant) | TBD |
| **Microservices / APIs** (high-frequency deploys) | **TBD** | GitHub Flow | Git Flow |
| **API / Backend** (multi-env, regulated) | **GitLab Flow** | Git Flow | TBD |
| **Open Source Library / SDK** | **Git Flow** | GitLab Flow (release variant) | GitHub Flow |
| **Internal Tool / Dashboard** | **GitHub Flow** | GitLab Flow | Git Flow |
| **Startup / Early-stage product** | **GitHub Flow** | TBD | Git Flow |
| **Embedded / Hardware Software** | **Git Flow** | GitLab Flow (release variant) | TBD |
| **Cloud-native / Containerised** | **TBD** | GitLab Flow | Git Flow |

---

## Decision Flowchart

```
Is your team small (< 5 devs) and moving fast?
    YES ──► GitHub Flow
    NO  ──► Continue

Do you need strict multi-environment promotion? (Dev → QA → UAT → PROD)
    YES ──► GitLab Flow (or Git Flow for versioned releases)
    NO  ──► Continue

Do you ship versioned, scheduled releases? (e.g., mobile, desktop, libraries)
    YES ──► Git Flow
    NO  ──► Continue

Do you deploy multiple times per day with strong CI/CD?
    YES ──► Trunk-Based Development
    NO  ──► GitHub Flow or GitLab Flow
```

---

## Key Differentiators (One-Liners)

| Model | One-Liner |
|-------|-----------|
| **Git Flow** | "Two permanent branches, structured releases, built for versioned software." |
| **GitHub Flow** | "One branch, ship anytime, keep it simple." |
| **TBD** | "Commit to trunk daily, use flags, always be deployable." |
| **GitLab Flow** | "Environment branches enforce promotion; code only flows downstream." |

---

## Detailed Documentation

For full details on each branching model, see the individual files:

- [git-flow.md](./git-flow.md) — Git Flow (structured, versioned releases)
- [github-flow.md](./github-flow.md) — GitHub Flow (simple, continuous delivery)
- [trunk-based-development.md](./trunk-based-development.md) — Trunk-Based Development (CI-first, flag-driven)
- [gitlab-flow.md](./gitlab-flow.md) — GitLab Flow (environment branches, multi-env promotion)

---

*Document version: 1.0 | Last updated: March 2026*
