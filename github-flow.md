# GitHub Flow Branching Model

## Overview

GitHub Flow is a lightweight, continuous-delivery-oriented branching strategy created by GitHub's own engineering team. It was designed to simplify branching to the absolute minimum — favouring **speed, simplicity, and continuous deployment** over structured release cycles.

The core principle is straightforward: **`main` is always deployable**, and all work happens in short-lived feature branches that are merged directly into `main` via Pull Requests.

---

## Branch Structure

### Long-Lived Branches (Permanent)

| Branch | Purpose | Default Branch? |
|--------|---------|----------------|
| `main` | Always reflects **production-ready, deployable** code. | ✅ Yes — single source of truth |

> GitHub Flow has **only one long-lived branch**. Everything else is a short-lived branch.

### Short-Lived Branches (Temporary)

| Branch Type | Naming Convention | Branched From | Merges Into | Lifespan |
|------------|------------------|--------------|------------|---------|
| Feature / Bug / Task | `feature/<description>` `fix/<description>` `chore/<description>` | `main` | `main` | Hours to a few days |
| Experimental | `experiment/<description>` | `main` | May be abandoned | Short |

> There are **no release branches, no develop branch, no hotfix branches** in pure GitHub Flow.

---

## Default Branch

- **`main`** is the only default and protected branch.
- Every branch is **directly branched from `main`** and merged back into `main`.
- The repository default branch setting in GitHub/GitLab should always be `main`.

---

## Branch → Environment Deployment Mapping

GitHub Flow relies on **deploy-on-merge** (to `main`) or **deploy-on-PR** (ephemeral preview environments). Teams commonly adapt it to multi-environment workflows like this:

| Branch | Deployed To | Notes |
|--------|------------|-------|
| `feature/*` | **Dev** (Preview / Ephemeral environment) | Automatically spun up per PR for isolated testing |
| `main` (before merge, via PR) | **QA** (Staging) | Some teams deploy the PR branch to staging for review |
| `main` (post merge) | **UAT → PROD** (auto or gated) | Merge triggers deployment; feature flags or approvals gate PROD |

### Adaptation for 4-Environment Teams

Since GitHub Flow is designed for CD (direct to prod), teams with **Dev → QA → UAT → PROD** requirements adapt it as follows:

```
feature/* (PR open)  ──► DEV (preview environment, auto-deployed per PR)
feature/* (PR review ready) ──► QA (deployed to QA for testing before merge)
main (post-merge)    ──► UAT (auto-deploy; stakeholder acceptance)
main (approved)      ──► PROD (manual approval gate or feature flag rollout)
```

| Branch/Event | Environment | Trigger |
|-------------|------------|---------|
| Push to `feature/*` | **Dev** (ephemeral) | Auto on push |
| PR opened/updated | **QA** | Auto on PR open |
| Merge to `main` | **UAT** | Auto on merge |
| UAT sign-off | **PROD** | Manual gate or scheduled promotion |

---

## Which Applications / Teams Should Use GitHub Flow?

| Application Type | Fit | Reason |
|-----------------|-----|--------|
| **SaaS Web Applications** | ✅ Excellent | Continuous deployment is the norm; `main` always ships |
| **Internal Web Dashboards / Tools** | ✅ Excellent | Lightweight, fast iteration |
| **APIs / Backend Microservices** | ✅ Excellent | Frequent small releases; stateless deployments |
| **Developer Tooling / CLIs** | ✅ Good | Small teams, fast feedback loops |
| **Startups / Early-stage products** | ✅ Excellent | Minimises overhead, maximises velocity |
| **Mobile Apps (iOS / Android)** | ⚠️ Limited | App store reviews and versioned releases make pure GitHub Flow awkward |
| **Enterprise Apps with regulated releases** | ⚠️ Moderate | Requires adding gates and approval steps on top of GitHub Flow |
| **Hardware-constrained embedded systems** | ❌ Poor | Versioned, tested releases are essential; GitHub Flow is too fluid |

---

## Pros and Cons

### ✅ Pros

- **Extreme simplicity** — one main branch, one rule: `main` is always deployable.
- **Fast iteration** — branches are short-lived (hours to 2–3 days), reducing merge conflicts dramatically.
- **CI/CD friendly** — the entire model is built around automation and continuous deployment.
- **Low cognitive overhead** — new developers understand it in minutes.
- **Encourages collaboration** — PRs are the central unit of review, discussion, and quality gates.
- **Ephemeral environments** — per-PR preview deployments (Vercel, Netlify, AWS Amplify, etc.) make testing trivial.
- **Suitable for small to medium teams** — governance is handled by PR approvals, not branch hierarchy.

### ❌ Cons

- **`main` must always be stable** — any bad merge can immediately affect production.
- **No native release management** — versioning and changelogs need to be handled separately (feature flags, semantic-release, etc.).
- **Not suitable for parallel version support** — you cannot easily maintain `v1.x` and `v2.x` simultaneously.
- **Hotfixes aren't distinct** — a production bug fix looks the same as a feature branch; teams need discipline.
- **Multi-environment compliance is implicit** — Dev → QA → UAT → PROD promotion requires extra tooling (not built into the model).
- **Requires mature CI/CD** — without automated testing and deployment, merging directly to `main` is risky.
- **Difficult for large teams** — without a `develop` buffer, simultaneous merges from many teams can destabilise `main`.

---

## Industry Practices & Customisations

### Common Industry Adaptations

#### 1. Feature Flags for Safe Merging
The most critical practice layered on top of GitHub Flow:
```
- Feature is merged to main (deployed everywhere)  
- Feature is hidden behind a flag: `if (feature_flag.isEnabled('new_checkout')) {...}`  
- Flag is enabled per environment: DEV → QA → UAT → PROD  
```
Tools: LaunchDarkly, Unleash, AWS AppConfig, Flagsmith.

#### 2. Per-PR Preview / Ephemeral Environments
Every PR automatically gets its own URL:
```
https://pr-234-my-app.preview.yourcompany.com
```
Tools: Vercel, Netlify, Railway, AWS Amplify, ArgoCD with dynamic namespaces.

This effectively replaces a dedicated Dev environment per developer.

#### 3. Branch Protection on `main`
Strict rules applied to `main`:
- Require PR; no direct pushes (even for admins)
- Minimum 1–2 approvals
- Required status checks: unit tests, integration tests, lint, security scan
- Linear history (squash or rebase merges only)

#### 4. Deployment Gates (for Regulated Environments)
Teams add manual approval gates in CI/CD for UAT → PROD:
```yaml
# GitHub Actions environment protection rules
environment:
  name: production
  url: https://app.yourcompany.com
# Requires designated reviewers to approve before deploying
```

#### 5. Standardised Branch Naming
Teams enforce naming conventions via branch policies or pre-push hooks:
```
feature/<JIRA-ID>-short-description
fix/<JIRA-ID>-bug-description
chore/update-dependencies
docs/update-readme
experiment/new-caching-approach
```

#### 6. Squash Merge Policy
Most teams using GitHub Flow enforce **squash merges**:
- Each PR becomes a single commit on `main`
- Clean, linear history
- Easy to revert a complete feature with `git revert <sha>`

#### 7. Automated Version Bumping
Since GitHub Flow has no release branches, teams automate versioning:
```
Conventional Commits → commitlint → semantic-release → GitHub Release + Tag
```
Example commit messages:
```
feat: add OAuth2 login          → minor version bump
fix: correct token expiry       → patch version bump
feat!: redesign API response    → major version bump (breaking change)
```

#### 8. QA on PR via Staging Sync
Some teams deploy the PR branch to a shared QA/staging environment:
```
Push to feature/* → CI builds image → Deploy to staging.yourcompany.com → QA tests → Approve PR → Merge to main
```

#### 9. Rollback Strategy
Since there are no reverting release branches, teams agree on rollback approaches:
- **Revert PR** (`git revert`) — creates a new commit undoing the change
- **Re-deploy previous image** — tag-based rollback in Kubernetes/ECS
- **Feature flag off** — fastest option when flags are in place

---

## Summary Diagram

```
          ┌──────────────────────────────────────────────────────────┐
          │                        main (UAT → PROD)                 │
          └──────────┬──────────────────┬───────────────┬────────────┘
                     │ merge            │ merge          │ merge
          ┌──────────┴──────┐  ┌────────┴──────┐  ┌─────┴───────────┐
          │  feature/login  │  │  fix/cart-bug │  │  chore/dep-upd  │
          │   (DEV/QA)      │  │   (DEV/QA)    │  │   (DEV/QA)      │
          └─────────────────┘  └───────────────┘  └─────────────────┘

  PR ──► CI ──► Code Review ──► Deploy to QA ──► Merge to main ──► UAT ──► PROD
```

---

*Document version: 1.0 | Last updated: March 2026*
