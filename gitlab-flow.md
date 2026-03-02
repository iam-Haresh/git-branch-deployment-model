# GitLab Flow Branching Model

## Overview

GitLab Flow is a branching strategy created by GitLab Inc. that sits between the simplicity of GitHub Flow and the rigidity of Git Flow. It was designed to address a key gap: **GitHub Flow doesn't have an opinionated story for deployments to multiple environments**, while **Git Flow adds too much complexity** for most teams.

GitLab Flow solves this by introducing **environment branches** — dedicated, long-lived branches that mirror specific deployment environments. Code flows **downstream** through these branches (never upstream), ensuring clear, auditable promotion paths.

There are two main variants:
1. **Environment branches model** — for teams with multiple static environments (Dev → Staging → Production)
2. **Release branches model** — for versioned software that needs to support multiple releases in production simultaneously

---

## Branch Structure

### Long-Lived Branches (Permanent)

| Branch | Purpose | Default Branch? |
|--------|---------|----------------|
| `main` | Primary development integration branch. Reflects the latest delivered code. | ✅ Yes — main development branch |
| `staging` / `pre-production` | Mirror of the staging/UAT environment. Code flows from `main` → `staging`. | Long-lived environment branch |
| `production` | Mirror of the live production environment. Code flows from `staging` → `production`. | Long-lived environment branch |
| `release/x.y` (Release variant) | Long-lived branch for a specific release (e.g., `release/1.4`). Receives cherry-picks of bug fixes. | Used in versioned release model |

> **Key rule**: Code always flows **downstream** — `main` → `staging` → `production`. You **never merge backward** (from `production` to `main`).

### Short-Lived Branches (Temporary)

| Branch Type | Naming Convention | Branched From | Merges Into | Lifespan |
|------------|------------------|--------------|------------|---------|
| Feature | `feature/<ticket>-description` | `main` | `main` | Days to 1 week |
| Bug Fix | `fix/<ticket>-description` | `main` | `main` | Hours to days |
| Hotfix | `hotfix/<ticket>-description` | `production` | `production` + cherry-pick to `main` | Hours |
| Experimental | `experiment/<name>` | `main` | May be abandoned | Short |

---

## Default Branch

- **`main`** is the default development branch where all feature work lands.
- **`production`** is the **source of truth for what is live** in production.
- GitLab strongly recommends setting `main` as the repository's default branch (for cloning, MR base, etc.).
- The `staging` and `production` branches are protected and only updated via Merge Requests from upstream environment branches.

---

## Branch → Environment Deployment Mapping

GitLab Flow's **defining characteristic** is that branch names map directly to environments. CI/CD pipelines trigger deployments automatically when branches are updated.

### Environment Branches Model (Recommended for Most Web/API Teams)

| Branch | Deployed To | Trigger | Notes |
|--------|------------|---------|-------|
| `feature/*` | **Dev** (per-MR or shared dev environment) | Push to feature branch | Individual feature testing |
| `main` | **QA / Testing** | Merge to `main` | Integration testing; automated test suite |
| `staging` | **UAT / Pre-Production** | Merge from `main` → `staging` | Business/stakeholder acceptance testing |
| `production` | **PROD** | Merge from `staging` → `production` | Live; reviewed and approved |

```
feature/* ──► DEV
main      ──► QA
staging   ──► UAT
production──► PROD

Flow direction (downstream only):
feature/* → main → staging → production
```

### Release Branches Model (For Versioned Software)

| Branch | Deployed To | Notes |
|--------|------------|-------|
| `main` | **QA** | Integration testing |
| `release/2.3` | **UAT** | Release candidate testing and hotfix back-ports |
| `release/2.3` (tagged) | **PROD** | Tagged release goes live |

```
main → release/2.3 (cut when ready) → UAT → tag → PROD
                                    ↑
                              (bug fixes cherry-picked in)
```

### Hotfix Flow

```
hotfix/* (from production) → PROD-fix → cherry-pick → main (to prevent regression)
```

---

## Which Applications / Teams Should Use GitLab Flow?

| Application Type | Fit | Reason |
|-----------------|-----|--------|
| **Web Applications (multi-environment)** | ✅ Excellent | Environment branches map directly to dev/qa/uat/prod setup |
| **APIs / Backend Services** | ✅ Excellent | Clear promotion path; easy to audit what's in each environment |
| **SaaS Products with compliance requirements** | ✅ Excellent | Clear audit trail of what code is in which environment |
| **Enterprise Applications** | ✅ Excellent | Balance between Git Flow structure and GitHub Flow simplicity |
| **Mobile Applications** | ✅ Good | Release branches variant handles app store versioning well |
| **Libraries / Open Source** | ✅ Good | Release branches for multiple supported versions |
| **Microservices** | ✅ Good | Per-service repositories each use GitLab Flow independently |
| **Small startups (1–5 devs)** | ⚠️ Moderate | Slightly heavier than GitHub Flow; may be overkill |
| **Hardware / Embedded** | ⚠️ Moderate | Release branches variant works; environment branches don't apply |

---

## Pros and Cons

### ✅ Pros

- **Bridges the gap** between GitHub Flow (too simple) and Git Flow (too complex).
- **Explicit environment promotion** — code can only move downstream; prevents accidental mixing of unstable code into prod.
- **Clear audit trail** — the `staging` and `production` branches always reflect exactly what is deployed; no guesswork.
- **Native CI/CD integration** — GitLab CI/CD auto-detects branch names and triggers environment-specific pipelines.
- **Supports both CD and versioned releases** — two variants cover most team needs.
- **Hotfix handling is explicit** — hotfixes go directly to `production` and are cherry-picked back to `main`.
- **Works well with Merge Requests** — GitLab's MR workflows (with approvals, environment protections) fit perfectly.
- **Low merge conflict risk** — short-lived feature branches and frequent integration reduce conflicts.

### ❌ Cons

- **Branch proliferation** — maintaining environment branches (`main`, `staging`, `production`) alongside feature branches adds management overhead.
- **Cherry-picking risk** — in the release branches variant, cherry-picking hotfixes from `production` to `main` can be error-prone.
- **Environment branches can diverge** — if deployments to `staging` are slow, `staging` can lag behind `main`, creating a staleness problem.
- **Not true TBD** — `main` isn't always production-ready (code goes through `staging` first), so it's not suitable for every-commit production deployments.
- **Requires CI/CD maturity** — full value is realised only when environment promotions are automated.
- **Upstream-only rule requires discipline** — teams must enforce that you never merge `production` back into `main`.
- **Release branches variant adds complexity** — managing cherry-picks across many release branches is labour-intensive.

---

## Industry Practices & Customisations

### Common Industry Adaptations

#### 1. Branch-Based CI/CD Pipeline Triggers
GitLab natively supports branch-specific pipeline configurations:
```yaml
# .gitlab-ci.yml example
stages:
  - build
  - test
  - deploy

deploy-dev:
  stage: deploy
  environment:
    name: development
  script: ./deploy.sh dev
  only:
    - /^feature\/.*/

deploy-qa:
  stage: deploy
  environment:
    name: qa
  script: ./deploy.sh qa
  only:
    - main

deploy-uat:
  stage: deploy
  environment:
    name: staging
    url: https://staging.yourapp.com
  script: ./deploy.sh uat
  only:
    - staging

deploy-prod:
  stage: deploy
  environment:
    name: production
    url: https://yourapp.com
  script: ./deploy.sh prod
  when: manual  # Manual approval required
  only:
    - production
```

#### 2. Merge Request (MR) as the Control Point
All environment promotions happen via MRs, not direct pushes:
```
New MR: main → staging  (requires: 2 approvers, CI green, QA sign-off)
New MR: staging → production (requires: release manager approval, UAT sign-off)
```
This provides a complete audit log of every promotion.

#### 3. Protected Branch Rules
Standard enterprise configuration:

| Branch | Push Protection | Merge Requires |
|--------|----------------|---------------|
| `main` | No direct push | PR + 1 approver + CI |
| `staging` | No direct push (only from `main`) | Release manager + UAT ticket |
| `production` | No direct push (only from `staging`) | CTO/VP approval + change ticket |

#### 4. Slack/Jira Integration for Promotion Notifications
Teams integrate notifications at each promotion event:
```
✅ [STAGING] main → staging promoted by @alice
   Build: #1234 | Commit: abc1234
   Deployed to: https://staging.yourapp.com
```

#### 5. Environment-Specific Configuration Management
Each environment branch maps to a different configuration:
```
main       → .env.qa
staging    → .env.uat
production → .env.prod
```
Secrets managed via GitLab CI/CD variables scoped to environment.

#### 6. Release Notes Automation
When a `main → staging → production` merge happens:
- Automated changelog generated from MR titles and conventional commits
- Release tag created with semver: `v2.3.0`
- GitLab Release page updated automatically

#### 7. The Downstream-Only Rule (Enforced via CODEOWNERS)
A `CODEOWNERS` file or MR template reminds contributors:
```markdown
## MR Checklist
- [ ] This MR only flows DOWNSTREAM (feature → main, main → staging, staging → production)
- [ ] No code is being merged upstream (never production → main)
- [ ] Bug fixes in production have been cherry-picked to main
```

#### 8. Cherry-Pick Strategy for Hotfixes
Hotfix discipline in GitLab Flow:
```bash
# 1. Create hotfix branch from production
git checkout -b hotfix/payment-crash production

# 2. Fix and commit
git commit -m "fix: prevent null pointer in payment processor"

# 3. Open MR to production (emergency approval)
# 4. After merge, cherry-pick to main to prevent regression
git checkout main
git cherry-pick <hotfix-commit-sha>
git push origin main
```

#### 9. Tagging Strategy for Production Deployments
Every production merge is tagged:
```bash
git tag -a v2.3.1 -m "Hotfix: payment crash fix"
git push origin v2.3.1
```
Tags serve as rollback points and changelog anchors.

#### 10. Multi-Region / Multi-Tenant Production
Enterprise teams with multiple production environments extend the model:
```
production-eu  → EU region deployment
production-us  → US region deployment
production-apac → APAC region deployment
```
All environment branches receive merges from `staging` but can be promoted independently.

---

## Summary Diagram

### Environment Branches Variant

```
  feature/* (DEV)
       │ MR
       ▼
  ┌──────────────────────────────────┐
  │         main (QA)                │
  └──────────────────┬───────────────┘
                     │ MR (downstream only)
                     ▼
  ┌──────────────────────────────────┐
  │        staging (UAT)             │
  └──────────────────┬───────────────┘
                     │ MR (downstream only)
                     ▼
  ┌──────────────────────────────────┐
  │       production (PROD)          │
  └──────────────────────────────────┘
```

### Release Branches Variant

```
          main (QA) ──────────────────────────────────────────►
               │                    │
        release/2.3 (UAT)    release/2.4 (UAT)
               │ tag                │ tag
               ▼                    ▼
             PROD               PROD (2.4)
```

---

*Document version: 1.0 | Last updated: March 2026*
