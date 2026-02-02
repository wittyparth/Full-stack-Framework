# PHASE 1: REQUIREMENTS ENGINEERING

## Overview
Requirements are the source of truth. Wrong requirements kill projects more than bad code. This phase determines what you build, not how you build it. Ambiguous requirements cause 40% of project delays.

***

## 1.1 REQUIREMENTS GATHERING

### 1.1.1 Stakeholder Interview Strategy

**What to do:**
- Conduct 1-on-1 interviews with each stakeholder identified in Phase 0
- Use open-ended questions (not leading questions)
- Probe for hidden pain points, not just stated desires
- Document constraints and red lines (what cannot be changed)
- Separate wants from needs (essential vs nice-to-have)
- Record each interview (with permission) for later reference

**Critical interview techniques:**
- Start with "What does your typical day look like?" (understand actual workflow, not hypothetical)
- Ask "What frustrates you most?" (reveals pain points)
- Ask "If you could change one thing, what would it be?" (prioritization signal)
- Ask "What would you NOT want in this system?" (reveals constraints)
- Ask "Who else uses the current system/process?" (identify users you might miss)
- Ask "What metrics matter to you?" (success criteria from their perspective)
- Do NOT ask "What features do you want?" (leads to feature wish list, not problem understanding)

**Interview structure:**
- 10-15 minutes: Introduction, context
- 20-30 minutes: Current state (how do they work now)
- 15-20 minutes: Pain points (what's broken)
- 10-15 minutes: Vision (what would ideal look like)
- 5-10 minutes: Prioritization (if you could fix 3 things, which 3)

**Output:** Recorded interviews + transcribed notes with themes extracted

**Sample questions by stakeholder type:**

**For End Users:**
- "Walk me through how you do [task] today"
- "What takes the most time in your current process?"
- "How do you know if something is wrong?"
- "How frequently do you do this?"
- "What happens if you can't do this task?"
- "Do you have workarounds for missing features?" (reveals what's really needed)

**For Product Managers:**
- "What's the business problem we're solving?"
- "How would we measure success?"
- "Who are we competing against, and what do they do better?"
- "What's our unfair advantage?"
- "What will we NOT build, and why?"
- "What's the timeline pressure for each feature?"

**For Finance/Operations:**
- "What's the cost of the current problem?" (financial impact)
- "What's the ROI target?" (payback period)
- "How will this affect other operations?"
- "What are the cost constraints?"

**For Security/Compliance:**
- "What regulatory requirements apply?" (GDPR, HIPAA, PCI-DSS, etc.)
- "What data sensitivity levels exist?"
- "What's our current security posture?"
- "What are audit requirements?"

***

### 1.1.2 User Personas Creation

**What to do:**
- From interviews, identify distinct user types (not everyone uses the system the same way)
- Create 3-5 personas (personas beyond 5 dilute focus)
- For each persona, document: role, goals, pain points, technical proficiency, frequency of use, decision-making power

**What makes a good persona:**
- Based on actual data from interviews (not guesses)
- Specific enough to make design decisions (if persona is "user", it's too vague)
- Represents a meaningful segment (if only 1% of users match, don't make a persona)
- Has clear motivation (why do they use the system, what do they want to accomplish)

**Persona template:**

```
NAME: [Person's role/type]
GOAL: What they're trying to accomplish
PAIN POINTS: Top 3 frustrations
FREQUENCY: How often they use the system
TECHNICAL LEVEL: Beginner/Intermediate/Advanced
CONTEXT: Where/when/how they use it
SUCCESS METRIC: How they measure if something works
QUOTE: Direct quote from interview that captures their perspective
```

**Example persona:**

```
NAME: Sarah, E-Commerce Store Manager
GOAL: Process 200 orders/day efficiently while tracking inventory in real-time
PAIN POINTS: 
  - Current system requires manual inventory updates (1 hour/day wasted)
  - Can't see trending products (guesses on ordering)
  - Inventory discrepancies cause customer complaints
FREQUENCY: All day, every day
TECHNICAL LEVEL: Intermediate (uses Excel, knows basics)
CONTEXT: Desktop computer at warehouse office, 8am-6pm
SUCCESS METRIC: Can process order, ship, update inventory in < 2 minutes per order
QUOTE: "I spend more time updating spreadsheets than actually helping customers"
```

**Output:** 3-5 detailed personas with supporting interview quotes

***

### 1.1.3 User Journey Mapping

**What to do:**
- For each persona, map the end-to-end journey (current state and desired state)
- Identify touchpoints (where they interact with system/product)
- Identify pain points at each stage
- Identify opportunities (where new system adds value)
- Map decision points (where they choose to do something or abandon)
- Include emotional state (are they frustrated, confident, uncertain)

**Current state journey example:**
```
PERSONA: Sarah, Store Manager
STAGE 1: Order Arrives (8:00am)
├─ Pain: Email arrives, no structured format
├─ Action: Open email, read order details manually
├─ Emotion: Mildly frustrated (email search is slow)
└─ Duration: 3 minutes

STAGE 2: Verify Inventory (8:03am)
├─ Pain: Must check 3 different spreadsheets
├─ Action: Switch to Excel, look up SKU, check quantity
├─ Emotion: Frustrated (spreadsheets don't sync)
├─ Decision point: Is item in stock? (determines next action)
└─ Duration: 5 minutes

STAGE 3: Pick & Pack (8:08am)
├─ Pain: Manual picking list is error-prone
├─ Action: Print picking list, walk warehouse, gather items
├─ Emotion: Anxious (wrong items shipped = complaints)
└─ Duration: 10 minutes

STAGE 4: Update Inventory (8:18am)
├─ Pain: Manual inventory update in Excel
├─ Action: Return to desk, update quantities in 3 spreadsheets
├─ Emotion: Bored & frustrated (manual data entry)
└─ Duration: 5 minutes

STAGE 5: Ship (8:23am)
├─ Pain: Generate label manually, record in another system
├─ Action: Login to shipping provider, create label, stick on box
├─ Emotion: Resigned (too many systems)
└─ Duration: 3 minutes

TOTAL TIME: 26 minutes per order × 200 orders = 86+ hours/month in process overhead
```

**Desired state journey example:**
```
PERSONA: Sarah, Store Manager
STAGE 1: Order Arrives (8:00am)
├─ System receives email, auto-parses order
├─ Real-time inventory check (system confirms stock)
└─ Duration: 30 seconds (automated)

STAGE 2: Sarah Reviews (8:00:30am)
├─ See pre-filled picking list with items highlighted
├─ One-click approval to start picking
├─ Duration: 10 seconds

STAGE 3: Pick & Pack (8:00:40am)
├─ Mobile app guides picking (barcode scan confirms correct item)
├─ Barcode scan auto-deducts inventory (no manual update needed)
└─ Duration: 8 minutes

STAGE 4: Ship (8:08am)
├─ App prints label, book shipping, update customer email
├─ All systems sync automatically
└─ Duration: 1 minute

TOTAL TIME: 10 minutes per order × 200 orders = 33 hours/month saved
```

**Key insight from journey:** System saves 53 hours/month = $5,300/month in labor costs = clear ROI

**Output:** Current state + desired state journey maps for each persona (visual + narrative)

***

### 1.1.4 Pain Point Identification & Prioritization

**What to do:**
- From all interviews and journeys, extract all pain points
- Categorize pain points (frequency, severity, impact)
- For each pain point, understand root cause (is the problem the tool, the process, the training, or the workflow)
- Prioritize (solve most impactful pain points first, defer nice-to-haves)

**Pain point analysis template:**

```
PAIN POINT: Manual inventory updates required daily
├─ FREQUENCY: Happens 20+ times/day
├─ SEVERITY: High (causes errors, conflicts with other systems)
├─ IMPACT: 
│   ├─ Time: 1 hour/day × 200 days/year = 200 hours/year
│   ├─ Quality: 5% inventory discrepancy (customer complaints)
│   └─ Cost: 200 hours × $25/hour = $5,000/year + lost sales from stockouts
├─ ROOT CAUSE: Current system doesn't auto-sync with warehouse
├─ CURRENT WORKAROUND: Export inventory hourly, manual reconciliation
└─ SOLUTION: Real-time inventory sync, barcode scanning
```

**Severity scale:**
- **Critical**: System completely unusable, stops business process (blockage)
- **High**: Significant workaround required, impacts efficiency/quality
- **Medium**: Inconvenience but workable, nice optimization
- **Low**: Cosmetic, doesn't impact core functionality

**Output:** Pain point matrix with frequency, severity, impact, root cause, solution direction

***

## 1.2 FUNCTIONAL REQUIREMENTS

### 1.2.1 User Stories with Acceptance Criteria

**What to do:**
- Convert pain points and personas into user stories
- Use format: "As a [persona], I want to [action], so that [outcome]"
- For each story, define acceptance criteria (how do you know it's done)
- Acceptance criteria must be testable (not vague)
- Estimate relative complexity (story points: 1, 2, 3, 5, 8)

**Good user story format:**

```
TITLE: Real-time Inventory Sync

AS A: Store Manager
I WANT: To see current inventory quantity update automatically when items are picked
SO THAT: I can make decisions based on accurate stock levels and avoid overselling

ACCEPTANCE CRITERIA:
✓ When item is picked from warehouse (barcode scanned), inventory decreases within 5 seconds
✓ Multiple concurrent picks don't create race condition (inventory accurate if 5 items picked simultaneously)
✓ If scanner loses connection, picks queue and sync when reconnected
✓ Admin can see real-time sync dashboard (how many items synced, error log)
✓ Page refresh shows updated inventory (not cached stale data)

STORY POINTS: 5

NOTES: 
- Need barcode scanner hardware integration
- Need real-time WebSocket or polling mechanism
- Race condition protection required
```

**Bad user story example (what NOT to do):**
```
"Add inventory management feature"
- Not user-focused (doesn't explain who or why)
- No acceptance criteria (how do you know it's done)
- Too vague (could mean anything)
```

**User story inventory template:**
- Title
- Epic (group of related stories: "Inventory Management", "Order Processing", "Reporting")
- Story (As a X, I want Y, so that Z)
- Acceptance criteria (5-10 testable criteria)
- Story points (1-8 scale)
- Blocking issues (what must be done first)
- Dependencies (which other stories must be done first)

**Output:** 30-80 user stories (depending on project size) with acceptance criteria and story points

**Typical distribution:**
- 40-50% core features (must have for MVP)
- 30-40% enhancement features (should have, nice to have)
- 10-20% technical stories (refactoring, performance, infrastructure)

***

### 1.2.2 Use Case Documentation

**What to do:**
- For critical workflows, document as use cases (more detailed than user stories)
- Include main flow (happy path), alternate flows (variations), and exception flows (errors)
- Identify actors (who initiates), preconditions (what must be true first), postconditions (what's true after)
- Use cases are more formal than user stories, useful for complex business logic

**Use case template:**

```
USE CASE: Process Order and Update Inventory

ID: UC-001
ACTOR: Store Manager
PRECONDITION: Order received, inventory system available
POSTCONDITION: Order shipped, inventory updated, customer notified

MAIN FLOW:
1. Store Manager receives notification of new order
2. System displays order details (customer, items, address)
3. Manager clicks "Approve & Ship"
4. System verifies inventory (checks if items in stock)
5. System generates picking list
6. Manager walks to warehouse, scans items with barcode scanner
7. For each item scanned:
   a. System verifies barcode matches picking list
   b. System marks item as picked
   c. System updates inventory in real-time
8. All items picked, Manager confirms pickup complete
9. System generates shipping label
10. Manager applies label to box
11. Manager scans final barcode
12. System marks order as shipped
13. System sends shipping confirmation email to customer
14. Use case ends (order fulfilled)

ALTERNATE FLOWS:

ALTERNATE A1: Item out of stock
2a. System discovers item is out of stock
2b. System notifies manager (popup alert)
2c. Manager chooses: (A1.1) Mark as backorder, (A1.2) Cancel item, (A1.3) Substitute item
A1.1: Manager marks item as backordered, order goes to "pending" status
A1.2: Manager cancels item, customer notified, proceeds with remaining items
A1.3: Manager selects substitute item, customer approval flow triggered
2d. Flow resumes at step 4 with updated items

ALTERNATE A2: Multiple orders
2a. Manager selects "Batch Process" option
2b. System displays 5 orders (or N orders)
2c. System optimizes picking (sorts by aisle to minimize walking)
2d. Manager picks all items for batch, then system routes them to correct orders
2e. Remainder same as main flow, but for multiple orders at once

EXCEPTION FLOWS:

EXCEPTION E1: Scanner hardware failure
6a. Scanner disconnects unexpectedly
6b. System detects connection loss
6c. System queues picks locally
6d. When scanner reconnects, queued picks auto-sync to server
6e. If timeout > 5 minutes, alert manager to switch to manual input

EXCEPTION E2: Network outage
5a. Inventory system becomes unavailable
5b. System shows error message: "Inventory service unavailable, try again in 30s"
5c. Manager can choose: (E2.1) Retry, (E2.2) Proceed with cached inventory (risky), (E2.3) Wait
E2.1: System retries connection every 30s until success
E2.2: Manager acknowledges risk, system allows proceed with 5-second-old inventory snapshot
E2.3: Manager waits, system monitors for reconnection

SPECIAL CASES:
- If order contains digital + physical items, physical items follow normal flow, digital delivery happens simultaneously
- If customer provides special instructions (gift wrap, custom message), picking list shows these notes
- If customer has subscription, system shows previous preferences, manager can use as defaults

NOTES:
- Entire flow must complete < 15 minutes for 200 orders/day workload
- System must handle 5 concurrent order processing streams
- If any step fails, must be able to roll back without data corruption
```

**Output:** 3-8 use cases for critical workflows (not every user story, just the complex ones)

***

### 1.2.3 Feature Prioritization (MoSCoW Method)

**What to do:**
- Classify every feature into 4 categories:
  - **MUST have**: Cannot launch without this, blocking feature
  - **SHOULD have**: Important, but can launch without if necessary
  - **COULD have**: Nice to have, add value but not critical
  - **WON'T have**: Explicitly defer (for next phase or never)
- Create prioritization matrix: impact vs effort
- Use MoSCoW to define MVP scope
- Use MoSCoW to identify Phase 1 vs Phase 2 releases

**MoSCoW example:**

```
MUST HAVE (MVP - cannot launch without):
├─ User authentication (email/password)
├─ Order creation & management
├─ Basic inventory tracking
├─ Order shipping workflow
├─ Customer notification (order confirmation, shipping notification)
└─ Basic admin dashboard (orders, inventory overview)

SHOULD HAVE (launch, but could cut if timeline slips):
├─ Real-time inventory sync
├─ Barcode scanning integration
├─ Advanced inventory forecasting
├─ Multi-warehouse support
├─ Bulk order processing
└─ Detailed analytics & reporting

COULD HAVE (post-launch nice-to-haves):
├─ Mobile app (iOS/Android)
├─ AI-powered order routing
├─ Predictive inventory management
├─ Supplier integration & auto-ordering
├─ Advanced customer segmentation
└─ Custom reporting builder

WON'T HAVE (explicit non-goals):
├─ Accounting/invoicing (integrate with existing tool)
├─ Employee time tracking (separate system)
├─ Customer loyalty program (Phase 2+)
├─ International shipping optimization (Phase 2+)
└─ Custom warehouse automation (too complex for MVP)
```

**Prioritization matrix visualization:**

```
HIGH IMPACT / LOW EFFORT (Do First):
├─ User authentication
├─ Order creation
├─ Order notification
└─ Basic inventory tracking

HIGH IMPACT / HIGH EFFORT (Do But Plan Carefully):
├─ Real-time inventory sync
├─ Advanced analytics
└─ Multi-warehouse support

LOW IMPACT / LOW EFFORT (Quick Wins):
├─ Dark mode toggle
├─ Export to CSV
└─ Keyboard shortcuts

LOW IMPACT / HIGH EFFORT (Avoid):
├─ Premium theme customization
├─ Legacy system integration (if no one using)
└─ Unsupported platforms (old browsers)
```

**Output:** MoSCoW prioritization matrix with business justification for each categorization

***

### 1.2.4 MVP Scope Definition

**What to do:**
- Define MVP (Minimum Viable Product) using MUST-have features only
- Answer: "What's the smallest thing we can ship that validates the core hypothesis?"
- List what's explicitly NOT in MVP
- Define launch criteria (what must be true to say "MVP is done")
- Estimate MVP timeline (different from full feature list)

**MVP scope example:**

```
MVP SCOPE:
✓ Can process 100+ orders/day efficiently (solves main pain point)
✓ Real-time inventory doesn't have race conditions
✓ Can ship product to 50 states in US
✓ Customer gets shipping notification
✓ Manager can see dashboard of orders/inventory

NOT IN MVP (Defer to Phase 2):
✗ Mobile app (can use responsive web)
✗ Multi-warehouse support (single warehouse first)
✗ Barcode scanning (manual input acceptable initially)
✗ Advanced analytics/forecasting (simple metrics only)
✗ Bulk order processing (single-order processing works fine)
✗ Customer-facing portal (manager-facing tool only)
✗ API for third parties (internal use only)
✗ International shipping (US only initially)

LAUNCH CRITERIA:
- Can process 100 orders without errors
- Inventory never goes negative (race conditions resolved)
- Average order processing time < 15 minutes
- Zero data loss during process
- 99% uptime SLA for 1 week of production testing
- All critical bugs fixed (no P0s, acceptable P1s with timeline)
- Documentation complete (for support team)
- Manager training complete (support staff trained)

MVP TIMELINE: 12-16 weeks from requirements to launch
Phase 1-8: 12 weeks development
Phase 9: 1 week integration testing
Phase 11-12: 1-2 weeks deployment & stabilization
```

**Output:** MVP scope document with features included, explicitly excluded, and launch criteria

***

### 1.2.5 Feature Dependency Mapping

**What to do:**
- Create dependency graph (feature A depends on feature B)
- Identify critical path (features that must be done first)
- Identify parallelizable features (can be done simultaneously)
- Use for release planning (which features can launch together, which must wait)

**Dependency example:**

```
FEATURE DEPENDENCY GRAPH:

User Authentication
  └─> User Roles & Permissions
      ├─> Order Management (can only manage own orders if permission set)
      ├─> Admin Dashboard (only admins see all orders)
      └─> Inventory Management (only managers can update)

Order Management
  ├─> Email Notifications (needs to email customer)
  │   └─> Email Service Integration (configure email provider)
  └─> Payment Processing (needs to charge customer)
      └─> Payment Gateway Integration (Stripe/Square integration)

Inventory Management
  ├─> Real-time Sync (depends on order processing to trigger picks)
  └─> Barcode Scanning (optional enhancement to manual input)

Basic Dashboard
  └─> Reports & Analytics (depends on data from other features)

CRITICAL PATH (Must be done in order):
1. User Authentication (foundation)
2. Order Management (core flow)
3. Inventory Management (core flow)
4. Email Service Integration (notifications)
5. Payment Integration (charges)
6. Dashboard (oversight)

PARALLEL WORK (Can be done simultaneously):
- Once Auth + Order Management started, can work on:
  - Inventory system
  - Email integration
  - Payment integration
  - Barcode scanning

RELEASE GATES:
- Phase 1.0 MVP: Auth + Order Management + Basic Inventory
- Phase 1.1: Add Real-time Sync + Analytics
- Phase 1.2: Add Barcode Scanning + Advanced Reports
```

**Output:** Dependency diagram showing feature prerequisites and parallelizable work

***

## 1.3 NON-FUNCTIONAL REQUIREMENTS

### 1.3.1 Performance Targets

**What to do:**
- Define specific, measurable performance targets (not "fast", but numbers)
- Performance targets drive architecture decisions (if you need p99 < 100ms, that's different from p99 < 500ms)
- Include targets for different user counts (performance at 10 users vs 10,000 users)
- Include targets for different operations (some operations are slower, that's OK if accepted upfront)

**Performance metrics to define:**

```
RESPONSE TIME TARGETS:
├─ Page Load Time (time to interactive)
│  └─ Target: < 2 seconds (desktop), < 4 seconds (mobile)
├─ API Response Time (p50/p95/p99)
│  ├─ Simple queries (list orders): p99 < 200ms
│  ├─ Complex queries (analytics): p99 < 2000ms
│  └─ Bulk operations (import 1000 orders): < 30 seconds total
├─ Search Response
│  └─ Full-text search: < 500ms
├─ Database Query Time (backend perspective)
│  ├─ Index hit: < 10ms
│  ├─ Full table scan: < 500ms
│  └─ Report generation: < 5 seconds
└─ File Upload
    └─ Upload 10MB file: < 10 seconds

THROUGHPUT TARGETS:
├─ Concurrent Users
│  ├─ Peak: 500 concurrent users
│  ├─ Sustained: 200 concurrent users
│  └─ Growth: Expected 100% YoY, plan for 1000 users in year 2
├─ Requests Per Second (RPS)
│  ├─ Peak: 5,000 RPS
│  └─ Can handle 2x peak for 5 minute burst (spike handling)
├─ Database Operations
│  ├─ Inserts: 1,000 per second
│  ├─ Updates: 500 per second
│  └─ Queries: 10,000 per second
└─ File Processing
    └─ Upload 1000 CSV rows: < 60 seconds

RESOURCE UTILIZATION TARGETS:
├─ CPU: Never exceed 80% utilization at peak load
├─ Memory: Never exceed 85% utilization
├─ Disk: Never exceed 90% used capacity
├─ Network Bandwidth: Sustained < 50% of available, peak < 80%
└─ Database Connections: Maintain 20% headroom (if pool size 100, use max 80)

CACHING TARGETS:
├─ Cache Hit Rate: > 80% for read-heavy operations
├─ Cache Miss Penalty: < 500ms (time to fetch and cache)
└─ Cache TTL:
    ├─ User data: 5 minutes (balance freshness vs load)
    ├─ Inventory: 1 minute (must be fresh for business logic)
    ├─ Reports: 1 hour (stale data acceptable)
    └─ Reference data: 24 hours (stable data)
```

**Example performance targets by scenario:**

```
SCENARIO 1: Normal Operating Day (9am-5pm)
├─ Expected users: 100-200 concurrent
├─ Expected RPS: 200-500
├─ Target p99 latency: < 300ms
├─ Target uptime: 99.9%
└─ Acceptable resources: Standard instance size

SCENARIO 2: Peak Processing Time (11am, 2pm - order rushes)
├─ Expected users: 300-500 concurrent
├─ Expected RPS: 1,000-2,000
├─ Target p99 latency: < 500ms (slower, acceptable at peak)
├─ Target uptime: 99.9%
└─ Auto-scaling triggered: Yes

SCENARIO 3: End-of-Month Report Generation (last day of month)
├─ Background job processing 1 year of data
├─ Expected time: < 5 minutes
├─ Impact on normal users: < 10% latency increase
└─ This job runs off-peak (11pm) to avoid conflicts

SCENARIO 4: File Import (manager imports 1000 order SKUs)
├─ Expected duration: < 2 minutes
├─ Can happen anytime (batched background processing)
├─ Doesn't block other operations
└─ User gets notification when complete
```

**Output:** Performance targets document with p50/p95/p99 latencies, throughput, resource limits, scenarios

***

### 1.3.2 Scalability Requirements

**What to do:**
- Define how many users/data you expect over time (growth trajectory)
- Define scaling strategy (vertical vs horizontal, when to scale)
- Define performance targets at different scales (performance at 100 users vs 100,000 users)
- Identify bottlenecks (what breaks first under load)

**Scalability planning:**

```
USER GROWTH PROJECTION:
├─ Launch (Month 0): 100 users
├─ Month 3: 500 users
├─ Month 6: 2,000 users
├─ Month 12: 10,000 users
└─ Year 2: 50,000 users

DATA GROWTH PROJECTION:
├─ Orders created/day:
│  ├─ Launch: 100 orders/day
│  ├─ Month 6: 5,000 orders/day
│  ├─ Year 1: 50,000 orders/day
│  └─ Year 2: 500,000 orders/day
├─ Database size:
│  ├─ Launch: 1GB
│  ├─ Year 1: 50GB
│  └─ Year 2: 200GB
└─ Archive strategy: Keep hot data (1 year), archive older

SCALING STRATEGY:

Phase 1 (Launch to 1,000 users):
├─ Single server architecture
├─ Database on same server (or managed service)
├─ Caching layer (Redis) added
├─ CDN for static assets
└─ Estimated cost: $500/month

Phase 2 (1,000 to 10,000 users):
├─ Separate backend servers (load balanced)
├─ Managed database (RDS/Cloud SQL)
├─ Read replicas for read-heavy operations
├─ Message queue for background jobs
└─ Estimated cost: $2,000-5,000/month

Phase 3 (10,000+ users):
├─ Horizontal scaling (auto-scaling groups)
├─ Database sharding by region (if multi-region)
├─ Dedicated caching cluster
├─ Dedicated job worker fleet
├─ CDN with regional edge caching
└─ Estimated cost: $10,000+/month

BOTTLENECK ANALYSIS (What breaks first under load):

At 1000 concurrent users:
├─ Database connection pool exhaustion (first bottleneck)
├─ Response time degrades < 500ms (acceptable)
└─ CPU utilization < 60%

At 2000 concurrent users:
├─ CPU becomes constrained (hit 80% utilization)
├─ Database query time increases (indexes help)
├─ Memory stable
└─ Mitigation: Add server replicas, improve queries

At 5000 concurrent users:
├─ Database queries become bottleneck (even with replicas)
├─ Network bandwidth increases
├─ Need read-write separation (CQRS pattern)
└─ Mitigation: Dedicated read replicas, query optimization

STRESS TEST TARGETS:
├─ 3x expected peak load: System degrades gracefully (higher latency, but no errors)
├─ 5x expected peak load: System returns errors but doesn't crash
├─ 10x expected peak load: System fails gracefully (graceful shutdown, not data corruption)
```

**Output:** Scalability plan showing growth projection, scaling strategy by phase, bottleneck analysis

***

### 1.3.3 Availability & Uptime Requirements (SLA Definition)

**What to do:**
- Define SLA (Service Level Agreement) - what uptime we promise customers
- Define SLO (Service Level Objective) - what uptime is our internal target
- Define RTO (Recovery Time Objective) - how quickly must we recover from outage
- Define RPO (Recovery Point Objective) - how much data loss is acceptable
- Consider different severity levels (partial outage vs complete outage)

**SLA definitions:**

```
SERVICE LEVEL AGREEMENT (Customer Facing):

UPTIME TARGETS:
├─ 99.0% (allows ~7 hours downtime/month)
├─ 99.9% (allows ~43 minutes downtime/month)
├─ 99.95% (allows ~22 minutes downtime/month)
├─ 99.99% (allows ~4 minutes downtime/month)
└─ Selected: 99.9% (industry standard for SaaS)

WHAT COUNTS AS DOWNTIME:
├─ System completely unavailable (returns 5xx errors)
├─ Database unavailable (can't save orders)
├─ Payment processing down (can't charge)
├─ Inventory service down (can't check stock)
└─ Does NOT count as downtime:
    └─ Reports running slow (not part of critical path)
    └─ Mobile app slow (web version available)
    └─ Optional features unavailable

DOWNTIME BUDGET:
├─ For 99.9% SLA: 43 minutes/month (5+ minutes/week buffer)
├─ For critical features: 5 minutes/month maximum (plan for 0 downtime)
└─ Planned maintenance: Separate window (evenings, weekends)

SEVERITY LEVELS:

P0 (Critical): System completely down, customers can't use core feature
├─ Acceptable downtime: < 5 minutes
├─ Response time: Immediate (on-call engineer paged)
├─ Escalation: Notify customer within 5 minutes
└─ Examples: Order processing down, payment failures

P1 (High): Significant impact, workaround exists but painful
├─ Acceptable downtime: < 30 minutes
├─ Response time: < 15 minutes
├─ Escalation: Notify customer within 15 minutes
└─ Examples: Reports slow, bulk import failing, dashboard down

P2 (Medium): Minor impact, customers not blocked
├─ Acceptable downtime: < 2 hours
├─ Response time: < 1 hour
├─ Escalation: Notify customer next business day
└─ Examples: Mobile app slow, UI bug in secondary feature

P3 (Low): Cosmetic issues, doesn't impact business
├─ Acceptable downtime: No time limit (fix in next release)
├─ Response time: Standard business hours
├─ Escalation: Add to backlog
└─ Examples: Typo in error message, animation jerky

RECOVERY TARGETS:

RTO (Recovery Time Objective):
├─ P0: 5 minutes (pages on-call engineer, auto-failover if possible)
├─ P1: 30 minutes (manual recovery process)
├─ P2: 2 hours (standard business day response)
└─ P3: Next release (no emergency response)

RPO (Recovery Point Objective):
├─ Critical data (orders, payments): RPO = 0 (no data loss acceptable)
│   └─ Strategy: Replicate synchronously, multiple regions
├─ Inventory data: RPO = 1 minute (lose 1 min of updates if disaster)
│   └─ Strategy: Replicate asynchronously, backup every minute
├─ Analytics/Reports: RPO = 1 hour (lose 1 hour of data)
│   └─ Strategy: Daily backups sufficient
└─ Logs: RPO = 1 week (lose up to 1 week of logs)
    └─ Strategy: Archive after 1 week, keep current week online

MONITORING & ALERTING:
├─ Monitor all P0/P1 thresholds (error rate, latency, availability)
├─ Alert triggers before hitting SLA boundary (don't wait until downtime)
├─ Auto-remediation for common issues (restart service, scale up, switch region)
└─ Manual review process for P0 incidents (root cause analysis)
```

**Output:** SLA/SLO document with uptime targets, severity levels, RTO/RPO, monitoring strategy

***

### 1.3.4 Security & Compliance Requirements

**What to do:**
- Identify applicable regulations (GDPR, CCPA, HIPAA, PCI-DSS, SOC 2, etc.)
- Identify data sensitivity levels (public, internal, confidential, restricted)
- Define authentication & authorization requirements
- Define encryption requirements (in transit, at rest)
- Define audit/compliance reporting needs

**Security & compliance framework:**

```
APPLICABLE REGULATIONS:

GDPR (if European users):
├─ User consent for data collection
├─ Right to access personal data
├─ Right to be forgotten (delete all user data)
├─ Data breach notification (within 72 hours)
├─ Data Processing Agreement with vendors
└─ Privacy impact assessment for new features

CCPA (if California users):
├─ Disclose what data is collected
├─ Right to delete personal data
├─ Right to know what data is shared
├─ Do not sell data (unless explicitly allowed)
└─ Privacy policy must be clear & conspicuous

PCI-DSS (if handling credit cards):
├─ Never store full credit card numbers
├─ Use PCI-compliant payment gateway (Stripe, Square, etc.)
├─ Encrypt payment data in transit
├─ Monthly security scans for vulnerabilities
├─ Annual penetration testing
└─ Access controls (only authorized personnel can see card data)

SOC 2 Type II (if handling customer data):
├─ Security: Prevent unauthorized access
├─ Availability: System uptime commitments
├─ Processing Integrity: Data accuracy and completeness
├─ Confidentiality: Protect sensitive data
└─ Privacy: Handle personal data per policy
└─ Requires: Audit by 3rd party, controls documentation, incident log

DATA SENSITIVITY LEVELS:

PUBLIC: Can be shared openly
├─ Blog posts, marketing materials, public documentation
└─ No encryption required, but integrity important

INTERNAL: Shared with employees only
├─ Internal documentation, metrics, non-sensitive reports
├─ Encrypt at rest if multi-tenant (isolate from other customers)
└─ Access logged but not strictly controlled

CONFIDENTIAL: Customer data, business-critical info
├─ Order data, inventory, customer contact info
├─ Encrypt at rest and in transit
├─ Access strictly controlled and logged
├─ Audit trail required (who accessed what, when)

RESTRICTED: Most sensitive (PII, payment data, health info)
├─ Credit card numbers (never store, use tokenization)
├─ Social security numbers (encrypt with field-level encryption)
├─ Password hashes (bcrypt/Argon2, never plaintext)
├─ API keys/secrets (store in secrets vault, rotate regularly)
└─ Access: Single person approval required, all access logged

AUTHENTICATION REQUIREMENTS:

User Authentication:
├─ Email/password with strong password policy (12+ chars, complexity)
├─ Password hashing: Bcrypt or Argon2 (not MD5, not SHA)
├─ Session timeout: 30 minutes of inactivity
├─ Multi-factor authentication: Required for admin accounts, optional for users
├─ Forgot password: Secure flow (email link, can't guess reset token)
└─ Failed login protection: Lockout after 5 failed attempts

API Authentication:
├─ Use JWT tokens (not session cookies) for APIs
├─ Token expiry: 1 hour for access token, 30 days for refresh token
├─ Revocation: Support immediate token revocation (logout)
├─ API Key for machine-to-machine: Rotate every 90 days
└─ Rate limiting per API key (prevent brute force, abuse)

AUTHORIZATION REQUIREMENTS:

Role-Based Access Control (RBAC):
├─ Roles: Admin, Manager, Employee, Viewer
├─ Admin: Full access to all features and user management
├─ Manager: Can manage orders, inventory, reports; cannot change system settings
├─ Employee: Can process orders, view inventory; cannot change any data
├─ Viewer: Read-only access to reports
└─ Default: Least privilege (give minimum access needed)

Permission Model:
├─ Check permissions on every action (not just UI hiding)
├─ Backend must verify (don't trust frontend claims)
├─ Log all permission checks (for audit)
└─ Examples:
    ├─ "Can only view orders from their warehouse"
    ├─ "Can only process orders assigned to them"
    └─ "Can only export reports they have access to"

ENCRYPTION REQUIREMENTS:

In Transit (TLS/HTTPS):
├─ All traffic must use HTTPS (no HTTP for authenticated sections)
├─ TLS 1.2 minimum (preferably 1.3)
├─ Certificate validation enforced
├─ HSTS header (prevent downgrade attacks)
└─ Certificate renewal: Automated, never expires

At Rest:
├─ Database: Encrypt confidential/restricted data (not all data)
├─ Backups: Encrypt with separate key from production
├─ Logs: Encrypt if contain sensitive data
├─ Configuration: Encrypt API keys, secrets (don't version control plaintext)
└─ Key Management: Use HSM or secrets vault (AWS KMS, HashiCorp Vault)

AUDIT & COMPLIANCE REPORTING:

Audit Trail:
├─ Log all sensitive actions (login, data export, permission changes)
├─ Include: User, action, resource, timestamp, IP address
├─ Cannot be deleted/modified (immutable log)
├─ Retention: 7 years for regulated data
└─ Access: Restricted, logged when accessed

Compliance Reports:
├─ GDPR: Data processing inventory, deletion requests log
├─ CCPA: Data collection practices, deletion requests log
├─ SOC 2: Security controls assessment, incident log
├─ PCI-DSS: Vulnerability scan results, penetration test results
└─ Generated: Quarterly for management review
```

**Output:** Security & compliance requirements document with regulations, data classification, controls

***

### 1.3.5 Accessibility Standards (WCAG 2.1)

**What to do:**
- Choose accessibility level (A, AA, AAA)
- Define specific requirements by category (visual, motor, cognitive, hearing)
- Identify tools/plugins for testing
- Include accessibility in acceptance criteria for every user story

**Accessibility standards:**

```
WCAG 2.1 COMPLIANCE LEVEL: AA (industry standard)

A = Minimum compliance
AA = Standard compliance (what most companies target)
AAA = Enhanced compliance (rare, very expensive)

VISUAL ACCESSIBILITY:

Color Contrast:
├─ Normal text: 4.5:1 ratio (dark text on light background)
├─ Large text (18pt+): 3:1 ratio (more lenient)
├─ UI components: 3:1 ratio minimum
├─ Examples of passing:
│   ├─ Black (#000000) on white (#FFFFFF): 21:1 ratio ✓
│   ├─ Dark blue (#003366) on white: 8.6:1 ratio ✓
│   ├─ Light gray (#999999) on white: 3.5:1 ratio ✗ (too light)
│   └─ Use WebAIM contrast checker to verify
└─ Do NOT rely on color alone to convey meaning (use icons, text, patterns)

Text Sizing:
├─ Minimum 12px font size (readable for most people)
├─ Users can zoom to 200% without horizontal scrolling
├─ Relative sizing (use ems, rems) not fixed pixels
└─ Allow font family change (not locked to custom fonts)

Visual Clarity:
├─ Focus indicators visible (when tabbing, 2px outline minimum)
├─ No auto-playing audio (must user initiate)
├─ No flashing > 3x per second (can cause seizures)
├─ Images have alt text describing content (not "image123.jpg")
├─ Icons paired with text (not icon-only buttons)
└─ Status messages announced to screen readers

MOTOR ACCESSIBILITY:

Keyboard Navigation:
├─ All functionality accessible via keyboard (not mouse-only)
├─ Tab order logical (left to right, top to bottom)
├─ Can skip navigation (skip link to main content)
├─ No keyboard traps (can exit any component)
├─ Buttons: Enter key to activate
├─ Checkboxes: Space to toggle
├─ Select dropdowns: Arrow keys to select, Enter to confirm
└─ Form submission: Tab to button, Enter to submit

Touch Targets:
├─ Minimum 44x44 pixels for touch targets (not for mouse)
├─ Spacing between buttons (no accidental clicks)
├─ No hover-only interactions (mobile users can't hover)
├─ Long-press alternatives (don't require sustained interaction)
└─ Gestures documented if required

COGNITIVE ACCESSIBILITY:

Language & Clarity:
├─ Use plain language (avoid jargon, complex sentences)
├─ Short paragraphs (3-4 sentences max)
├─ Lists for multiple items (not dense paragraphs)
├─ Headings for structure (clear hierarchy)
├─ Links descriptive ("click here" is bad, "Download annual report" is good)
└─ Readability: Flesch Reading Ease score > 60 (high school level)

Consistency:
├─ Navigation in same place (don't move it)
├─ Button styles consistent (same button = same action everywhere)
├─ Terminology consistent (don't call it "export" in one place, "download" elsewhere)
├─ Error messages consistent format
└─ Workflow consistent (don't make users relearn process)

Focus Management:
├─ Focus ring visible when tabbing
├─ Focus moves logically through page
├─ Dialog/modal: Focus moved inside, trapped until closed
├─ Skip to content: Jump past navigation
└─ After action: Focus moved to appropriate location (form submitted → success message focused)

HEARING ACCESSIBILITY:

Audio & Video:
├─ Videos have captions (accurate, including speaker identification)
├─ Videos have transcripts (for deaf-blind users)
├─ Audio content has transcript (podcasts, videos)
├─ Sound not required (visual alternative for alerts/notifications)
└─ No sound-only content (combine with visual)

Alerts & Notifications:
├─ Don't rely on sound alone
├─ Use visual indicators (color change, icon, animation)
├─ Announce to screen readers (role="alert" in HTML)
├─ Toast notifications: Announced even if off-screen
└─ Example: "Order shipped" shown as text + icon + sound + screen reader announcement

IMPLEMENTATION REQUIREMENTS:

Semantic HTML:
├─ Use proper tags (<button> not <div onclick>)
├─ Use <nav>, <main>, <section>, <article> for structure
├─ Form fields have <label> associated
├─ List items in <ul>/<ol>, not nested divs
└─ Headings in order (<h1>, then <h2>, not <h1> then <h3>)

ARIA (Accessibility Rich Internet Applications):
├─ Use ARIA only when semantic HTML insufficient
├─ role="button" for clickable divs (but prefer <button>)
├─ aria-label for icon-only buttons ("aria-label='Close dialog'")
├─ aria-hidden="true" for decorative elements (not in tab order)
├─ aria-live for dynamic content updates (screen readers announced)
└─ aria-describedby for error messages linked to inputs

Testing Requirements:
├─ Keyboard testing: Navigate entire app with Tab key only
├─ Screen reader testing: Use NVDA (Windows), VoiceOver (Mac)
├─ Color contrast: Use WebAIM contrast checker
├─ Automated testing: axe DevTools, WAVE, Lighthouse
├─ Manual testing: Real users with disabilities (if possible)
└─ Accessibility audit: 3rd party audit annually

ACCESSIBILITY CHECKLIST FOR EVERY FEATURE:
☐ All buttons/links keyboard accessible
☐ Color contrast >= 4.5:1 (normal text) or 3:1 (large text)
☐ Focus indicator visible when tabbing
☐ Images have descriptive alt text
☐ Form labels properly associated
☐ Error messages linked to fields
☐ No auto-playing audio/video
☐ Dynamic content announced to screen readers
☐ Responsive (works on mobile without horizontal scroll)
☐ Tested with actual accessibility tools (axe, NVDA, etc.)
```

**Output:** Accessibility requirements document with WCAG 2.1 AA compliance checklist

***

### 1.3.6 Browser & Device Compatibility Matrix

**What to do:**
- Define which browsers must be supported (Chrome, Firefox, Safari, Edge, IE?)
- Define minimum browser versions
- Define device types (desktop, tablet, mobile)
- Define OS versions (iOS 13+, Android 10+, etc.)
- Identify what doesn't work on older browsers (graceful degradation)

**Compatibility matrix:**

```
BROWSER SUPPORT:

Chrome:
├─ Minimum version: Latest - 2 (security + feature support)
├─ Desktop: 120+
├─ Mobile: 120+
├─ Market share: ~65% (support heavily)
└─ Notes: Progressive enhancement, most features assumed

Firefox:
├─ Minimum version: 118+
├─ Desktop only (negligible mobile share)
├─ Market share: ~10%
└─ Notes: Close to Chrome, CSS/JS compatibility high

Safari:
├─ Desktop minimum: 16+ (2022+)
├─ Mobile (iOS) minimum: 15+ (2021+)
├─ Market share: ~25%
└─ Notes: Different JS engine, some CSS quirks, test thoroughly

Edge (Chromium-based):
├─ Minimum version: 120+
├─ Desktop only
├─ Market share: ~5% (declining)
└─ Notes: Chromium-based, same as Chrome generally

NOT SUPPORTED:
├─ Internet Explorer: Any version (unsupported by Microsoft)
├─ Old Android browsers: < version 10
├─ Opera: Unless specifically request
└─ Niche browsers: Support only if > 1% market share

DEVICE SUPPORT:

Desktop:
├─ Minimum resolution: 1024x768 (old laptops)
├─ Optimal: 1440x900 (modern laptop)
├─ High DPI: Support retina displays (2x DPI)
└─ Keyboard + mouse assumed

Tablet:
├─ iPad: iOS 15+ (minimum)
├─ Android tablets: Android 10+
├─ Minimum screen: 768px width
├─ Touch interface: All interactions touch-friendly
└─ Portrait + Landscape: Both orientations supported

Mobile:
├─ iPhone: iOS 15+ (SE, 11, 12, 13, 14+)
├─ Android phones: Android 10+ (Google Pixel, Samsung Galaxy)
├─ Minimum screen: 320px width (old iPhones)
├─ Max screen: 430px width (responsive design)
├─ Touch-only: No hover states (must have tap alternatives)
├─ Network: Support slow 3G (500ms latency)
└─ Battery: Avoid excessive JS (battery drain)

FEATURE DEGRADATION:

Feature: WebGL 3D graphics
├─ Desktop Chrome/Firefox/Safari: Supported, use WebGL
├─ Mobile: Performance too poor, use 2D canvas fallback
└─ Old browsers: Graceful degradation to images

Feature: Real-time WebSocket sync
├─ Modern browsers: Use WebSocket
├─ Old IE11: Fallback to long-polling
├─ Mobile on slow network: Fallback to polling (less battery drain)

Feature: IndexedDB offline storage
├─ Support: All modern browsers
├─ Fallback: LocalStorage (but limited size)
└─ Old IE: Server always required (no offline)

Feature: Service Workers (offline + caching)
├─ Support: All modern browsers
├─ Fallback: Works online only, no offline mode
└─ Benefits: Worth adding, graceful if not supported

TEST MATRIX:

Priority 1 (Test on all):
├─ Latest Chrome (desktop)
├─ Latest Safari (desktop)
├─ Latest iOS Safari
├─ Latest Android Chrome
└─ Frequency: Every major release

Priority 2 (Test weekly):
├─ Firefox latest
├─ Edge latest
└─ Older iOS (previous version)

Priority 3 (Manual spot checks):
├─ Tablet landscape mode
├─ iPhone SE (small screen)
├─ iPhone 14 Pro (large screen)
├─ 5-year-old MacBook (slow)
└─ Android mid-range phone

TOOLS FOR TESTING:
├─ BrowserStack: Live testing on real devices
├─ Sauce Labs: Automated cross-browser testing
├─ Local testing: Chrome DevTools device emulation
├─ Lighthouse: Performance on different networks
└─ Real devices: At least test on 2-3 real phones

GRACEFUL DEGRADATION RULES:
- Core functionality works everywhere
- Progressive enhancement for modern features
- Test in oldest supported browser (ensure not broken)
- Mobile users primary focus (mobile-first design)
- Old browsers: Function works, just slower/less polished
```

**Output:** Browser/device compatibility matrix with support levels, fallbacks, test plan

***

### 1.3.7 Internationalization (i18n) Requirements

**What to do:**
- Define which languages must be supported (English, Spanish, German, etc.)
- Define which locales (en-US vs en-GB, es-ES vs es-MX, etc.)
- Define cultural adaptations (currency, date format, right-to-left languages)
- Define translation strategy (in-house vs agency vs community)

**i18n requirements:**

```
LANGUAGE SUPPORT:

MVP Languages (Launch):
├─ English (en-US) - primary language
└─ Spanish (es-MX) - 2nd largest market

Phase 1.1:
├─ German (de-DE)
├─ French (fr-FR)
├─ Portuguese (pt-BR)
└─ Japanese (ja-JP)

Phase 1.2+:
├─ Mandarin Chinese (zh-CN)
├─ Russian (ru-RU)
└─ Any user-requested language

LOCALE REQUIREMENTS:

English (en-US):
├─ Date format: MM/DD/YYYY
├─ Currency: USD ($)
├─ Decimal separator: . (1,234.56)
├─ Thousand separator: , (1,234.56)
└─ Timezone: Eastern (ET) to Pacific (PT)

Spanish (es-MX):
├─ Date format: DD/MM/YYYY
├─ Currency: MXN ($)
├─ Decimal separator: . (1,234.56) or , in Spain
├─ Thousand separator: , or space
└─ Timezone: Mexico City (CST)

German (de-DE):
├─ Date format: DD.MM.YYYY
├─ Currency: EUR (€)
├─ Decimal separator: , (1.234,56)
├─ Thousand separator: . (1.234,56)
└─ Timezone: Central European Time (CET)

Japanese (ja-JP):
├─ Date format: YYYY年MM月DD日 (2024年01月15日)
├─ Currency: JPY (¥)
├─ Number format: 1,234,567 (same as en-US)
├─ Text direction: Left-to-right (not RTL)
└─ Timezone: Japan Standard Time (JST)

CULTURAL ADAPTATIONS:

Right-to-Left (RTL) Languages:
├─ If adding Arabic (ar-SA), Hebrew (he-IL): Entire UI flips
├─ Navigation: Right side instead of left
├─ Text direction: Right-to-left reading order
├─ Images: May need localization (culturally appropriate)
├─ Estimated effort: 20-30% of localization time
└─ Tools: Framer Motion supports RTL via CSS logical properties

Date/Time Handling:
├─ Always store as UTC in database
├─ Convert to user timezone on display
├─ Allow user to override timezone preference
├─ Daylight saving time: Auto-handled by platform library
└─ Example: Order placed "2024-01-15 14:30 EST" shows as "2024-01-15 19:30 GMT" in Europe

Number Formatting:
├─ Store all numbers as numeric in database (not strings)
├─ Format on display based on locale
├─ Examples:
│   ├─ 1234.56 in en-US displays as "1,234.56"
│   ├─ 1234.56 in de-DE displays as "1.234,56"
│   └─ 1234.56 in zh-CN displays as "1234.56" (usually comma optional)
└─ Use library (intl.NumberFormat) to avoid manual formatting

Currency Handling:
├─ Store all prices in base currency (USD) + exchange rate
├─ Allow users to view in their local currency
├─ Purchase: Always in user's currency choice
├─ Exchange rates: Update daily (via external API)
└─ VAT/Tax: Adjust based on user location (EU VAT varies by country)

Images & Icons:
├─ Some icons culturally specific (hand gestures)
├─ Review images with native speakers
├─ Avoid text in images (can't translate)
└─ Flag emoji: Only if necessary (can be offensive, use text instead)

TRANSLATION STRATEGY:

Text Extraction:
├─ All user-facing text in code extracted to translation files
├─ Don't hardcode strings (use i18n keys)
├─ Example: Instead of <button>Submit</button>
│   Use: <button>{t('button.submit')}</button>
├─ Helper texts, error messages, tooltips all included
└─ Tool: i18next, react-i18next, or equivalent

Translation Process:
├─ MVP: Professional translation for Spanish (80% of launch users)
├─ Quality: Native speaker review (not Google Translate)
├─ Storage: Translation files versioned in Git
├─ Update cycle: New translations within 2 weeks of feature release
└─ Cost: ~$0.10-0.20 per word (so 10,000 words = $1,000-2,000 per language)

Plural Handling:
├─ Different languages have different plural rules
├─ Example: English: 0, 1, many (0 orders, 1 order, 2 orders)
│   Russian: 5+ plural rules
│   Japanese: Same singular for all (no plural)
├─ Tool: Use i18n library that handles plurals automatically
└─ Translation key structure:
    ```
    "orders": {
      "singular": "You have {{count}} order",
      "plural": "You have {{count}} orders"
    }
    ```

Missing Translations:
├─ Fallback to English if translation missing
├─ Alert developer (log missing translation keys)
├─ Don't show raw translation keys to users
└─ Translation coverage: Aim for 100%, accept 95%+ for launch

TECHNICAL IMPLEMENTATION:

URL Structure:
├─ Option 1: Subdomain (en.site.com, es.site.com) - SEO better
├─ Option 2: Path-based (/en/, /es/) - simpler
├─ Option 3: Parameter (?lang=en) - worst for SEO
└─ Selected: Path-based (/en/, /es/) for simplicity

Detection:
├─ Detect language from: Accept-Language header, URL, user preference
├─ Priority: User preference > URL > Accept-Language > default
├─ Remember user preference (localStorage, user profile)
└─ Switching languages: User control (dropdown, settings)

Meta Tags:
├─ Set lang attribute on HTML (<html lang="en">)
├─ Set hreflang for alternate languages (for SEO)
│   <link rel="alternate" hreflang="es" href="/es/page" />
└─ Help search engines understand multi-language site

PERFORMANCE:
├─ Load only current language file (don't bundle all languages)
├─ Translation keys lazy-loaded on demand
├─ Cache translations after loading
└─ Language switching: < 1 second (pre-load other languages)

QA FOR TRANSLATIONS:
☐ All user-facing text translated
☐ Text fits in UI (some languages longer: German, Turkish)
☐ RTL languages (if applicable): Layout flips correctly
☐ Date/time formatting correct for locale
☐ Currency displays correctly
☐ Images culturally appropriate
☐ Placeholder text in forms translated
☐ Error messages translated
☐ No hardcoded English strings remaining
☐ Search/filtering works in local language
☐ Performance: Language switching doesn't lag
```

**Output:** i18n requirements document with language support, locale specifics, translation strategy

***

## 1.4 TECHNICAL CONSTRAINTS

### 1.4.1 Budget Limitations

**What to do:**
- Document total budget allocated
- Break down by category (people, infrastructure, tools, external services, contingency)
- Understand if budget is flexible or hard cap
- Calculate cost per feature (helps prioritization)
- Identify cost drivers (what's most expensive)

**Budget breakdown example:**

```
TOTAL PROJECT BUDGET: $250,000

LABOR COSTS (60% of budget = $150,000):
├─ Backend Engineer: $40/hour × 1000 hours = $40,000
├─ Frontend Engineer: $40/hour × 1000 hours = $40,000
├─ DevOps/Infrastructure: $50/hour × 400 hours = $20,000
├─ QA/Testing: $30/hour × 500 hours = $15,000
├─ Product Manager: $45/hour × 300 hours = $13,500
├─ Designer/UX: $45/hour × 200 hours = $9,000
└─ Project Manager: $40/hour × 200 hours = $8,000

INFRASTRUCTURE & SERVICES (20% of budget = $50,000):
├─ Cloud infrastructure (AWS/GCP): $20,000
├─ Database hosting: $8,000
├─ CDN & caching: $5,000
├─ Payment gateway (Stripe fees): 2.9% of transaction volume = $8,000
├─ Email service (Sendgrid): $2,000
├─ Monitoring & logging: $3,000
├─ Security tools: $2,000
└─ SMS/notifications: $2,000

TOOLS & SOFTWARE (10% of budget = $25,000):
├─ Development tools (IDE licenses, if needed): $1,000
├─ Testing tools (BrowserStack, LoadImpact): $5,000
├─ Design tools (Figma, InVision): $2,000
├─ Project management (Jira, Asana): $2,000
├─ Code repository (GitHub, GitLab): $1,000
├─ CI/CD tools: $3,000
├─ Security scanning (Snyk, SonarQube): $3,000
├─ Analytics tools: $2,000
├─ Backup & disaster recovery: $2,000
├─ Communication tools: $1,000
└─ Training & courses: $3,000

THIRD-PARTY SERVICES (5% of budget = $12,500):
├─ Legal/compliance review: $3,000
├─ Security audit (penetration testing): $4,000
├─ Translation services: $3,000
├─ API services (maps, weather, etc.): $1,500
└─ Stock images/assets: $1,000

CONTINGENCY (5% of budget = $12,500):
└─ Buffer for unexpected costs, scope creep, delays

COST PER FEATURE:

Feature: Real-time Inventory Sync
├─ Backend: 200 hours = $8,000
├─ Frontend: 150 hours = $6,000
├─ Infrastructure: Additional Redis instance = $1,000/year
├─ Testing: 100 hours = $3,000
├─ Total: $18,000 (7% of budget)
└─ Business value: Saves 10 hours/day in labor = ROI < 2 years

Feature: Mobile App
├─ Design: 150 hours = $6,750
├─ iOS Development: 400 hours = $16,000
├─ Android Development: 400 hours = $16,000
├─ Testing: 200 hours = $6,000
├─ Store fees & distribution: $100/year
├─ Total: $44,850 (18% of budget)
└─ Business value: New user segment, not required for MVP

BUDGET DECISION FRAMEWORK:

If cost overrun threatens deadline:
├─ Cut low-ROI features (mobile app, analytics)
├─ Reduce scope (support fewer countries, simpler reporting)
├─ Extend timeline (trades cost for schedule)
├─ Increase budget (if possible)
└─ Accept reduced quality (dangerous, usually leads to technical debt)

If business impact high but budget insufficient:
├─ Seek additional funding/budget
├─ Phase features (MVP without advanced analytics, add later)
├─ Partner with vendors (use managed services vs build)
└─ Defer non-critical features (launch with 80%, add 20% in Phase 2)
```

**Output:** Budget breakdown with labor, infrastructure, tools, contingency, cost per feature

***

### 1.4.2 Timeline Constraints

**What to do:**
- Document hard deadline (if any)
- Understand flexibility (can you slip 2 weeks, or is date immovable)
- Identify external dependencies that affect timeline
- Document what must be ready by which milestone
- Include buffer (contingency time)

**Timeline analysis:**

```
PROJECT TIMELINE:

Hard Deadline: March 31, 2024 (business commitment, cannot slip)
├─ Reason: Major conference, partnership announcement, fiscal year cutoff
├─ Flexibility: Zero (this date is hard constraint)
└─ Consequence of missing: Lose customer, partnership broken, revenue target missed

Working Backward from Deadline:
├─ Deployment/Launch (Week 16-17): March 24-31, 2024
├─ Phase 11 (Deployment)
├─ Phase 9-10 (Testing + Documentation): 2-3 weeks before
├─ Phase 7-8 (Development): 4-8 weeks before
├─ Phase 6 (Infrastructure): 1 week before development
├─ Phase 2-5 (Design + Architecture): First 4 weeks
├─ Phase 0-1 (Requirements): Week 1-2
└─ Total: 16-17 weeks from now (start by Dec 15, 2023)

MILESTONE DATES:

Week 1 (Dec 15): Phase 0-1 complete (requirements approved)
├─ Stakeholder sign-off: REQUIRED
├─ Team fully allocated: REQUIRED
└─ No slip acceptable (cascades to all downstream)

Week 4 (Jan 5): Phase 2-5 complete (architecture, design, API, infrastructure setup)
├─ Architecture reviewed: REQUIRED
├─ Infrastructure provisioned: REQUIRED
├─ Can slip 1 week max (still OK if development parallelized)
└─ Slip > 1 week: Start losing development time

Week 12 (Feb 23): Phase 7-8 complete (development done, testing begins)
├─ Backend feature-complete: REQUIRED
├─ Frontend feature-complete: REQUIRED
├─ Can slip 3 days max (testing needs 2+ weeks minimum)
└─ Slip > 1 week: Reduces testing time, increases risk

Week 14 (Mar 8): Phase 9 complete (testing, bugs fixed)
├─ All critical bugs fixed: REQUIRED
├─ Performance targets met: REQUIRED
├─ Security audit passed: REQUIRED
├─ Cannot slip (deployment only 3 weeks remaining)

Week 16 (Mar 22): Phase 10 complete (documentation)
├─ Runbooks written: REQUIRED
├─ User guides written: REQUIRED
└─ Overlaps with Phase 11

Week 17 (Mar 31): Deployed live
├─ Database migrated: DONE
├─ Application live: DONE
├─ Post-launch monitoring: STARTED
└─ Success: Happy customers

EXTERNAL DEPENDENCIES & CRITICAL PATH:

Dependency 1: Third-Party API (Payment Gateway Integration)
├─ Required by: Week 6 (Feb 2)
├─ Partner availability: Dec 15 onward
├─ Risk: Partner slow, API changes, approval delays
├─ Mitigation: Contact partner immediately, get sandbox access early, build mock first
└─ Blocker if: They can't provide API keys by Jan 10

Dependency 2: Cloud Vendor Account (AWS, GCP)
├─ Required by: Week 5 (Jan 26) for infrastructure setup
├─ Lead time: 1-2 weeks (procurement, approvals)
├─ Risk: Procurement delays, budget approval
├─ Mitigation: Start procurement immediately, have fallback region
└─ Blocker if: Account not created by Jan 12

Dependency 3: Design Review/Approval
├─ Required by: Week 3 (Dec 29)
├─ Stakeholder review: Week 2-3
├─ Risk: Stakeholder unavailable (holidays), lots of revision requests
├─ Mitigation: Get approval in writing, schedule review early in week 2
└─ Blocker if: Not approved by Dec 27

Dependency 4: Security Audit (Third Party)
├─ Required by: Week 14 (Mar 8) - pre-launch security review
├─ Lead time: 2-4 weeks
├─ Risk: Auditor booked (schedule early), findings require fixes
├─ Mitigation: Schedule audit by Week 11, start early Feb, allow 1 week fix time
└─ Blocker if: Can't schedule until March (too late for fixes)

TIMELINE ASSUMPTIONS:

- Team fully allocated starting Week 1 (no part-time, no vacations)
- Requirements stable after Week 2 (changes = timeline slip)
- Design approved Week 3 (no design cycle 2)
- Technology decisions final Week 2 (no mid-project tech switches)
- No key person departures (if happens, timeline extends 4+ weeks)
- External dependencies deliver on time (mitigations in place)
- Testing parallelized with development (not sequential)
- Deployment doesn't encounter production surprises (infrastructure well-planned)

WHAT HAPPENS IF TIMELINE SLIPS:

Slip 1 week (Feb 2 → Feb 9):
├─ Testing duration reduced from 3 weeks to 2 weeks
├─ Risk: Catch fewer bugs, but still acceptable if focused on critical paths
└─ Mitigation: Reduce scope, cut low-priority features

Slip 2 weeks (Feb 2 → Feb 16):
├─ Testing duration reduced to 1 week
├─ Risk: High chance of bugs in production
├─ Mitigation: Cut features, extend Phase 8 (ship with less functionality)
└─ Decision: Defer features to Phase 1.1 (1-2 weeks after launch)

Slip 3+ weeks (Feb 2 → Feb 23+):
├─ Testing nearly eliminated
├─ Risk: Very high (major bugs guaranteed)
├─ Mitigation: ONLY option is to cut scope significantly
│   ├─ Cut mobile responsiveness (desktop only)
│   ├─ Cut advanced features (keep MVP only)
│   ├─ Cut internationalization (English only)
│   └─ Cut analytics (add later)
└─ Likely outcome: Launch with reduced scope, Phase 1.1 adds features

HARD DEADLINE MANAGEMENT:

Never says "we'll make it work":
├─ Slip happens, you're now in crisis mode
├─ Quality suffers (technical debt)
├─ Team burns out
├─ Long-term velocity decreases

Instead: Plan scope for timeline
├─ Identify MVP (absolute minimum)
├─ Everything else deferred to Phase 1.1
├─ Communicate clearly: "Launch March 31 with 80% features, Phase 1.1 in April with remaining 20%"
└─ Better to ship less on time than ship everything late
```

**Output:** Timeline document with milestones, critical path, external dependencies, contingency

***

### 1.4.3 Team Skill Assessment & Hiring Needs

**What to do:**
- For each required role, assess current team capability
- Identify skill gaps (what expertise is missing)
- Create hiring plan (who to hire, by when)
- Understand onboarding time (new people need ramp-up)
- Consider contractors vs employees (trade-offs)

**Team assessment:**

```
CURRENT TEAM ASSESSMENT:

Backend Engineering:
├─ Current: 1 senior backend engineer (10 years experience)
├─ Capability: Can design architecture, lead backend development
├─ Experience: Built 3 large-scale systems, database performance expert
├─ Available: 100% allocation for this project
├─ Gap: No. 1 senior is sufficient to lead, but would benefit from 1 junior to pair
└─ Action: Hire 1 junior backend engineer (can ramp in 4-6 weeks)

Frontend Engineering:
├─ Current: 0 (need to hire)
├─ Experience needed: React, TypeScript, responsive design, accessibility
├─ Why empty: Was using contractors, quality was poor, decision to build in-house
├─ Hiring plan:
│   ├─ Senior frontend engineer: Hire by Week 1 (interview immediately)
│   │   ├─ Role: Technical lead for frontend
│   │   ├─ Ramp-up: 2-3 weeks (familiar with stack)
│   │   └─ Salary: $140k-160k
│   └─ Mid-level frontend engineer: Hire by Week 2
│       ├─ Role: Implement features, QA components
│       ├─ Ramp-up: 4-5 weeks (less familiar with internal tools)
│       └─ Salary: $100k-120k
├─ Timeline: Start recruiting Week 0, offer extended Week 1-2, start Week 3-4
└─ Risk: Can't find senior engineer (plan contractor option)

DevOps/Infrastructure:
├─ Current: 0.5 FTE (part-time from another project)
├─ Experience: Has set up AWS before, understands Docker
├─ Capability: Can get infrastructure running, but not at scale optimization level
├─ Gap: Needs full-time person for this complexity (Kubernetes, auto-scaling, monitoring)
├─ Hiring plan:
│   └─ Senior DevOps engineer: Hire by Week 0
│       ├─ Role: Design infrastructure, CI/CD, monitoring
│       ├─ Ramp-up: 1-2 weeks (infrastructure already somewhat familiar)
│       └─ Salary: $150k-170k
├─ Contractor option: If can't hire, contract senior DevOps for setup phase
└─ Critical: Must start building infrastructure by Week 5

QA / Testing:
├─ Current: 0 (manual testing handled by developers)
├─ Need: Dedicated QA for test plan, automation, regression testing
├─ Hiring plan:
│   └─ QA Engineer: Hire by Week 2
│       ├─ Role: Test planning, manual testing, automation framework
│       ├─ Ramp-up: 2-3 weeks (learn feature requirements)
│       └─ Salary: $80k-100k
├─ Contractor option: Accept less automation in Phase 1, hire after launch
└─ Decision: Hire full-time (better for ongoing support)

Product Manager:
├─ Current: 1 product manager (working on 2 other projects)
├─ Time allocation: 30% available for this project (not enough)
├─ Action: Add 1 full-time PM or elevate to 100% allocation
├─ Hiring: Hire full-time PM (or contractor if no hires planned)
└─ Salary: $120k-140k

Designer / UX:
├─ Current: 1 designer (freelancer, part-time)
├─ Capability: Good design skills, but not accessibility expert
├─ Allocation: 50% for this project (enough for Phase 1-2)
├─ Gap: Needs accessibility training (or contractor audit)
└─ Action: Keep freelancer, add accessibility review by consultant

Project Manager:
├─ Current: 0 (engineering lead managing informally)
├─ Action: No dedicated PM (engineering lead + product manager sufficient for <10 person team)

HIRING PLAN TIMELINE:

Week 0 (Now):
├─ Post 3 open roles (Senior Frontend, Senior DevOps, Mid Frontend)
├─ Interview pipeline start
└─ Offer target: Week 1-2

Week 1:
├─ Senior Frontend candidate interviews (3-4 candidates)
├─ Senior DevOps candidate interviews
├─ Extended offer to top candidates
└─ Target start: Week 3-4

Week 2:
├─ Mid-level Frontend interviews
├─ Offer extended
└─ Target start: Week 4-5

Week 3:
├─ Senior Frontend starts (ramp-up begins)
├─ Prepare onboarding (dev environment, access, documentation)
└─ Pair with senior backend engineer

Week 4:
├─ Senior DevOps starts (infrastructure design)
├─ Mid Frontend starts (feature implementation)
└─ Training on architecture, tools, deployment

TOTAL TEAM BY WEEK 4:

```
Backend:
├─ 1 Senior (10 yrs) - Architecture, complex features
├─ 1 Junior (2 yrs) - Assist, testing, docs
└─ 2 total

Frontend:
├─ 1 Senior (8 yrs) - Architecture, complex features, component design
├─ 1 Mid (5 yrs) - Feature implementation, testing
└─ 2 total

DevOps:
├─ 1 Senior (12 yrs) - Infrastructure, CI/CD, monitoring
└─ 1 total

QA:
├─ 1 QA Engineer (4 yrs) - Test planning, automation, reports
└─ 1 total

Product:
├─ 1 PM (6 yrs) - Requirements, roadmap, stakeholder management
└─ 1 total

Design:
├─ 1 Freelancer (part-time, design)
└─ 0.5 total

TOTAL: 7.5 people

ONBOARDING PLAN FOR EACH NEW HIRE:

Day 1:
├─ Welcome, company intro, tools setup
├─ Git access, code repo cloned, IDE configured
├─ Slack, email, calendar setup
└─ Goal: Can read code

Day 2-3:
├─ Architecture walkthrough (system design, why choices made)
├─ Codebase walkthrough (how code organized, conventions)
├─ Development environment setup (can compile, run tests locally)
└─ Goal: Can run project locally

Week 1:
├─ Pair programming with team (observe, then co-implement)
├─ Small task assignment (fix a test, add small feature)
├─ Code review participation (review others' code, learn standards)
└─ Goal: Understands how things work

Week 2-3:
├─ Assigned to small story (4-8 story points)
├─ Regular 1-on-1s to check understanding, remove blockers
├─ Code review feedback
└─ Goal: Can implement independently

Week 4-6:
├─ Ramped to normal productivity
├─ Can mentor other junior people
├─ Feedback on documentation (what was confusing)
└─ Goal: Fully productive member

CONTRACTOR vs EMPLOYEE DECISION:

When to use Contractors:
├─ Specialized skill for limited time (security audit, load testing, translation)
├─ Spike/research (evaluate tech, proof of concept)
├─ Peak work period (extra hands for 2-3 weeks)
├─ No long-term role (no ongoing need)

When to hire Employees:
├─ Core team need (will last > 6 months)
├─ Knowledge retention (what they learn stays in company)
├─ Long-term mentorship (growth path for junior people)
├─ Critical path (can't afford onboarding delays)

Cost Comparison:
├─ Contractor rate: $80-150/hour (8x to 15x more than employee cost)
├─ Employee salary: $80k-160k annually = $40-77/hour fully loaded
├─ Break-even: Contractor cheaper if < 3 months, employee cheaper if > 6 months

Recommendation for this project:
├─ Hire: Frontend (core team), DevOps (infrastructure owner), QA (ongoing)
├─ Contract: Accessibility audit (4 weeks), UX review (8 weeks), Performance optimization (4 weeks)
└─ Current freelancer: Keep for design, evaluate for hiring after Phase 1

TEAM COMMUNICATION & COORDINATION:

With 7+ people:
├─ Daily standup: 15 minutes (blockers, progress)
├─ Weekly sync: 1 hour (planning, blockers, decisions)
├─ Architecture review: Bi-weekly (major decisions)
├─ Code review: Continuous (all PRs reviewed before merge)
├─ 1-on-1s: Weekly with each report (manager or senior lead)
└─ Slack channels: #backend, #frontend, #devops, #questions, #random

Tools:
├─ Code: GitHub (private repo, protected branches)
├─ Project management: Jira (stories, sprints, burndown)
├─ Communication: Slack (async discussion), Zoom (meetings)
├─ Documentation: Confluence (technical docs, decisions)
├─ Design: Figma (design collaboration, prototyping)
└─ Monitoring: Slack alerts (infrastructure alerts via Slack)
```

**Output:** Team assessment with hiring plan, onboarding timeline, contractor needs

---

### 1.4.4 Legacy System Integration Needs

**What to do:**
- If integrating with existing systems, document requirements
- Understand API contracts (what data flows between systems)
- Identify data migration needs (existing data → new system)
- Plan integration timeline (when does integration need to be ready)

**Integration analysis:**

```
EXISTING SYSTEMS TO INTEGRATE:

System 1: Legacy Accounting Software (QuickBooks)
├─ Current use: All invoicing, financial reporting
├─ Integration need: Sync orders from new system to QuickBooks
├─ API: REST API available (documented)
├─ Data flow: Order created → Sync to QB as invoice
├─ Frequency: Real-time (within 5 minutes)
├─ Data mapped:
│   ├─ Customer name → Customer in QB
│   ├─ Order items → Line items
│   ├─ Order total → Amount
│   └─ Payment status → Invoice status
├─ Challenges:
│   ├─ QB API rate limit: 200 requests/minute (need to batch)
│   ├─ Data inconsistency if sync fails (need retry + reconciliation)
│   └─ QB doesn't support multi-currency (we'll support it, QB gets USD converted)
├─ Error handling: If QB unavailable, queue order, retry every 5 minutes
├─ Timeline: Integration ready by Week 10 (can use manual export initially)
└─ Contractor: May need QB integration specialist (cost: $5-10k)

System 2: Email System (Microsoft Office 365)
├─ Current use: Customer emails sent from Office 365
├─ Integration need: Send transactional emails from new system
├─ API: Microsoft Graph API (modern OAuth2)
├─ Data flow: Event triggered (order placed) → Send email via Office 365
├─ Frequency: Every transaction (potentially 1000s/day)
├─ Challenge: Office 365 API rate limits (think we'll exceed)
├─ Alternative: Use SendGrid (handles email scale, integrates with O365 for archival)
├─ Decision: Use SendGrid for sending, O365 archive for compliance
└─ Timeline: Integration ready by Week 8

System 3: Existing Customer Database (SQL Server)
├─ Current use: Stores customer data, 50k customers
├─ Data needed: Name, email, address, historical purchases
├─ Problem: Customer data in new system must sync with old database
├─ Challenge: Data quality issues (duplicate customers, missing emails)
├─ Migration strategy:
│   ├─ Week 4: Audit existing customer data (identify duplicates, quality issues)
│   ├─ Week 5: Deduplicate, clean, create migration script
│   ├─ Week 7: Dry run (migrate sample of 1000 customers, verify accuracy)
│   ├─ Week 9: Full migration (migrate all 50k customers)
│   └─ Week 10: Sync new system ↔ old system (keep old system read-only)
├─ Risk: Data loss if migration fails (plan rollback)
└─ Contingency: 2-week delay acceptable (can launch without customer import, manually add key customers)

System 4: Shipping Integration (FedEx, UPS APIs)
├─ Current use: Manual label printing (staff uses web portal)
├─ Integration need: Auto-generate shipping labels from new system
├─ APIs: FedEx Web Services API (SOAP), UPS OnRate & OnTrack (REST)
├─ Challenge: Both APIs complex, need credentials/setup
├─ Alternative: Use Shippo (unified shipping API, supports FedEx, UPS, USPS)
├─ Decision: Use Shippo (faster integration, costs ~2.9% + $0.01/label)
├─ Timeline: Integration ready by Week 9
└─ Testing: Ship 100 test packages to verify labels print correctly

INTEGRATION PRIORITIES:

Must have for launch (Week 15):
├─ Email sending (SendGrid)
├─ Customer data import from old database
└─ Shipping label generation (Shippo)

Nice to have for launch (Week 16-17):
├─ QB sync (manual workaround acceptable initially)
└─ Advanced inventory sync with legacy system

DATA MIGRATION PLAN:

Existing customer database:
├─ 50,000 customers
├─ Schema: CustomerID, Name, Email, Address, Phone, CreatedDate
├─ Migration steps:
│   1. Extract data from SQL Server
│   2. Validate/clean (remove duplicates, fix emails)
│   3. Transform (map old schema to new schema)
│   4. Load into new database
│   5. Verify (random sample audit)
│   6. Sync ongoing (new system = source of truth, old system read-only)
├─ Tools: Custom Python script (ETL process)
├─ Dry run: Week 7 (100 customers, verify 100% accuracy)
├─ Production migration: Week 9 (Sunday night, 2-hour maintenance window)
├─ Risk: 1% data loss acceptable (e.g., 500 bad records)
└─ Contingency: Rollback script ready (can revert if migration fails)

API INTEGRATION CONTRACTS:

SendGrid Email API:
├─ Endpoint: POST /v3/mail/send
├─ Request: From, To, Subject, Body, Track (open/click)
├─ Response: Message ID (for tracking)
├─ Rate limit: 300 requests/second
├─ Retry: 3 retries on failure, exponential backoff
├─ Testing: Sandbox environment available

Shippo Shipping API:
├─ Endpoints: 
│   ├─ POST /shipments/ (create shipment)
│   ├─ POST /transactions/ (generate label)
│   └─ GET /transactions/{ID}/ (track label)
├─ Request: From address, To address, parcel size/weight, carrier
├─ Response: Label URL, tracking number, cost
├─ Rate limit: 60 requests/minute (batch if needed)
└─ Testing: Test mode available (no real labels created)

QuickBooks API:
├─ Endpoints:
│   ├─ POST /v2/companyID/invoice (create invoice)
│   ├─ PUT /v2/companyID/invoice/{ID} (update)
│   └─ GET /v2/companyID/query (query invoices)
├─ Auth: OAuth 2.0 (expires after 60 days, refresh needed)
├─ Sandbox: Yes, available for testing
├─ Rate limit: 200 requests/minute
├─ Challenge: Auth token refresh (build into system)
└─ Risk: If QB unavailable, queue orders locally (retry when available)

RISK ASSESSMENT:

High risk:
├─ Customer data migration (50k records, one-time, must be perfect)
├─ QB API rate limits (may need batching, delays acceptable)
└─ FedEx/UPS API complexity (Shippo reduces risk)

Medium risk:
├─ SendGrid integration (well-documented, low risk if tested)
├─ Email deliverability (SendGrid handles, reputation needed)

Mitigation:
├─ Start integrations early (Week 6, not Week 14)
├─ Spike on risky APIs (proof of concept before full build)
├─ Contract expert for QB/FedEx if internal expertise lacking
├─ Customer data: Dry run 2 weeks before production migration
└─ Alerts: Monitor integration failures, manual remediation process
```

**Output:** Integration requirements document with API contracts, data migration plan, risk assessment

---

### 1.4.5 Third-Party Service Dependencies

**What to do:**
- List all external services system depends on (payment gateway, analytics, logging, etc.)
- For each, understand: cost, uptime SLA, API limits, lock-in risk
- Plan failover strategy (what if Stripe goes down)
- Identify single points of failure (if X service down, are we completely blocked)

**Third-party dependencies:**

```
CRITICAL SERVICES (System cannot operate without):

1. Payment Gateway (Stripe)
├─ Purpose: Process customer payments
├─ Cost: 2.9% + $0.30 per transaction (~5% of revenue if 100k orders)
├─ SLA: 99.99% uptime
├─ Rate limits: No transaction limit (unlimited)
├─ API: REST, well-documented, SDKs available
├─ Risk: If Stripe down, can't process payments (blocking)
├─ Failover: Have backup (Square, PayPal), but requires setup time
│   ├─ Cost to implement backup: $10k-15k (3 weeks dev)
│   └─ Decision: Not required for MVP (Stripe very reliable)
├─ Lock-in risk: Medium (can switch, but customer data portability issues)
└─ Contingency: If payment fails, queue order, retry when Stripe available

2. Email Service (SendGrid)
├─ Purpose: Send transactional emails (order confirmation, shipping notice)
├─ Cost: $10-100/month depending on volume (1k-100k emails/month)
├─ SLA: 99.95% uptime
├─ Rate limits: 300 requests/second (plenty for our volume)
├─ API: REST, excellent documentation
├─ Risk: If SendGrid down, emails don't send (non-blocking, can queue)
├─ Failover: Use O365 directly (lower rate limit, but functional)
├─ Lock-in risk: Low (easy to switch, emails portable)
└─ Contingency: Queue emails, retry on SendGrid recovery

3. Cloud Infrastructure (AWS)
├─ Purpose: Host all application servers, databases, storage
├─ Cost: Estimated $20k/year for expected load
├─ SLA: 99.99% for EC2, 99.99% for RDS, 99.9% for S3
├─ Regions: We'll use US-East (Virginia) as primary
├─ Risk: If AWS down, entire system down (catastrophic)
├─ Failover: Requires multi-region setup (expensive, not for MVP)
│   ├─ Cost to implement: $50k additional infrastructure + $15k setup
│   └─ Decision: Not required for MVP (AWS very reliable)
├─ Single region acceptable: 99.9% SLA for region (acceptable for SaaS)
└─ Contingency: Data replicated to backup region (for disaster recovery)

4. DNS Provider (Route 53)
├─ Purpose: Domain name resolution (site.com → IP address)
├─ Cost: Negligible ($0.50/month)
├─ SLA: 100% (mission critical from AWS)
├─ Risk: If DNS down, no one can access site (catastrophic)
├─ Failover: Secondary DNS provider (Cloudflare) configured (no cost if just backup)
└─ Contingency: Failover DNS in < 5 minutes (manual process if needed)

5. Monitoring & Alerting (DataDog)
├─ Purpose: Monitor application health, alert on issues
├─ Cost: $20-50/month (depends on hosts, logs, custom metrics)
├─ SLA: 99.9%
├─ Risk: If DataDog down, we don't know there's a problem (bad for response)
├─ Fallback: Metrics still flowing to CloudWatch (AWS native), manual checks
└─ Contingency: DataDog is observability luxury, not critical for function

IMPORTANT SERVICES (Enhance experience, not critical):

6. Analytics (Google Analytics)
├─ Purpose: Understand user behavior, traffic patterns
├─ Cost: Free (GA4), paid for enterprise features
├─ Risk: If GA down, we can't see analytics (non-blocking, internal only)
└─ Contingency: Fallback to server-side event logging

7. Logging (ELK Stack - Elasticsearch)
├─ Purpose: Store application logs for debugging
├─ Cost: Managed Elasticsearch ~$500/month
├─ Risk: If logging down, can't debug issues (operational pain)
├─ Fallback: CloudWatch (AWS native logging)
└─ Contingency: Use CloudWatch, upgrade to Elasticsearch later

8. Content Delivery Network - CDN (CloudFront)
├─ Purpose: Cache static assets globally, speed up content delivery
├─ Cost: $0.085/GB (depends on usage)
├─ Risk: If CDN down, site loads from origin (slower but still works)
├─ Fallback: Serve directly from S3
└─ Contingency: Non-critical for function

DEPENDENCY MANAGEMENT:

Adding new service requires:
1. Evaluate necessity (is this actually needed, or nice-to-have)
2. Cost analysis (what's true cost including setup, monitoring, support)
3. SLA/reliability (how much uptime do we need, can we tolerate failures)
4. Failover strategy (what happens if service goes down)
5. Lock-in risk (can we switch vendors later, what's the cost)
6. Integration timeline (when must be ready)

Approval process:
├─ <$5k/year: Engineering lead approves
├─ $5k-20k/year: CTO approves
├─ >$20k/year: CEO + CFO approves (budget impact)

COST TRACKING & OPTIMIZATION:

Infrastructure costs:
├─ Monthly cost estimate: $20k
├─ Breakdown:
│   ├─ AWS EC2 (servers): $10k
│   ├─ AWS RDS (database): $5k
│   ├─ AWS S3 (storage): $1k
│   ├─ AWS CloudFront (CDN): $2k
│   ├─ DataDog (monitoring): $1k
│   ├─ SendGrid (email): $0.5k
│   ├─ Stripe (payment processing): 5% of revenue (~$1k for 100k orders)
│   └─ Other services: $0.5k
├─ Scaling: Add $10k/month for every 10x user growth
└─ Optimization: Monthly review (unused resources, overprovisioned instances)

Cost optimization opportunities:
├─ Reserved instances (AWS): Save 30% on compute (but 1-3 year commitment)
├─ Spot instances (AWS): Save 70% on compute (but can be interrupted)
├─ Auto-scaling: Only pay for resources in use (stop servers at night)
├─ Data compression: Reduce bandwidth (CDN costs)
├─ Query optimization: Reduce database load (RDS costs)
└─ Caching: Reduce API calls to third-party services

VENDOR LOCK-IN RISK:

High risk (difficult to switch):
├─ Cloud infrastructure (AWS): Requires re-architecting for GCP/Azure
├─ Database (RDS): Data migration effort
├─ Payment gateway (Stripe): Customer data portability
└─ Mitigation: Design for portability, avoid proprietary features

Medium risk:
├─ Email (SendGrid): Easy to switch to O365 or AWS SES
├─ Logging (Elasticsearch): Data portable to other solutions
└─ Mitigation: Standard formats (JSON logs, standard APIs)

Low risk (easy to switch):
├─ Analytics (GA): Just disconnect and setup new service
├─ CDN (CloudFront): Reconfigure DNS, data portable
└─ Mitigation: Minimal risk

DISASTER RECOVERY PLANNING:

Tier 1 Critical Services (RTO < 5 minutes):
├─ Cloud infrastructure (AWS)
├─ Database (RDS)
├─ Payment gateway (Stripe)
└─ Action: Replicate to backup region, auto-failover if available

Tier 2 Important Services (RTO < 30 minutes):
├─ Email (SendGrid)
├─ DNS (Route 53)
├─ Monitoring (DataDog)
└─ Action: Manual failover, restore from backup

Tier 3 Supportive Services (RTO < 24 hours):
├─ Logging (Elasticsearch)
├─ Analytics (GA)
├─ CDN (CloudFront)
└─ Action: Recreate from logs/backup

SERVICE UPTIME TRACKING:

Monthly dashboard:
├─ Stripe: Uptime %, response time
├─ SendGrid: Uptime %, bounce rate
├─ AWS: Region uptime %, service incidents
├─ DataDog: Collection uptime %
└─ Alert if any service < SLA target
```

**Output:** Third-party dependencies document with costs, SLAs, failover strategies, lock-in risk

---

## 1.5 DELIVERABLES & GATES

### Deliverable 1: Requirements Specification Document

**Contains:**
- All user stories with acceptance criteria (30-80 stories depending on project size)
- User personas (3-5 detailed)
- User journey maps (current state and desired state)
- Pain point analysis (with severity, frequency, impact)
- Feature prioritization (MoSCoW matrix)
- MVP scope definition (what's in, what's out)
- Non-functional requirements (performance, scalability, availability, security, accessibility)
- Technical constraints (budget, timeline, team, legacy systems, third-party services)

**Quality checks:**
- Every user story has 5-10 testable acceptance criteria
- No vague requirements ("fast" vs "p99 latency < 200ms")
- Success criteria are measurable
- Constraints clearly documented (no surprises mid-project)

---

### Deliverable 2: Feature Prioritization Matrix

**Contains:**
- MoSCoW categorization (MUST/SHOULD/COULD/WON'T)
- Impact vs effort matrix (high-impact/low-effort at top)
- Business justification for each categorization
- MVP definition (features shipping in launch)
- Phase 1.1 features (Phase 2, deferred)

**Quality checks:**
- Stakeholder alignment (everyone agrees with MUST vs SHOULD)
- Business owner signs off on scope

---

### Deliverable 3: Success Metrics Definition

**Contains:**
- KPIs (Key Performance Indicators) with targets
- User engagement metrics
- Business metrics (revenue, cost savings, ROI)
- Quality metrics (uptime SLA, error rates, performance targets)
- Measurement plan (how will we track, who owns)

**Quality checks:**
- Metrics are measurable (not "users are happy")
- Tracking is automated (not manual processes)
- Success criteria are realistic (achievable but ambitious)

---

### Deliverable 4: Technical Constraints & Integration Document

**Contains:**
- Budget breakdown (labor, infrastructure, tools, contingency)
- Timeline (milestones, critical path, external dependencies)
- Team composition (roles, hiring plan, onboarding timeline)
- Legacy system integration (API contracts, data migration)
- Third-party services (costs, SLAs, failover strategies)

**Quality checks:**
- Timeline is realistic (20-30% buffer for unknowns)
- Budget is complete (no hidden costs)
- Team can realistically deliver (skills match requirements)

---

## 1.6 APPROVAL GATES

### Gate 1: Requirements Completeness
**Criteria:**
- All user stories documented with acceptance criteria
- Success criteria are measurable and specific
- Stakeholders have reviewed and signed off
- Ambiguity has been resolved

**Who approves:** Product Manager, Business Sponsor

---

### Gate 2: Feasibility Confirmation
**Criteria:**
- Technical team confirms all requirements are buildable
- Timeline is realistic (not optimistic)
- Budget is reasonable
- No showstoppers identified

**Who approves:** CTO, Engineering Lead, Project Sponsor

---

### Gate 3: Scope & Priority Alignment
**Criteria:**
- MVP is clearly defined (what launches, what doesn't)
- MoSCoW is agreed by all stakeholders
- Business owner confirms priorities are correct
- Phase 1.1 features are identified and deferred

**Who approves:** Product Manager, Business Sponsor, Engineering Lead

---

### Final Gate: Requirements Approved for Phase 2
**Criteria:**
- All deliverables complete
- All gates passed
- No ambiguity or blockers remaining
- Team ready to start architecture phase

**Who approves:** Project Sponsor, CTO

---

## 1.7 ANTI-PATTERNS & PITFALLS

| Pitfall | Why It Kills Projects | How to Avoid |
|---------|---------------------|------------|
| **Vague acceptance criteria** | "User can order product" vs "User can add item to cart, select shipping, enter payment, see order confirmation" - first is untestable | Write specific, measurable criteria (3-10 per story) |
| **Too many features in MVP** | 80 features → 6 month timeline → competitors ship faster → lose market | MVP = Minimum, not all features (ruthlessly cut) |
| **No prioritization** | Everything is MUST-have → timeline slip → cut happens anyway, but chaotic | Use MoSCoW, stick to it, defer ruthlessly |
| **Scope creep** | "Let's add X while we're at it" → timeline slip → quality suffers | Scope is locked after Phase 1 (changes = Phase 1.1) |
| **Unrealistic timeline** | 12-week estimate for 16-week project → disappointment → morale drop | Add 20-30% contingency, use historical data |
| **Stakeholder misalignment** | Finance wants cost savings, Marketing wants growth features, CEO wants revenue → conflict mid-project | Align on success criteria upfront (written down) |
| **Hidden constraints discovered mid-project** | "Oh, we need GDPR compliance" → architecture changes → delay | Surface all constraints in Phase 1 |
| **Budget doesn't cover reality** | $100k budget, but team costs $150k → project underfunded | Budget must be complete (don't hide costs) |
| **Team skill mismatch** | Junior team for complex project → velocity too low → timeline slip | Assess team capability, hire or contract for gaps |
| **External dependency underestimated** | "Partner API is easy" → API actually complex → 4 weeks instead of 2 | Contact external partners early, understand their SLA |
| **No success metrics** | Ship product, but no one knows if it's working | Define KPIs upfront (automated measurement) |
| **Accessibility ignored** | "We'll add accessibility later" → never happens → legal risk | Include accessibility in every user story acceptance criteria |
| **Performance targets undefined** | Ship, then discover "too slow" → architecture redesign | Define p99 latency, throughput targets upfront |

---

## 1.8 PHASE 1 CHECKLIST

- [ ] All stakeholder interviews completed (10+ interviews)
- [ ] 3-5 user personas created with interview quotes
- [ ] Current state + desired state journey maps created
- [ ] Pain points extracted and prioritized (severity/frequency)
- [ ] 30-80 user stories written with acceptance criteria
- [ ] User stories estimated (story points assigned)
- [ ] Use cases documented for critical workflows
- [ ] Feature prioritization (MoSCoW) completed
- [ ] MVP scope defined (what ships, what's cut)
- [ ] Feature dependencies mapped
- [ ] Success criteria defined (measurable KPIs)
- [ ] Performance targets set (p50/p95/p99 latencies)
- [ ] Scalability plan created (growth projection, scaling strategy)
- [ ] SLA/SLO/RTO/RPO defined
- [ ] Security & compliance requirements documented
- [ ] Accessibility standards (WCAG 2.1 AA) defined
- [ ] Browser/device compatibility matrix created
- [ ] Internationalization requirements documented (if applicable)
- [ ] Budget breakdown completed (labor, infrastructure, tools)
- [ ] Timeline with milestones created (realistic, with buffer)
- [ ] External dependencies identified and timeline confirmed
- [ ] Team hiring plan created (who, when, ramp-up timeline)
- [ ] Legacy system integrations documented
- [ ] Third-party services (costs, SLAs, failover) documented
- [ ] Requirements Specification Document written
- [ ] Feature Prioritization Matrix created
- [ ] Success Metrics Definition documented
- [ ] Technical Constraints Document written
- [ ] Stakeholder sign-off obtained (all gates passed)
- [ ] No ambiguity remaining (all questions answered)
- [ ] Team ready to proceed to Phase 2

---

## 1.9 ESTIMATED DURATION

**Timeline: 2-3 weeks**

- Stakeholder interviews & personas: 4-5 days
- User journeys & pain points: 3-4 days
- User stories & prioritization: 4-5 days
- Non-functional requirements & constraints: 3-4 days
- Documentation & approval: 2-3 days
- **Total: 16-21 days (2-3 weeks)**

**Parallel work:** Requirements gathering while scheduling interviews, pain point analysis while interviews happening

---

## 1.10 TRANSITION TO PHASE 2

**When Phase 1 is complete:**
- Requirements are signed off by all stakeholders
- Success criteria are measurable and agreed
- MVP scope is locked (no more additions)
- Team has confirmed feasibility
- Timeline is realistic
- Budget is approved
- All constraints are documented
- No surprises waiting

**Phase 2 begins when:** Stakeholder formally approves proceeding to architecture design

---

**END OF PHASE 1**

Phase 1 is the foundation. Quality here directly translates to project success. Rushing requirements causes 40% of project delays. Invest 2-3 weeks here, save 4-8 weeks downstream.