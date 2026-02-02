# PHASE 11: DEPLOYMENT - PRODUCTION CHECKLIST

## 🎯 Deployment Philosophy

**Deployment is not a moment. It's a process.**

```
Traditional: Build → Test → Deploy (pray nothing breaks)
Professional: Build → Test → Stage → Validate → Canary → Full → Monitor
```

**Failure rate difference:**
- Traditional: ~15% failure rate (rollbacks, incidents)
- Professional: <0.1% failure rate (controlled, automated)

**This phase reduces risk from "fingers crossed" to "mathematically proven."**

---

## 📋 Pre-Deployment Checklist (Week Before)

### Infrastructure Verification (Owner: SRE/DevOps)

```yaml
Network & Firewall:
  [ ] Load balancer healthy (health checks passing)
  [ ] Security groups configured (no unexpected blocks)
  [ ] WAF rules deployed (DDoS protection)
  [ ] CDN origin configured (pointing to correct load balancer)
  [ ] DNS pointing to staging (verify before flipping to prod)
  [ ] SSL certificates valid (expiry > 30 days, no warnings)

Database:
  [ ] Production database backed up (< 1 hour old backup)
  [ ] Backup verified restorable (actually restore to test env)
  [ ] Replication working (primary → replica lag < 100ms)
  [ ] Monitoring alerts set (CPU, memory, connections)
  [ ] Capacity planned (growth for next 3 months)
  [ ] Slow query log enabled
  [ ] Connection pooling configured

Compute:
  [ ] Server capacity verified (CPU < 60%, memory < 70%)
  [ ] Auto-scaling policies set (add servers when CPU > 70%)
  [ ] Kubernetes nodes healthy (if K8s: all nodes ready)
  [ ] Container registry accessible (images pull without errors)
  [ ] Resource limits set (prevent one pod killing cluster)

Storage:
  [ ] S3 buckets created + versioning enabled
  [ ] Backup strategy verified (cross-region replication)
  [ ] Object lifecycle policies set (old versions deleted)
  [ ] Permissions correct (principle of least privilege)

Secrets:
  [ ] All secrets injected via vault (not in code/config)
  [ ] Secret rotation configured (every 90 days auto-rotate)
  [ ] Backup keys stored securely (offline, encrypted)
  [ ] Secrets readable by app but not by humans

Monitoring & Logging:
  [ ] Log aggregation configured (CloudWatch, Datadog, ELK)
  [ ] Metrics collected (CPU, memory, requests, errors)
  [ ] Alerts configured (PagerDuty integration)
  [ ] Dashboards created (visualize what matters)
  [ ] Retention policy set (logs deleted after 30 days)
```

### Application Readiness (Owner: Engineering Lead)

```yaml
Code Quality:
  [ ] All tests passing (unit + integration + E2E)
  [ ] Coverage > 80% (code coverage report generated)
  [ ] Linting passed (no warnings, all auto-fixable issues fixed)
  [ ] Type checking passed (TypeScript strict mode, no errors)
  [ ] Security scan passed (OWASP top 10, dependency vulnerabilities)
  [ ] Performance benchmarks met (no regressions)

Build Validation:
  [ ] Production build succeeds (npm run build, no errors)
  [ ] Bundle size acceptable (< 150KB gzipped for SPAs)
  [ ] No console errors/warnings in production build
  [ ] Assets fingerprinted (cache-busting for CDN)
  [ ] Source maps excluded from production (security)

Configuration:
  [ ] Environment variables all set (no defaults in prod config)
  [ ] Feature flags configured (disable new features if risky)
  [ ] API endpoints point to production (not staging)
  [ ] Error tracking configured (Sentry, Rollbar)
  [ ] Analytics configured (GA, Mixpanel)
  [ ] Email provider configured (SendGrid, AWS SES)

Dependencies:
  [ ] All dependencies pinned to versions (reproducible builds)
  [ ] No dev dependencies in production build
  [ ] License check passed (no GPL dependencies if proprietary)
  [ ] Vulnerability scan passed (npm audit, trivy, snyk)

Deployment Package:
  [ ] Docker image built and tested
  [ ] Docker image tagged (production, commit hash)
  [ ] Docker image pushed to registry (verified pullable)
  [ ] Container startup time < 30 seconds
  [ ] Health check endpoint working (readiness + liveness probes)
```

### Data & Migrations (Owner: Database Admin / Backend Lead)

```yaml
Database Migrations:
  [ ] All migrations written and tested
  [ ] Rollback migrations written (every up has a down)
  [ ] Migrations tested on staging data (same volume as prod)
  [ ] Estimated run time < 5 minutes (no long locks)
  [ ] Zero-downtime plan (if large table: use pt-online-schema-change)

Data Validation:
  [ ] Production data backed up (pre-migration snapshot)
  [ ] Data consistency checks written (verify counts, checksums)
  [ ] Foreign key constraints verified (no orphaned records)
  [ ] Indexes analyzed (will index bloat slow queries?)
  [ ] Baseline metrics collected (before migration)

Risk Mitigation:
  [ ] Migration schedule communicated (users know when it happens)
  [ ] Rollback tested (restore from backup, verify recovery time)
  [ ] SRE on-call notified (weekend deployment? SRE must be available)
  [ ] Escalation path clear (who to call if data corruption)
```

### Integration Testing (Owner: QA)

```yaml
Third-Party Services:
  [ ] Stripe connected (test mode → production mode verified)
  [ ] Email service working (test email → production working)
  [ ] Payment gateway responding (health check passing)
  [ ] Analytics configured (events flowing to production account)
  [ ] CDN responding (test request via CDN URL successful)
  [ ] DNS resolving correctly (nslookup, dig)

End-to-End Flows:
  [ ] User signup flow tested (signup → email verification → login)
  [ ] Payment flow tested (add payment method → charge → webhook receipt)
  [ ] Admin workflow tested (create order → update → refund)
  [ ] Error handling tested (payment fails → correct error shown)
  [ ] Fallback tested (CDN down → origin still works)
```

### Deployment Plan (Owner: Release Manager)

```yaml
Deployment Strategy:
  [ ] Strategy chosen (blue-green, canary, rolling, dark launch)
  [ ] Rollback plan written (steps to roll back in 5 minutes)
  [ ] Load test results available (peak load verified)
  [ ] Gradual rollout scheduled (not all servers at once)

Communication:
  [ ] Announcement sent (Slack, status page)
  [ ] Incident commander assigned (who decides if we roll back?)
  [ ] On-call team assembled (SRE, backend, frontend, DBA)
  [ ] Runbook reviewed (everyone knows their role)
  [ ] Stakeholders notified (product, support, sales)

Approval & Sign-Off:
  [ ] Product manager approves (feature set complete)
  [ ] Tech lead approves (code quality acceptable)
  [ ] SRE approves (infrastructure ready)
  [ ] Security approves (security review passed)
```

---

## 🚀 Deployment Execution (Day Of)

### T-30 Minutes: Pre-Launch Meeting

```yaml
Participants: Release manager, incident commander, SRE, backend lead, frontend lead, DBA

Agenda (30 minutes):
  1. Walk through rollback plan (5 min)
     - Who has database access?
     - How long does restore take?
     - Who tests post-restore?

  2. Verify monitoring (5 min)
     - Error rate dashboard open
     - Performance dashboard open
     - Database metrics visible
     - All alerts configured

  3. Confirm communication (5 min)
     - Slack channel live (#deployment)
     - Status page ready
     - Support team notified
     - Customer notification drafted

  4. Review incidents (5 min)
     - Recent incidents discussed
     - Similar failure modes prevented?
     - Lessons applied?

  5. Final go/no-go (5 min)
     - Release manager: "Are we ready?"
     - Each lead: "Yes" or specific blockers
     - Record decision (audit trail)

Exit Criteria:
  [ ] All leads approve
  [ ] All systems healthy
  [ ] Communication channels open
  [ ] Rollback verified testable
```

### T-0: Database Migration (If Needed)

```bash
# Step 1: Pre-migration snapshot (backup current state)
mysqldump -u root -p production_db > pre_migration_backup.sql
BACKUP_SIZE=$(du -h pre_migration_backup.sql | awk '{print $1}')
echo "Backup created: $BACKUP_SIZE"

# Step 2: Run migration
migration_start=$(date +%s)
npm run migrate:up
migration_end=$(date +%s)
migration_time=$((migration_end - migration_start))
echo "Migration completed in $migration_time seconds"

# Step 3: Validate migration (run checks)
npm run validate:migration
if [ $? -ne 0 ]; then
  echo "Validation failed! Rolling back..."
  npm run migrate:down
  exit 1
fi

# Step 4: Data consistency check
npx ts-node scripts/verify-data-consistency.ts
if [ $? -ne 0 ]; then
  echo "Data consistency check failed! Rolling back..."
  npm run migrate:down
  exit 1
fi

echo "Migration successful: $migration_time seconds"
```

**Metrics to collect:**
- Migration duration
- Rows affected
- Lock time (if applicable)
- Data consistency scores

### T+5: Application Deployment (Canary Release)

```yaml
Strategy: Rolling canary (10% → 25% → 50% → 100%)

Step 1: Deploy to 1 server (10% traffic)
  [ ] New version deployed
  [ ] Health checks passing
  [ ] Error rate monitored (< 1%)
  [ ] Response time monitored (< 5% increase)
  → Wait 5 minutes, observe

Step 2: If healthy, deploy to 25% (25% traffic)
  [ ] 3 more servers deployed
  [ ] Error rate still < 1%
  [ ] Response time still acceptable
  → Wait 5 minutes, observe

Step 3: If healthy, deploy to 50% (50% traffic)
  [ ] 7 more servers deployed
  [ ] Database query performance stable
  [ ] No cascading failures
  → Wait 10 minutes, observe

Step 4: If healthy, deploy to 100%
  [ ] All remaining servers deployed
  [ ] All checks passing
  [ ] Declare deployment successful
  → Continue monitoring
```

**Abort if:**
```
Error rate > 5%
Response time > 50% increase
Database CPU > 90%
Database connections > 80% limit
Memory usage > 85%
Disk space < 10% free
Any critical alert firing
```

### T+30: Smoke Testing (Automated + Manual)

```yaml
Automated Smoke Tests (Run immediately post-deploy):
  [ ] Homepage loads (GET /)
  [ ] API health check passes (GET /health)
  [ ] Authentication works (POST /api/auth/login)
  [ ] Create order flow works (POST /api/orders)
  [ ] Database queries respond (< 500ms)
  [ ] CDN serving assets (CSS, JS, images load)
  [ ] Third-party integrations working (Stripe, SendGrid)

Manual Spot Checks (Release manager + product):
  [ ] Signup flow completes (account created in database)
  [ ] Login works (JWT token generated)
  [ ] User dashboard displays correctly
  [ ] Checkout flow processes payment (test card charged)
  [ ] Admin panel functions (create user, view logs)
  [ ] Mobile app connects (API version check)

Check Logs for Errors:
  [ ] No 5xx errors in application logs
  [ ] No "undefined" errors
  [ ] No database connection errors
  [ ] No timeout errors
  [ ] Migration logs show no errors

Database Sanity Checks:
  [ ] Row counts match expectations (no data loss)
  [ ] Indexes on critical tables (user lookup fast)
  [ ] No lock waits (queries not blocked)
  [ ] Replication lag < 100ms (slaves in sync)
```

### T+60: Metric Validation

```yaml
Performance Metrics (Compare to baseline):
  Metric                Before      After        Status
  ─────────────────────────────────────────────────────
  API Response Time     150ms       145ms        ✅ Better
  Error Rate            0.5%        0.4%         ✅ Better
  Database Query Time   50ms        48ms         ✅ Better
  Page Load Time        2.0s        1.9s         ✅ Better
  Worker Queue Depth    100         95           ✅ Better

System Health:
  [ ] CPU utilization: 45% (target: < 60%)
  [ ] Memory usage: 62% (target: < 75%)
  [ ] Disk usage: 78% (target: < 85%)
  [ ] Database connections: 35/100 (healthy)
  [ ] API error rate: 0.4% (< 1% threshold)
  [ ] P95 latency: 250ms (< 500ms SLA)

Business Metrics:
  [ ] Payments processing (orders flowing)
  [ ] User signups working (new users created)
  [ ] Email delivery working (password resets sent)
  [ ] Webhooks firing (Stripe webhooks received)
```

---

## 🔄 Deployment Strategies Explained

### Strategy 1: Blue-Green (Zero Downtime)

```
Blue Environment (current production)
├── Load Balancer (points here)
└── 10 servers running v1.2.3

Green Environment (new version)
├── Identical infrastructure
└── 10 servers running v1.2.4

Deployment Process:
1. Deploy v1.2.4 to green (blue stays live)
2. Run smoke tests against green
3. Flip load balancer: blue → green
4. Monitor green for 30 minutes
5. Keep blue as instant rollback

Rollback: Flip load balancer back to blue (< 1 minute)

Pros: Instant rollback, zero downtime
Cons: Need 2x infrastructure cost
```

### Strategy 2: Canary Release (Gradual Risk)

```
Traffic Split Over Time:
├── T+0: 10% → new servers (1 out of 10)
├── T+5m: 25% → new servers (3 out of 10)
├── T+10m: 50% → new servers (5 out of 10)
└── T+20m: 100% → new servers (all 10)

Monitoring Between Each Step:
- Error rate < 1%? Continue
- Response time increase < 5%? Continue
- Memory usage normal? Continue
- Database stable? Continue

If any metric fails:
→ Stop rollout
→ Analyze issue
→ Fix + redeploy, or rollback

Pros: Catch problems early (1% traffic affected vs 100%)
Cons: Deployment takes 20-30 minutes
```

### Strategy 3: Feature Flags (Code-Level Control)

```typescript
// In application code
if (featureFlags.newCheckoutFlow) {
  // New checkout logic
} else {
  // Old checkout logic
}

Deployment Process:
1. Deploy new code (feature flag OFF)
2. All traffic uses old logic (safe)
3. Smoke tests pass (internal team tests new logic)
4. Enable flag for 10% users
5. Monitor error rate + conversion
6. Gradually increase: 10% → 25% → 50% → 100%
7. Once confident, remove old code

Pros: Instant on/off without redeployment
Cons: Requires careful code changes (both versions in parallel)
```

### Strategy 4: Rolling Deployment (Gradual Replacement)

```
Servers 1-2: Deploy v1.2.4 (v1.2.3 still on servers 3-10)
→ Health checks pass? Continue

Servers 3-4: Deploy v1.2.4 (v1.2.3 still on servers 5-10)
→ Health checks pass? Continue

Servers 5-6: Deploy v1.2.4 (v1.2.3 still on servers 7-10)
→ And so on until all servers running v1.2.4

Monitor During Entire Process:
- Error rate at each step
- Database load
- Memory usage
- Request latency

Pros: Automatic (most CI/CD tools support this)
Cons: Complex if versions not compatible
```

**Recommendation for startups:** Start with feature flags + canary. Easiest to control.

---

## 🔙 Rollback Procedures

### When to Rollback (Automatic Criteria)

```yaml
Rollback Triggers (Automatic):
  Error Rate:
    - > 5% for 2 minutes → automatic rollback
    - > 10% for 30 seconds → immediate rollback

  Performance:
    - P95 latency > 2000ms for 5 minutes → rollback
    - API response time 2x baseline → rollback

  Infrastructure:
    - Database CPU > 95% sustained → rollback
    - Database connections > 95% limit → rollback
    - Memory usage > 90% sustained → rollback

  Critical Errors:
    - "Cannot connect to database" errors → immediate rollback
    - Cascading service failures → immediate rollback
    - Payment processing failures → immediate rollback

Human Decision (Manual Rollback):
    - Incident commander says "rollback"
    - Any critical business impact
    - Data corruption detected
```

### Rollback Execution (5-Minute Recovery)

```bash
#!/bin/bash
# Rollback script - keep it simple, tested, ready to run

PREVIOUS_VERSION=$(git describe --abbrev=0 --tags HEAD^)
echo "Rolling back to: $PREVIOUS_VERSION"

# Step 1: Stop current deployment
kubectl set image deployment/api api=$DOCKER_REGISTRY/api:$PREVIOUS_VERSION

# Step 2: Verify rollback
sleep 30
HEALTH=$(curl -s http://localhost:3000/health | jq '.status')
if [ "$HEALTH" != "ok" ]; then
  echo "Rollback failed! Health check returned: $HEALTH"
  exit 1
fi

# Step 3: Confirm data integrity (if migration was run)
npm run validate:data-integrity
if [ $? -ne 0 ]; then
  echo "Data integrity check failed!"
  exit 1
fi

echo "Rollback successful. Deployed: $PREVIOUS_VERSION"
```

**Test rollback weekly:**
```bash
# Every week, actually run rollback in staging to verify
npm run deploy:staging
sleep 5
npm run rollback:staging
# Verify everything works
```

---

## 📊 Post-Deployment Monitoring (24 Hours)

### Hour 1-2: Intensive Monitoring

```yaml
Every 5 Minutes:
  [ ] Error rate (should return to baseline)
  [ ] Response time (should be < 5% above baseline)
  [ ] Database connections (should normalize)
  [ ] Memory usage (should stabilize)
  [ ] Queue depth (should clear backlog)

Check Logs:
  [ ] Search for "ERROR" (should see none related to new code)
  [ ] Search for "WARN" (acceptable count?)
  [ ] Check for cascading errors (one error causing others)
  [ ] Monitor third-party integrations (stripe webhook lag?)

Team Communication:
  [ ] Post metrics to #deployment Slack every 10 minutes
  [ ] Release manager updates status page
  [ ] On-call team ready (don't leave desk yet)
```

### Hour 2-6: Active Monitoring

```yaml
Every 15 Minutes:
  [ ] Error dashboard (any spike?)
  [ ] Performance dashboard (any degradation?)
  [ ] Database health (CPU, memory, locks?)
  [ ] User complaints (check support emails/Slack)
  [ ] Log errors trending (increasing or decreasing?)

User Acceptance:
  [ ] Support team reports: "Any customer complaints?"
  [ ] Product team reports: "Feature working as intended?"
  [ ] Analytics: "User behavior normal?"
  [ ] Payment processing: "All orders successful?"

Team Presence:
  [ ] On-call team in Slack (available for questions)
  [ ] Release manager watching metrics
  [ ] SRE monitoring infrastructure
  [ ] Can be available within 5 minutes if rollback needed
```

### Hour 6-24: Background Monitoring

```yaml
Automated Alerts (PagerDuty configured):
  [ ] Error rate > 1% → page on-call engineer
  [ ] P95 latency > 500ms → page on-call engineer
  [ ] Database CPU > 80% → page on-call engineer
  [ ] Any critical errors → page on-call engineer

Manual Checks (4x per day):
  [ ] Morning: Review overnight metrics (any issues?)
  [ ] Afternoon: Spot check key features
  [ ] Evening: Review user feedback
  [ ] Before bed: Final health check

Success Criteria:
  [ ] No rollbacks in 24 hours
  [ ] No customer-facing errors
  [ ] Metrics in acceptable range
  [ ] No on-call incidents
  [ ] All features working as intended

Deployment Complete:
  [ ] Close deployment issue on GitHub
  [ ] Update status page (mark as complete)
  [ ] Schedule post-mortem if any issues
  [ ] Celebrate deployment! 🎉
```

---

## 🛡️ Safety Practices (Non-Negotiable)

### Never Do These Things

```yaml
❌ NEVER:
  - Deploy on Friday afternoon (rollback on Monday? Disaster)
  - Deploy without smoke tests
  - Deploy without testing rollback first
  - Deploy without monitoring setup
  - Deploy when on-call team is understaffed
  - Deploy during peak traffic hours
  - Deploy in silence (no communication)
  - Deploy database migrations + app code together (decouple them)
  - Deploy feature without feature flag (can't disable if broken)
  - Deploy if any team member has concerns

✅ ALWAYS:
  - Deploy in morning (< 2pm so team can monitor all day)
  - Deploy with full team present (not at night, not on weekends)
  - Deploy with tested rollback plan
  - Deploy with monitoring + alerts active
  - Deploy gradually (not all servers at once)
  - Deploy with communication (update everyone)
  - Deploy database migrations separately (24 hours before app code)
  - Deploy behind feature flags (easy on/off)
  - Deploy only if all team approves
```

### Communication Template

```markdown
## 🚀 Deployment: Feature X

**Time**: Tuesday, Jan 24, 2pm IST
**Duration**: ~30 minutes
**Strategy**: Canary release (10% → 100%)
**Rollback Time**: < 5 minutes if needed

**What's Changing**:
- New checkout flow (speeds up payment processing)
- Updated database schema (adding payment_method_id)
- Three new API endpoints (/api/checkout, /api/payment-methods, /api/refund)

**What Users Will See**:
- Checkout page redesign (cleaner, faster)
- One less form field
- Faster payment processing (< 2 seconds)

**Risks & Mitigations**:
- Risk: Payment processing breaks
  - Mitigation: Feature flag allows instant disable
- Risk: Database migration fails
  - Mitigation: Rollback plan tested, SRE on-call
- Risk: New code has bugs
  - Mitigation: 100% test coverage, staging tested

**If Something Goes Wrong**:
- Error rate spikes → automatic rollback (< 30 seconds)
- Manual rollback: Click "Rollback" in deployment dashboard
- Questions? Ask in #deployment Slack

**Stakeholder Approvals**:
- ✅ Product Manager (feature complete)
- ✅ Tech Lead (code quality)
- ✅ SRE (infrastructure ready)
- ✅ Security (no vulnerabilities)

**Support Team Briefing**:
- New checkout UI (link to training video)
- Common issues & solutions (document prepared)
- Escalation: Page backend team if payments broken

**Status Updates**:
- T+0: Deployment started
- T+10: 25% traffic on new version
- T+20: 50% traffic on new version
- T+30: 100% traffic, all systems nominal
- T+60: Post-deployment checks complete

Questions before we proceed?
```

---

## 📈 Deployment Metrics Dashboard

```yaml
Track These Metrics:

Deployment Frequency:
  - How often do we deploy?
  - Target: 1x per week for features, 1x per day for hotfixes
  - Measure: Deployments per month

Lead Time for Changes:
  - How long from code commit to production?
  - Target: < 2 hours for hotfixes, < 1 day for features
  - Measure: Average commit-to-prod time

Deployment Failure Rate:
  - How many deployments require rollback?
  - Target: < 5% (1 rollback per 20 deployments)
  - Measure: Rollbacks / Total deployments

Mean Time to Recovery (MTTR):
  - How long to fix broken deployment?
  - Target: < 5 minutes (rollback is fast)
  - Measure: Incident detection time to rollback

Change Failure Rate:
  - How many deployments cause incidents?
  - Target: < 15% (1 incident per 6 deployments)
  - Measure: Incidents caused by deployments / Total deployments

Availability:
  - How much downtime caused by deployments?
  - Target: < 99.9% (max 43 minutes downtime per month)
  - Measure: Uptime percentage

Team Safety:
  - How many on-call incidents per deployment?
  - How many escalations?
  - How much overtime?
  - Target: < 0.5 incidents per deployment
```

---

## 🏁 Deployment Sign-Off Checklist

```
PRE-DEPLOYMENT (48 hours before):
  [ ] Release notes written (what changed, why, who should care)
  [ ] Deployment plan reviewed (strategy, rollback, timeline)
  [ ] Infrastructure verified (all systems healthy)
  [ ] Database migrations tested (tested on prod-like environment)
  [ ] Smoke tests passing (all critical flows work)
  [ ] Monitoring configured (dashboards, alerts, SLAs)
  [ ] Team briefed (everyone knows timeline, roles, procedures)
  [ ] Stakeholders notified (product, support, sales, customers)

DEPLOYMENT DAY:
  [ ] Pre-launch meeting completed (30 min before)
  [ ] All team members present and ready
  [ ] Slack channels open (#deployment, #incidents)
  [ ] Incident commander assigned
  [ ] On-call team assembled
  [ ] Rollback plan reviewed one final time

DEPLOYMENT EXECUTION:
  [ ] Database migrations completed (if needed)
  [ ] Application deployed to canary (10%)
  [ ] Smoke tests passing on canary
  [ ] Metrics normal for 5 minutes
  [ ] Application deployed to 25%
  [ ] Error rate < 1%, response time acceptable
  [ ] Application deployed to 50%
  [ ] Database still healthy, no cascading failures
  [ ] Application deployed to 100%
  [ ] Manual spot checks pass (product team verifies)

POST-DEPLOYMENT (24 hours):
  [ ] Hour 1: Intensive monitoring (team at desk)
  [ ] Hour 2-6: Active monitoring (ready to rollback)
  [ ] Hour 6-24: Background monitoring (alerts configured)
  [ ] No critical issues requiring rollback
  [ ] All user-facing features working
  [ ] Performance metrics acceptable
  [ ] Support team reports zero deployment-related complaints

SIGN-OFF:
  Release Manager: _________________ Date: _______
  Incident Commander: _____________ Date: _______
  Product Manager: ________________ Date: _______
  SRE Lead: _____________________ Date: _______
```

---

## 🚨 Common Deployment Failures & Prevention

| Failure | Why It Happens | How to Prevent |
|---------|---|---|
| **Forgot to update API URL** | Config copied from staging, never updated | Use environment variable, not hardcoded |
| **Database migration broke** | Ran migration without testing first | Test migration on backup of prod data |
| **New code incompatible with old** | Feature not gated behind feature flag | Deploy behind feature flag, decouple release |
| **Memory leak causing OOM** | New code not tested under load | Load test before deployment |
| **External service down** | Didn't check third-party health | Add health check to deployment checklist |
| **Secrets not in environment** | Forgot to set env var in production | Use automation to inject secrets, test in staging |
| **Database connections exhausted** | New code opens connections, doesn't close | Connection pooling + monitoring connection count |
| **CDN cache serving old version** | Cache busting not configured | Fingerprint assets + set cache-control headers |
| **Forgot to run database migrations** | Migration script not in deploy pipeline | Automatic migration in deploy script, verify it ran |
| **Rollback takes 30 minutes** | Rollback procedure not tested | Test rollback weekly, keep it < 5 minutes |

---

## 📚 One-Page Deployment Truth

**Deployment is risk management.**

**Three Deployment Laws:**

1. **Test everything in staging first (no exceptions)**
   - If it fails in prod, it failed in staging too
   - Staging != prod? Fix staging to match prod

2. **Deploy gradually (not all servers at once)**
   - Canary (10%) first, watch for 5 minutes
   - Catch bugs at 1% impact, not 100% impact

3. **Monitor obsessively (for 24 hours post-deploy)**
   - Error rate trending up? Rollback immediately
   - P95 latency spiked? Investigate or rollback
   - Any critical error? Don't wait, act fast

**Three Rollback Rules:**

1. **Rollback must be < 5 minutes (or fix deployment process)**
   - If it takes 30 minutes, you'll hesitate
   - Fast rollback = easy decisions

2. **Test rollback weekly (or it's broken)**
   - Actually run rollback in staging
   - Verify it works before you need it

3. **Communicate rollback (don't hide failures)**
   - Tell users what happened
   - Tell team what to fix
   - Tell investors what you learned

**Success = Boring Deployments**

Exciting deployments = something went wrong.
Boring deployments = everything went according to plan.

Target boring deployments.
```

---

## 🎯 Graduation Criteria for Phase 11

```yaml
Deployment Process:
  [ ] Written runbook for every deployment
  [ ] Automated smoke tests (run post-deploy)
  [ ] Monitoring configured (dashboards + alerts)
  [ ] Rollback tested weekly
  [ ] Rollback time measured (< 5 minutes)

Infrastructure:
  [ ] Staging environment identical to production
  [ ] Database backups automated + tested
  [ ] Load balancer health checks active
  [ ] CDN configured + tested
  [ ] Secrets management configured

Safety:
  [ ] Feature flags for every risky feature
  [ ] Database migrations decouple from code
  [ ] Canary deployments automated
  [ ] Kill switches for features/services
  [ ] Circuit breakers for external services

Monitoring:
  [ ] Error rate dashboard visible
  [ ] Performance metrics trending
  [ ] Business metrics dashboards (revenue, signups)
  [ ] Alerting configured (PagerDuty)
  [ ] On-call runbooks prepared

Team Capability:
  [ ] Every team member can deploy
  [ ] Every team member can rollback
  [ ] Post-mortems after incidents
  [ ] Deployment metrics tracked
  [ ] Deployment process improved monthly

Approval:
  [ ] Engineering Lead: _______________
  [ ] SRE Lead: ___________________
  [ ] VP Engineering: _______________
  [ ] Product Lead: _______________
```

---

## 💡 Advanced: Progressive Delivery Roadmap

**Phase 11 (Now):** Manual canary releases + feature flags
**Phase 12:** Blue-green deployments + automated rollback
**Phase 13:** Multi-region deployments + real-time replication
**Phase 14:** GitOps (Argo CD, Flux) + declarative infrastructure
**Phase 15:** Chaos engineering + "Game Days" (simulated failures)

---

**Deployment is where theory meets reality. Master this phase, and you never wake up at 3am to a broken system.**

