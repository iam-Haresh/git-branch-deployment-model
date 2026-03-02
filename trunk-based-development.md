# Trunk-Based Development (TBD)

## Overview

Trunk-Based Development (TBD) is a branching strategy where developers integrate small, frequent code changes directly into a single shared branch called the **trunk** (typically `main` or `master`). It is the foundation for **true Continuous Integration (CI)** and is championed by high-performing engineering teams like Google, Facebook, and Netflix.

The core philosophy is: **commit small, commit often, always keep the trunk releasable.** Feature Flags are the primary mechanism for hiding incomplete work rather than long-lived branches.

There are two common variants:
1. **Pure TBD** — developers commit directly to `main` (small teams)
2. **Scaled TBD** — developers use very short-lived feature branches (1–2 days max) merged via PR

---

## Branch Structure

### Long-Lived Branches (Permanent)

| Branch | Purpose | Default Branch? |
|--------|---------|----------------|
| `main` / `trunk` | The single shared integration branch. **Always in a releasable state.** | ✅ Yes — the only long-lived branch |
| `release/x.y` (optional) | Short-term stabilisation branches cut from `main` when a release needs hardening | Created only at release time; deleted quickly |

### Short-Lived Branches (Temporary)

| Branch Type | Naming Convention | Branched From | Merges Into | Max Lifespan |
|------------|------------------|--------------|------------|-------------|
| Short-lived feature | `feature/<description>` or `users/<name>/<task>` | `main` | `main` | **1–2 days maximum** |
| Release stabilisation | `release/x.y.z` | `main` | — (deployed directly, no merge back) | Days to 1 week |

> **Key rule in TBD**: If a branch lives longer than 2 days, it is a red flag. Break the work into smaller pieces.

---

## Default Branch

- **`main`** (or `trunk`) is the only default branch.
- Developers integrate into `main` at least **once per day** (ideally multiple times).
- The trunk must **always be green** (all CI checks passing).

---

## Branch → Environment Deployment Mapping

TBD is inherently designed for continuous deployment. Environment promotions happen from `main` using feature flags, pipeline gates, or release branches.

| Branch / Event | Deployed To | Notes |
|---------------|------------|-------|
| `main` (every commit / merge) | **Dev** | Every commit triggers automated build and deploy to Dev |
| `main` (scheduled or on demand) | **QA** | Automated test suite runs; QA gates pipeline progression |
| `main` (release candidate tag) | **UAT** | Tagged commit (e.g. `rc/1.4.0`) promoted to UAT |
| `release/x.y` or tagged `main` | **PROD** | Feature flags fully enabled; approved release goes live |

```
main commit ──► DEV (auto, every push)
              ──► QA  (auto, on CI green)
              ──► UAT (manual gate or scheduled; RC tag)
              ──► PROD (approval gate + feature flag rollout)
```

### Feature Flag Environment Promotion

```
Feature merged to main
     ↓
DEV:  flag = ON  (developers test)
QA:   flag = ON  (automated + manual QA)
UAT:  flag = ON  (business acceptance)
PROD: flag = OFF → gradual rollout → flag = ON (100%)
```

---

## Which Applications / Teams Should Use TBD?

| Application Type | Fit | Reason |
|-----------------|-----|--------|
| **High-velocity SaaS Web Apps** | ✅ Excellent | Multiple deploys per day; CI/CD native |
| **Microservices / APIs** | ✅ Excellent | Independent deployable units; frequent changes |
| **Developer Platforms / SDKs** | ✅ Good | Frequent patch releases; dogfooding culture |
| **Cloud-native applications** | ✅ Excellent | Container-based deployments align with every-commit CD |
| **Large engineering organisations (Google scale)** | ✅ Excellent | Google uses a monorepo with TBD for 2000+ engineers |
| **Mobile Apps (iOS / Android)** | ⚠️ Moderate | App store releases complicate pure TBD; release branches needed |
| **Regulated / Financial systems** | ⚠️ Moderate | Strict UAT sign-off requirements need gates added on top |
| **Embedded / Hardware systems** | ❌ Poor | Risks too high for direct-to-trunk; versioned releases essential |
| **Small agencies / freelancers** | ⚠️ Overhead | Feature flags add tooling complexity for small projects |

---

## Pros and Cons

### ✅ Pros

- **True Continuous Integration** — all developers integrate into a shared branch daily; conflicts surface immediately.
- **Eliminates long merge hell** — no long-lived feature branches means no "big bang" merges.
- **Always-deployable trunk** — the trunk is production-ready at all times; releases are low-risk.
- **Fastest feedback loops** — code reaches QA/staging within hours of being written.
- **Encourages small, incremental changes** — forces good software design (modular, decoupled code).
- **Scales to very large teams** — Google, Meta, and Netflix use this at massive scale.
- **Simple branching model** — one branch to understand, one branch to rule them all.

### ❌ Cons

- **Requires mature CI/CD** — without comprehensive automated tests, committing to trunk is dangerous.
- **Feature flags are mandatory** — incomplete features must be hidden behind flags; adds operational complexity.
- **Discipline required** — developers must commit small, frequently, and keep the trunk green at all times.
- **Feature flag debt** — flags accumulate over time and become a maintenance burden if not cleaned up.
- **Difficult for junior teams** — the "always green trunk" rule fails without strong engineering culture.
- **Complex for mobile** — app store submission cycles don't align with multiple-deploys-per-day.
- **Harder to support multiple concurrent versions** — maintaining v1.x and v2.x simultaneously is difficult.
- **Incident response requires feature flags or fast rollback** — no separate branch to "hold back."

---

## Industry Practices & Customisations

### Common Industry Adaptations

#### 1. Feature Flags as the Core Safety Mechanism
TBD **cannot function safely without feature flags**. This is non-negotiable.
```javascript
// Example using LaunchDarkly
if (featureFlags.isEnabled('new-payment-flow', user)) {
  return <NewPaymentFlow />;
}
return <LegacyPaymentFlow />;
```
Standard tooling: **LaunchDarkly, Unleash, AWS AppConfig, Optimizely, Split.io, Flagsmith**

#### 2. The Expand-Contract (Parallel Change) Pattern
For breaking API or database changes, teams use the expand-contract pattern:
```
Step 1 (Expand):  Add new column/endpoint alongside old one — deploy
Step 2 (Migrate): Migrate consumers to new column/endpoint — deploy  
Step 3 (Contract): Remove old column/endpoint — deploy
```
This avoids coordinated deployments and keeps trunk green throughout.

#### 3. Branch by Abstraction
For large refactors, an abstraction layer wraps both old and new implementations:
```
New code path hidden behind abstraction → merged to trunk → gradually flipped via flag
```

#### 4. Trunk Must Always Be Green
- CI pipeline blocks any merge that breaks tests.
- If the trunk is broken, **fixing it is the top priority** — all other work stops.
- "Stop the line" culture (borrowed from lean manufacturing).

#### 5. Commit Conventions and Size
Industry standards for TBD commits:
- Each commit should take **no more than 1–2 hours of work**.
- Commits should have passing tests locally before push.
- Conventional Commits standard used for automation:
  ```
  feat: add retry logic to payment service
  fix: handle null pointer in user service
  refactor: extract auth helper into shared module
  ```

#### 6. Release Branches (Scaled TBD)
High-maturity teams cut a release branch only for **stabilisation**:
```bash
git checkout -b release/2.4.0 main
# Only bug fixes with cherry-picks allowed
# Branch deleted after release goes live
```
No new features are added to a release branch.

#### 7. Pipeline-as-Code for Environment Promotion
```yaml
# Simplified CI/CD with environment gates
stages:
  - build
  - test
  - deploy-dev     # auto on every push to main
  - deploy-qa      # auto if tests pass
  - deploy-uat     # manual approval required
  - deploy-prod    # manual approval + canary rollout
```

#### 8. Canary Releases and Progressive Delivery
TBD teams rarely do big-bang PROD deployments:
- **Canary**: Route 5% of traffic to new version → watch metrics → expand to 100%
- **Blue/Green**: Two identical environments; switch traffic when confident
- Tools: **Argo Rollouts, Spinnaker, AWS CodeDeploy, Flagger**

#### 9. Automated Test Pyramid
TBD demands a robust test suite:
```
                    /\
                   /E2E\        (few, slow)
                  /──────\
                 / Integ  \     (moderate)
                /──────────\
               /  Unit Tests\   (many, fast)
              /──────────────\
```
- **Unit tests**: Must run in < 10 minutes on CI.
- **Integration tests**: Run per PR before merge.
- **E2E tests**: Run on Dev/QA; gate UAT promotion.

#### 10. Observability and Rollback Discipline
- Every deployment includes monitoring dashboards and alerts.
- Agreed rollback trigger: if error rate > X% within 15 minutes → auto rollback.
- Feature flags are the fastest rollback: flip the flag, no redeployment needed.

---

## Summary Diagram

```
     Developer A          Developer B          Developer C
          │                    │                    │
     commits daily        commits daily        commits daily
          │                    │                    │
          └─────────┬──────────┘         ┌──────────┘
                    ▼                    │
          ┌─────────────────────────────────────────────────┐
          │               main / trunk                      │
          │          (always green, always deployable)      │
          └────────┬────────────────────┬───────────────────┘
                   │ auto               │ gated
                   ▼                    ▼
                  DEV ──► QA ──► UAT ──► PROD
                      (pipeline with approval gates)
```

---

*Document version: 1.0 | Last updated: March 2026*
