# PHASE 11: CI/CD PIPELINE SETUP

## Overview
CI/CD (Continuous Integration/Continuous Deployment) transforms code commits into deployed software through automated pipelines. A well-designed pipeline catches issues early, enforces quality gates, and enables safe, frequent deployments. Without CI/CD, you're relying on human memory and manual processes—both unreliable.

***

## 11.1 CI/CD PHILOSOPHY

### 11.1.1 Core Principles

**Continuous Integration principles:**
- Every commit triggers automated build and tests
- Fix broken builds immediately (within 10 minutes)
- Keep the build fast (<15 minutes for core feedback)
- Everyone commits at least daily
- All tests pass before merge

**Continuous Deployment principles:**
- Every successful build is deployable
- Deployment is automated, repeatable, and auditable
- Production deployments happen frequently (daily or more)
- Rollback is automated and fast
- Feature flags decouple deployment from release

**Pipeline design goals:**
```
┌─────────────────────────────────────────────────────────┐
│                   IDEAL PIPELINE                         │
├─────────────────────────────────────────────────────────┤
│  Fast feedback    → Results in <15 minutes              │
│  Reliable         → Same inputs = same outputs          │
│  Comprehensive    → Catches issues before production    │
│  Auditable        → Who deployed what, when, why        │
│  Secure           → No secrets exposed, signed builds   │
└─────────────────────────────────────────────────────────┘
```

**Output:** CI/CD principles document

***

### 11.1.2 Pipeline Architecture

**Standard pipeline stages:**

```
┌──────────────────────────────────────────────────────────────────────┐
│                          CI/CD PIPELINE                               │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────┐   ┌──────────┐   ┌─────────┐   ┌──────────┐   ┌───────┐│
│  │  BUILD  │ → │   TEST   │ → │ ANALYZE │ → │  DEPLOY  │ → │VERIFY ││
│  │         │   │          │   │         │   │          │   │       ││
│  │• Install│   │• Unit    │   │• SAST   │   │• Staging │   │• Smoke││
│  │• Compile│   │• Integr. │   │• DAST   │   │• Prod    │   │• E2E  ││
│  │• Bundle │   │• E2E     │   │• Deps   │   │• Rollout │   │• Perf ││
│  └─────────┘   └──────────┘   └─────────┘   └──────────┘   └───────┘│
│       │              │             │              │            │     │
│       └──────────────┴─────────────┴──────────────┴────────────┘     │
│                            FEEDBACK LOOP                              │
│                    (Fail fast, notify immediately)                    │
└──────────────────────────────────────────────────────────────────────┘
```

**Stage breakdown:**

| Stage | Trigger | Duration | Failure Action |
|-------|---------|----------|----------------|
| Build | Every push | 2-5 min | Block merge |
| Unit Tests | Every push | 2-5 min | Block merge |
| Integration Tests | Every PR | 5-10 min | Block merge |
| Security Scan | Every PR | 5-10 min | Warn or block |
| Deploy to Staging | Merge to main | 5-10 min | Alert team |
| E2E Tests | After staging deploy | 10-20 min | Block production |
| Production Deploy | Manual trigger/auto | 5-10 min | Auto-rollback |
| Post-Deploy Verify | After production | 5 min | Alert on-call |

**Output:** Pipeline architecture diagram

***

## 11.2 GIT BRANCHING STRATEGY

### 11.2.1 Branch Strategy Selection

**Common strategies:**

| Strategy | Best For | Complexity |
|----------|----------|------------|
| **GitHub Flow** | Simple, continuous deployment | Low |
| **GitFlow** | Scheduled releases, multiple versions | High |
| **Trunk-Based** | High-frequency deployment, feature flags | Medium |
| **Release Flow** | Large teams, release trains | Medium-High |

**GitHub Flow (Recommended for most):**
```
main (production-ready)
├── feature/add-user-auth
├── feature/payment-integration
├── fix/login-bug
└── hotfix/security-patch

Rules:
1. main is always deployable
2. Branch from main, PR back to main
3. PRs require review and passing tests
4. Merge triggers deployment to staging
5. Manual promotion to production (or auto with gates)
```

**Trunk-Based Development (Advanced teams):**
```
main (trunk)
├── Short-lived feature branches (<2 days)
└── All code behind feature flags if incomplete

Rules:
1. Everyone commits to main daily
2. Incomplete features hidden by flags
3. All commits must pass tests
4. Instant deployment capability
```

**Output:** Branching strategy document with flow diagram

***

### 11.2.2 Branch Naming Conventions

**Standard naming:**
```
Type/description-with-dashes

Types:
- feature/  → New functionality
- fix/      → Bug fixes
- hotfix/   → Urgent production fixes
- refactor/ → Code improvements (no behavior change)
- docs/     → Documentation only
- test/     → Test additions/improvements
- chore/    → Maintenance tasks

Examples:
- feature/user-authentication
- fix/order-calculation-error
- hotfix/security-vulnerability
- refactor/extract-payment-service
```

**Branch protection rules:**
```yaml
# main branch protection
main:
  required_reviews: 1
  dismiss_stale_reviews: true
  require_status_checks:
    - build
    - test
    - lint
    - security-scan
  require_linear_history: true
  restrict_push: maintainers_only
```

**Output:** Branch naming conventions document

***

## 11.3 CODE REVIEW PROCESS

### 11.3.1 Pull Request Guidelines

**PR requirements:**
```
Title: [TYPE] Brief description of change
  - [FEAT] Add user authentication
  - [FIX] Correct order total calculation
  - [REFACTOR] Extract payment processing logic

Description template:
## Summary
What does this PR do? Why is it needed?

## Changes
- Added X
- Modified Y
- Removed Z

## Testing
- How was this tested?
- Screenshots if UI change

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
```

**PR size guidelines:**
- Ideal: <200 lines changed
- Acceptable: 200-500 lines
- Too large: >500 lines (split into smaller PRs)
- Exception: Generated files, migrations

**Output:** PR template file

***

### 11.3.2 Code Review Checklist

**Reviewer checklist:**
```
Functionality:
- [ ] Code does what the PR claims
- [ ] Edge cases handled
- [ ] Error handling appropriate
- [ ] No obvious bugs

Security:
- [ ] No hardcoded secrets
- [ ] Input validation present
- [ ] Authorization checked
- [ ] No SQL injection / XSS risks

Performance:
- [ ] No obvious N+1 queries
- [ ] No unnecessary computations
- [ ] Appropriate caching considered

Code Quality:
- [ ] Follows project conventions
- [ ] Names are clear and descriptive
- [ ] DRY (Don't Repeat Yourself)
- [ ] SOLID principles applied

Testing:
- [ ] Tests exist and are meaningful
- [ ] Tests cover happy path and edge cases
- [ ] No flaky tests introduced

Documentation:
- [ ] Complex logic is commented
- [ ] API changes documented
- [ ] README updated if needed
```

**Review response times:**
```
Target: Review started within 4 hours
Maximum: Review completed within 24 hours
Urgent (hotfix): Review within 1 hour
```

**Output:** Code review guidelines document

***

## 11.4 BUILD AUTOMATION

### 11.4.1 Build Configuration

**Build responsibilities:**
```
1. Install dependencies (deterministic)
   - Lock file required (package-lock.json, yarn.lock)
   - Cache dependencies for speed
   
2. Compile/Transpile
   - TypeScript → JavaScript
   - SCSS → CSS
   - JSX → JavaScript
   
3. Bundle/Optimize
   - Tree shaking
   - Minification
   - Code splitting
   
4. Generate artifacts
   - Docker images
   - Static files
   - Lambda packages
```

**Example GitHub Actions build:**
```yaml
name: Build

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
          retention-days: 7
```

**Build optimization tips:**
- Cache dependencies (save 1-3 minutes)
- Parallelize independent jobs
- Use incremental builds when possible
- Skip unchanged projects in monorepos

**Output:** Build pipeline configuration

***

### 11.4.2 Containerization

**Docker build best practices:**
```dockerfile
# ✅ GOOD: Multi-stage build, minimal runtime image
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:20-alpine AS runtime
WORKDIR /app

# Non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Copy only necessary artifacts
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules

USER nextjs
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

**Image tagging strategy:**
```
Tags:
- :latest          → Latest main build (mutable)
- :1.2.3           → Semantic version (immutable)
- :sha-abc123      → Git SHA (immutable, for debugging)
- :pr-456          → PR build (for preview environments)

Convention:
registry.example.com/app-name:tag
```

**Container security:**
- Scan images for vulnerabilities (Trivy, Snyk)
- Use minimal base images (Alpine, Distroless)
- Don't run as root
- No secrets in image layers

**Output:** Docker build configuration

***

## 11.5 TEST AUTOMATION IN PIPELINE

### 11.5.1 Test Stage Configuration

**Test execution order:**
```yaml
test:
  runs-on: ubuntu-latest
  
  services:
    postgres:
      image: postgres:15
      env:
        POSTGRES_PASSWORD: test
        POSTGRES_DB: test_db
      ports:
        - 5432:5432
      options: >-
        --health-cmd pg_isready
        --health-interval 10s
        --health-timeout 5s
        --health-retries 5
  
  steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Lint
      run: npm run lint
    
    - name: Type check
      run: npm run typecheck
    
    - name: Unit tests
      run: npm run test:unit -- --coverage
    
    - name: Integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgres://postgres:test@localhost:5432/test_db
    
    - name: Upload coverage
      uses: codecov/codecov-action@v4
      with:
        files: ./coverage/lcov.info
        fail_ci_if_error: true
```

**Parallelization:**
```yaml
test:
  strategy:
    matrix:
      shard: [1, 2, 3, 4]
  
  steps:
    - name: Run tests (shard ${{ matrix.shard }})
      run: npm run test -- --shard=${{ matrix.shard }}/4
```

**Output:** Test pipeline configuration

***

### 11.5.2 Coverage Gates

**Coverage enforcement:**
```yaml
# Fail if coverage drops
- name: Check coverage thresholds
  run: |
    npm run test:coverage -- --coverageThreshold='{
      "global": {
        "statements": 70,
        "branches": 65,
        "functions": 70,
        "lines": 70
      }
    }'
```

**Coverage reporting:**
```yaml
# Post coverage to PR comment
- name: Coverage Report
  uses: davelosert/vitest-coverage-report-action@v2
  if: github.event_name == 'pull_request'
```

**Output:** Coverage policy document

***

## 11.6 DEPLOYMENT AUTOMATION

### 11.6.1 Environment Management

**Environment structure:**
```
┌─────────────────────────────────────────────────────────────┐
│                    ENVIRONMENTS                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐ │
│  │    Local     │ →   │   Staging    │ →   │  Production  │ │
│  │              │     │              │     │              │ │
│  │ • Developer  │     │ • Pre-prod   │     │ • Live users │ │
│  │ • Fast       │     │ • Prod-like  │     │ • Monitored  │ │
│  │ • Mocks OK   │     │ • E2E tested │     │ • Audited    │ │
│  └──────────────┘     └──────────────┘     └──────────────┘ │
│                                                              │
│  Optional: Preview environments per PR                       │
└─────────────────────────────────────────────────────────────┘
```

**Environment configuration:**
```yaml
# Environment-specific config (not secrets!)
development:
  LOG_LEVEL: debug
  API_URL: http://localhost:3000
  FEATURE_NEW_UI: true

staging:
  LOG_LEVEL: info
  API_URL: https://api.staging.example.com
  FEATURE_NEW_UI: true

production:
  LOG_LEVEL: warn
  API_URL: https://api.example.com
  FEATURE_NEW_UI: false  # Feature flag off in prod
```

**Secrets per environment:**
- Use separate secrets for each environment
- Never use production secrets in staging
- Rotate secrets independently

**Output:** Environment configuration matrix

***

### 11.6.2 Deployment Strategies

**Deployment options:**

| Strategy | Risk | Complexity | Rollback Speed |
|----------|------|------------|----------------|
| **Big Bang** | High | Low | Slow |
| **Rolling** | Medium | Medium | Medium |
| **Blue-Green** | Low | Medium-High | Instant |
| **Canary** | Very Low | High | Instant |
| **Feature Flags** | Very Low | Medium | Instant (toggle) |

**Rolling deployment:**
```
┌────────────────────────────────────────────────────────┐
│  ROLLING DEPLOYMENT                                     │
├────────────────────────────────────────────────────────┤
│                                                         │
│  Step 1: [v1] [v1] [v1] [v1]  → All running v1         │
│  Step 2: [v2] [v1] [v1] [v1]  → 25% on v2              │
│  Step 3: [v2] [v2] [v1] [v1]  → 50% on v2              │
│  Step 4: [v2] [v2] [v2] [v1]  → 75% on v2              │
│  Step 5: [v2] [v2] [v2] [v2]  → All on v2              │
│                                                         │
│  If errors spike → stop rollout, rollback              │
└────────────────────────────────────────────────────────┘
```

**Blue-Green deployment:**
```
┌────────────────────────────────────────────────────────┐
│  BLUE-GREEN DEPLOYMENT                                  │
├────────────────────────────────────────────────────────┤
│                                                         │
│  Before:                                                │
│    Load Balancer → [Blue: v1] ← Active                 │
│                    [Green: v1] ← Idle                   │
│                                                         │
│  Deploy:                                                │
│    Load Balancer → [Blue: v1] ← Active                 │
│                    [Green: v2] ← Deploy here           │
│                                                         │
│  Switch:                                                │
│    Load Balancer → [Blue: v1] ← Idle (keep for rollback) │
│                    [Green: v2] ← Active                │
│                                                         │
│  Rollback = instant switch back to Blue                │
└────────────────────────────────────────────────────────┘
```

**Output:** Deployment strategy document

***

### 11.6.3 Deployment Pipeline

**Production deployment pipeline:**
```yaml
deploy-production:
  needs: [build, test, security-scan, deploy-staging, e2e-staging]
  runs-on: ubuntu-latest
  environment:
    name: production
    url: https://example.com
  
  steps:
    - name: Download artifacts
      uses: actions/download-artifact@v4
      with:
        name: build
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
        aws-region: us-east-1
    
    - name: Deploy to ECS
      run: |
        aws ecs update-service \
          --cluster production-cluster \
          --service app-service \
          --force-new-deployment
    
    - name: Wait for deployment
      run: |
        aws ecs wait services-stable \
          --cluster production-cluster \
          --services app-service
    
    - name: Smoke tests
      run: npm run test:smoke -- --url https://example.com
    
    - name: Notify success
      if: success()
      run: |
        curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
          -d '{"text": "✅ Deployed to production: ${{ github.sha }}"}'
    
    - name: Notify failure
      if: failure()
      run: |
        curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
          -d '{"text": "❌ Production deploy failed: ${{ github.sha }}"}'
```

**Output:** Deployment pipeline configuration

***

## 11.7 ROLLBACK STRATEGIES

### 11.7.1 Automated Rollback

**Rollback triggers:**
```yaml
# Auto-rollback on health check failure
deployment:
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  
  healthCheck:
    path: /health
    interval: 10s
    timeout: 5s
    retries: 3
    
  rollback:
    trigger:
      - health_check_failure
      - error_rate > 5%
      - latency_p99 > 2s
```

**Rollback procedure:**
```
Manual rollback steps:
1. Identify the issue (monitoring, logs)
2. Decision to rollback (incident commander)
3. Execute rollback (one command/click)
4. Verify rollback successful
5. Communicate status
6. Post-incident review

Automated rollback:
1. Health check fails 3 times
2. Previous healthy version restored
3. Alert sent to on-call
4. Deployment marked as failed
```

**Database rollback considerations:**
```
If migration is backward-compatible:
  → Rollback application only, keep DB changes

If migration is NOT backward-compatible:
  → Run down migration before app rollback
  → This is why forward-only migrations are preferred
```

**Output:** Rollback runbook

***

### 11.7.2 Feature Flags for Safe Releases

**Feature flag benefits:**
- Decouple deployment from release
- Instant rollback (toggle flag off)
- A/B testing capability
- Progressive rollouts

**Feature flag implementation:**
```javascript
// Using LaunchDarkly, Unleash, or similar
const newCheckoutEnabled = await featureFlags.isEnabled(
  'new-checkout-flow',
  { userId: user.id }
);

if (newCheckoutEnabled) {
  return <NewCheckoutFlow />;
} else {
  return <OldCheckoutFlow />;
}
```

**Flag lifecycle:**
```
1. CREATED    → Flag added, default off
2. DEVELOPING → Flag on for developers/staging
3. CANARY     → Flag on for 1-5% of users
4. ROLLING    → 25% → 50% → 75% → 100%
5. PERMANENT  → Flag removed, new code becomes default
6. DEPRECATED → Old code removed

Timeline: 2-4 weeks from created to permanent
```

**Output:** Feature flag strategy document

***

## 11.8 PIPELINE SECURITY

### 11.8.1 Secrets in CI/CD

**Secrets management:**
```yaml
# ✅ SECURE: Use secrets manager
- name: Deploy
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    API_KEY: ${{ secrets.API_KEY }}

# ❌ INSECURE: Hardcoded secrets
- name: Deploy
  env:
    DATABASE_URL: "postgres://user:password@host/db"
```

**Secret security rules:**
- Never log secrets (mask in output)
- Rotate secrets regularly
- Use short-lived credentials (OIDC tokens)
- Limit secret access to required jobs only

**OIDC for cloud access (no long-lived secrets):**
```yaml
permissions:
  id-token: write  # Required for OIDC

steps:
  - name: Configure AWS credentials
    uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789:role/GitHubActions
      aws-region: us-east-1
      # No access keys needed - uses OIDC
```

**Output:** CI/CD secrets policy

***

### 11.8.2 Supply Chain Security

**Protect against compromised dependencies:**
```yaml
# Lock actions to specific SHA
- uses: actions/checkout@8ade135a41bc03ea155e62e844d188df1ea18608 # v4.1.0

# Pin dependencies in lock file
- run: npm ci  # Uses package-lock.json exactly

# Verify signatures (if available)
- name: Verify npm signature
  run: npm audit signatures
```

**Dependency scanning in CI:**
```yaml
- name: Dependency audit
  run: npm audit --audit-level=high
  
- name: License check
  run: npx license-checker --failOn "GPL-3.0"
```

**Output:** Supply chain security checklist

***

## 11.9 MONITORING & OBSERVABILITY (CI/CD)

### 11.9.1 Pipeline Metrics

**Metrics to track:**

| Metric | Target | Why |
|--------|--------|-----|
| Build success rate | >98% | CI reliability |
| Time to first feedback | <10 min | Developer productivity |
| Deployment frequency | Daily | Delivery capability |
| Change failure rate | <5% | Quality signal |
| Mean time to recovery | <1 hour | Incident response |
| Lead time (commit → prod) | <24 hours | Delivery speed |

**Dashboard:**
```
┌─────────────────────────────────────────────────────┐
│           CI/CD HEALTH DASHBOARD                     │
├─────────────────────────────────────────────────────┤
│  Build Success Rate: 98.5% (target: 98%)            │
│  Avg Build Time: 8m 32s (target: <10m)              │
│  Deploys Today: 5 (prod: 2, staging: 3)             │
│  Failed Deploys (7d): 1                             │
│  MTTR Last Incident: 23 minutes                     │
│  Queue Time: 45s avg                                │
└─────────────────────────────────────────────────────┘
```

**Output:** CI/CD metrics dashboard

***

## 11.10 DELIVERABLES & GATES

### Deliverable 1: CI/CD Architecture Document
**Contains:**
- Pipeline stages and flow
- Tool selection with justification
- Environment configuration

### Deliverable 2: Pipeline Configuration
**Contains:**
- Build pipeline (GitHub Actions, GitLab CI, etc.)
- Test automation configuration
- Deployment scripts

### Deliverable 3: Branching Strategy
**Contains:**
- Branch naming conventions
- Protection rules
- Merge requirements

### Deliverable 4: Runbooks
**Contains:**
- Deployment procedure
- Rollback procedure
- Emergency hotfix process

***

## 11.11 PHASE 11 CHECKLIST

- [ ] CI/CD platform selected and configured
- [ ] Build pipeline automated
- [ ] Test automation in pipeline
- [ ] Security scanning integrated
- [ ] Staging deployment automated
- [ ] Production deployment automated (with approval gate)
- [ ] Rollback procedure documented and tested
- [ ] Feature flag system implemented
- [ ] Secrets properly managed (no hardcoded secrets)
- [ ] Pipeline metrics dashboard available
- [ ] Branch protection rules enforced
- [ ] PR template and review process defined

***

## 11.12 ESTIMATED DURATION

**Timeline: 1-2 weeks**

- CI/CD platform setup: 1-2 days
- Build and test pipeline: 2-3 days
- Deployment automation: 2-3 days
- Rollback and feature flags: 1-2 days
- Documentation: 1 day

***

**END OF PHASE 11**

A great CI/CD pipeline is invisible—developers commit code, tests run, deployments happen, and everyone sleeps well at night knowing the system catches issues before users do.
