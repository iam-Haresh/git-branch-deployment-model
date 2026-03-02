# Git Flow Branching Model

## Overview

Git Flow is a robust branching strategy introduced by Vincent Driessen in 2010. It defines a strict branching model designed around project releases. It provides a solid framework for managing larger projects and is especially popular in teams following scheduled, versioned releases.

Git Flow uses **two permanent long-lived branches** and several supporting short-lived branches, each with a specific role.

---

## Branch Structure

### Long-Lived Branches (Permanent)

| Branch | Purpose | Default Branch? |
|--------|---------|----------------|
| `main` / `master` | Always reflects **production-ready** code. Every release commit here is a tagged. | ✅ Yes (production baseline) |
| `develop` | Integration branch. Reflects the **latest delivered development** changes for the next release. | Acts as the default for feature work |

### Short-Lived Branches (Temporary)

| Branch Type | Naming Convention | Branched From | Merges Into | Lifespan |
|------------|------------------|--------------|------------|---------|
| Feature | `feature/<ticket-id>-short-description` | `develop` | `develop` | Days to weeks |
| Release | `release/<version>` e.g. `release/1.4.0` | `develop` | `main` + `develop` | Days to 1–2 weeks |
| Hotfix | `hotfix/<version>` e.g. `hotfix/1.4.1` | `main` | `main` + `develop` | Hours to a day |
| Bugfix (not standard) | `bugfix/<ticket-id>-description` | `develop` or `release/*` | Same branch it came from | Days |

---

## Default Branch

- **`main`** is the default/protected branch representing production.
- **`develop`** is the integration branch where all completed features land.
- Most developers set `develop` as the *clone default* in repository settings so new contributors start there.

---

## Branch → Environment Deployment Mapping

| Branch | Deployed To | Notes |
|--------|------------|-------|
| `feature/*` | **Dev** (optional per-feature environment or shared Dev) | Developers validate individual features here |
| `develop` | **Dev** | Automated tests and QA team testing happen here after feature merges |
| `release/*` | **QA/UAT** | Business/client acceptance testing; only bug fixes allowed here |
| `main` / `master` | **PROD** | **Tagged** releases go live; deployment is triggered on merge + tag |
| `hotfix/*` | **hotfix → main → tag → PROD** (fast-tracked via UAT smoke test) | Emergency fixes bypass the full release cycle |

```
feature/* ──► DEV
develop   ──► QA
release/* ──► UAT
main      ──► PROD
hotfix/*  ──► PROD (expedited)
```

---

## Which Applications / Teams Should Use Git Flow?

| Application Type | Fit | Reason |
|-----------------|-----|--------|
| **Enterprise Web Applications** | ✅ Excellent | Structured releases, multiple environments, large teams |
| **Mobile Apps (iOS / Android)** | ✅ Excellent | App store releases are versioned; hotfix branches map to patch releases |
| **Desktop Applications** | ✅ Excellent | Versioned installers, long QA cycles |
| **SaaS B2B Platforms** | ✅ Good | Client-driven release windows, UAT sign-off requirements |
| **Open Source Libraries / SDKs** | ✅ Good | Semantic versioning aligns perfectly |
| **Microservices / APIs (fast-paced)** | ⚠️ Moderate | Can be heavy for very small, rapidly deployed services |
| **Internal Tools / Prototypes** | ❌ Overkill | Overhead not justified for small, frequently shipped tools |

---

## Pros and Cons

### ✅ Pros

- **Clear, explicit structure** — every developer knows where features, releases, and hotfixes live.
- **Parallel development** — multiple features can be developed simultaneously without interference.
- **Supports scheduled releases** — perfect for teams that ship on a defined cadence (sprints, quarters).
- **Hotfix isolation** — production issues can be patched without destabilising in-progress feature work.
- **Audit-friendly** — release branches and tags provide a clean history for compliance and auditing.
- **Mature tooling** — `git flow` CLI extensions, IDE plugins, and CI/CD integrations are widely available.
- **Works well for versioned software** — semantic versioning (`major.minor.patch`) maps directly to the workflow.

### ❌ Cons

- **High branch overhead** — maintaining `main`, `develop`, and multiple feature/release/hotfix branches increases complexity.
- **Slow delivery** — code must traverse feature → develop → release → main; unsuitable for continuous delivery.
- **Frequent merge conflicts** — long-lived feature branches diverge from `develop`, leading to large, painful merges.
- **Not CI/CD friendly** — the multi-branch structure complicates fully automated pipelines.
- **Overkill for small teams** — teams of 1–3 developers rarely need the full ceremony of Git Flow.
- **`develop` can become a bottleneck** — if many features merge simultaneously, `develop` can become unstable.

---

## Industry Practices & Customisations

### Common Industry Adaptations

#### 1. Naming Conventions
Teams often standardise branch names for automation and CI/CD routing:
```
feature/JIRA-1234-user-authentication
bugfix/JIRA-5678-login-timeout
release/2.3.0
hotfix/2.3.1
```

#### 2. Pull Request (PR) / Merge Request (MR) Gates
All merges (especially into `develop` and `main`) go through mandatory PRs with:
- Minimum 1–2 reviewer approvals
- Passing CI pipeline (lint, unit tests, security scan)
- No unresolved comments

#### 3. Squash Merging Features
Many teams **squash commits** when merging feature branches into `develop` to keep the history clean and readable.

#### 4. Release Freeze on Release Branches
Once a `release/*` branch is cut, teams enforce a **feature freeze** — only bug fixes and documentation changes are permitted. This mimics a traditional code freeze.

#### 5. Automated Environment Promotion
CI/CD pipelines auto-deploy based on branch:
```yaml
# Example GitHub Actions trigger pattern
on:
  push:
    branches:
      - feature/**   # → deploys to DEV
      - develop      # → deploys to Dev/Integration
      - release/**   # → deploys to UAT
      - tag         # → deploys to PROD (tag crated from main)
```

#### 6. Hotfix Fast-Track Process
Industry practice usually requires:
1. Hotfix branch created from `main` (tagged version)
2. Fix committed and reviewed via expedited PR (1 approver, not 2)
3. UAT smoke test (30–60 mins, not full regression)
4. Merged back to both `main` AND `develop`
5. `main` tagged with patch version (`v2.3.1`)

#### 7. Long-Running Release Branches for Mobile
Mobile teams (iOS/Android) sometimes keep `release/*` branches open for 1–2 weeks:
- App store review periods
- Back-porting bug fixes between store submission and approval
- Supporting multiple OS version builds from the same release branch

#### 8. Branch Protection Rules
Standard enterprise setup:
| Branch | Protection |
|--------|-----------|
| `main` | No direct push; PR required; admin bypass only for hotfixes |
| `develop` | PR required; 1+ approver; CI must pass |
| `release/*` | PR required; release manager approval |

#### 9. Semantic Versioning Integration
Releases are always tagged:
```
git tag -a v2.3.0 -m "Release 2.3.0"
git push origin v2.3.0
```
Many teams automate this with tools like **semantic-release** or **standard-version**.

#### 10. Git Flow Tooling
- **git-flow** CLI: `brew install git-flow-avh`
- **SourceTree** has built-in Git Flow support
- **IntelliJ / VS Code** plugins for Git Flow
- **GitHub / GitLab** branch templates to enforce naming patterns

---

## Summary Diagram

```
          ┌─────────────────────────────────────────────────────┐
          │                     main (PROD)                     │
          └────────────────────┬────────────────────────────────┘
                               │ merge + tag
          ┌────────────────────┴────────────────────────────────┐
          │                release/x.y.z (UAT)                  │
          └──────────────────┬──────────────────────────────────┘
                             │ cut release branch
          ┌──────────────────┴──────────────────────────────────┐
          │                    develop (QA)                      │
          └──┬──────────────────┬─────────────────┬─────────────┘
             │                  │                  │
    ┌────────┴──────┐  ┌────────┴──────┐  ┌───────┴────────┐
    │  feature/A    │  │  feature/B    │  │  feature/C     │
    │    (DEV)      │  │    (DEV)      │  │    (DEV)       │
    └───────────────┘  └───────────────┘  └────────────────┘
```

---

*Document version: 1.0 | Last updated: March 2026*
