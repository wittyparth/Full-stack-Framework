# PHASE 2: SYSTEM ARCHITECTURE & DESIGN

## Overview
Architecture determines everything downstream: scalability, maintainability, cost, performance, security. Bad architecture decisions made here are 10x more expensive to fix later. This phase transforms requirements into a buildable blueprint.

---

## 2.1 ARCHITECTURE DECISION MAKING

### 2.1.1 Monolith vs Microservices Decision Framework

**What to evaluate:**

**MONOLITH ARCHITECTURE:**

When to choose monolith:
├─ Team size: < 10 engineers (microservices overhead not justified)
├─ Project complexity: Medium (3-5 major components)
├─ Timeline pressure: Tight (faster to build initially)
├─ Scalability needs: Single feature needs scaling (can scale entire app)
├─ Operational maturity: Early stage (fewer operational concerns)
└─ Database: Single database, strong consistency needed

Monolith advantages:
├─ Simpler development (fewer moving parts, easier debugging)
├─ Easier deployment (one artifact, one deploy, rollback simple)
├─ Better performance initially (no network latency between components)
├─ Transaction support (ACID transactions across features)
├─ Easier testing (integration tests simpler)
├─ Cheaper infrastructure (single server or load balanced servers)
└─ Faster MVP (less complexity to coordinate)

Monolith disadvantages:
├─ Scalability wall (hit ~500 concurrent users, then scaling becomes complex)
├─ Technology lock-in (entire app uses same language/framework)
├─ Deployment risk (one bug = entire app down)
├─ Team coordination (all teams modifying same codebase)
├─ Data coupling (schema change affects entire app)
└─ Separate scaling of components impossible (pay for components you don't use)

Monolith failure mode:
- Starts at 1k LOC (fast)
- Grows to 100k LOC (still manageable)
- Grows to 1M LOC (slow velocity, difficult onboarding, high defect rate)
- Needs rewrite (sunk cost, team burned out)

**MICROSERVICES ARCHITECTURE:**

When to choose microservices:
├─ Team size: > 15 engineers (distributed team benefits)
├─ Project complexity: High (5+ independent services)
├─ Scaling needs: Different components scale independently (Orders 1000 RPS, Reports 10 RPS)
├─ Technology diversity: Need Python + Go + Node in different services
├─ Organizational maturity: DevOps team, mature processes, monitoring
├─ High availability requirement: Can't afford single point of failure
└─ Independent deployment: Different teams shipping at different cadence

Microservices advantages:
├─ Independent scaling (scale Order Service 10x, keep Report Service at 1x)
├─ Technology diversity (Order Service in Go, Analytics in Python, Frontend in React)
├─ Independent deployment (Order Service deploys without affecting Reports)
├─ Resilience (one service fails, others still work)
├─ Team independence (Team A owns Orders, Team B owns Analytics)
├─ Easier to replace components (rewrite one service without touching others)
└─ Potentially unlimited scalability (add more services, not more instances)

Microservices disadvantages:
├─ Operational complexity (monitor 10 services instead of 1)
├─ Network latency (service-to-service calls slower than function calls)
├─ Distributed transactions (ACID no longer possible, eventual consistency)
├─ Data consistency challenges (data lives in multiple services)
├─ Testing complexity (integration tests need multiple services running)
├─ Infrastructure costs (more servers, monitoring, observability)
├─ Debugging difficulty (error happens across 3 services, hard to trace)
└─ DevOps maturity required (containers, orchestration, service mesh)

Microservices failure mode:
- Start 2-3 services (extra complexity not justified)
- Network latency kills performance (calls go p50 from 10ms to 100ms)
- Data consistency issues (orders created but inventory not updated)
- Operational burden (team spends 50% of time on ops instead of features)
- Cost explosion (infrastructure > development savings)

**DECISION FRAMEWORK:**

```
Team size < 10?
├─ YES → Monolith (microservices overhead not justified)
└─ NO → Continue

Need independent scaling for different components?
├─ YES → Microservices (Orders 1000 RPS, Reports 10 RPS)
└─ NO → Continue

Need different technology stacks for different components?
├─ YES → Microservices (Python for ML, Go for performance, Node for APIs)
└─ NO → Continue

High availability non-negotiable?
├─ YES → Microservices (one service failure shouldn't impact others)
└─ NO → Continue

Timeline critical (need MVP in 4 weeks)?
├─ YES → Monolith (faster to build)
└─ NO → Continue

→ MONOLITH is optimal choice
```

**For this project:** MONOLITH architecture selected (team size 7, medium complexity, tight timeline, single database)

**Monolith structure:**
```
├─ Backend API (single codebase)
│  ├─ Authentication & Authorization
│  ├─ Order Management
│  ├─ Inventory Management
│  ├─ Payment Processing
│  ├─ Shipping Integration
│  └─ Reporting & Analytics
├─ Frontend (React SPA)
├─ Database (PostgreSQL)
├─ Cache (Redis)
└─ Job Queue (for background processing)
```

**Future migration path:** If service X becomes bottleneck (e.g., Reports generating 1000 RPS), extract as separate service without rearchitecting entire app.

---

### 2.1.2 Tech Stack Selection with Justification

**What to decide:**
- Backend language & framework
- Frontend framework & build tools
- Database (primary, secondary)
- Caching layer
- Message queue
- Authentication/Authorization solution
- Monitoring & logging infrastructure

**BACKEND TECHNOLOGY SELECTION:**

Decision criteria:
├─ Performance (throughput, latency at scale)
├─ Developer productivity (speed to market)
├─ Learning curve (team can onboard quickly)
├─ Ecosystem (libraries, tools, community support)
├─ Long-term viability (won't die in 2 years)
├─ Operational simplicity (easy to monitor, debug)
└─ Cost (compute resources needed)

Candidates evaluated:

**Node.js/Express:**
```
Pros:
├─ JavaScript ecosystem (reuse code with frontend)
├─ Non-blocking I/O (handles high concurrency)
├─ Fast startup (good for serverless)
├─ Rich npm ecosystem
└─ Team already knows JavaScript

Cons:
├─ Single-threaded (CPU-bound tasks struggle)
├─ Memory usage (heavier than Go, lighter than JVM)
├─ Debugging complexity (async callbacks, promises)
├─ Production monitoring tools less mature
└─ Not ideal for compute-heavy operations (reports, analytics)

Performance profile:
├─ Throughput: 5-10k RPS per instance (good)
├─ Latency: p99 < 100ms (good)
├─ Concurrency: 10k connections/process (excellent)
└─ Memory: 200-300MB per process (medium)

Recommendation: SELECTED for this project
├─ Reason 1: Team knows JavaScript
├─ Reason 2: Frontend integration (code sharing)
├─ Reason 3: Rapid development (express.js ecosystem)
├─ Reason 4: Our scale (100-500 concurrent users, Node handles easily)
└─ Fallback if performance issues: Extract report generation to Python/Go
```

**Go:**
```
Pros:
├─ Blazingly fast (compiled, not interpreted)
├─ Handles concurrency elegantly (goroutines)
├─ Single binary (easy to deploy)
├─ Fast startup (milliseconds)
├─ Excellent for systems programming
└─ Built-in profiling tools

Cons:
├─ Steeper learning curve (if team unfamiliar)
├─ Smaller ecosystem than Node/Python
├─ Less suitable for rapid iteration
├─ Fewer libraries available
└─ Compile time (not compiled code like Node)

Performance profile:
├─ Throughput: 20-50k RPS per instance (excellent)
├─ Latency: p99 < 50ms (excellent)
├─ Concurrency: 100k+ connections (outstanding)
└─ Memory: 50-100MB per process (excellent)

Recommendation: NOT SELECTED for MVP
├─ Reason: Team needs ramp-up time
├─ Consideration: If performance becomes bottleneck, extract hot path to Go
```

**Python/Django:**
```
Pros:
├─ Rapid development (very productive)
├─ Rich ecosystem (Django ORM excellent)
├─ Great for data processing (numpy, pandas)
├─ Excellent documentation
└─ Perfect for background jobs

Cons:
├─ Slower performance (interpreted language)
├─ GIL limitation (multi-threading doesn't parallelize)
├─ Memory usage (heavier than Go/Node)
├─ Scaling requires more instances
└─ Deployment complexity (virtual environments, dependencies)

Performance profile:
├─ Throughput: 500-2k RPS per instance (poor)
├─ Latency: p99 < 500ms (slow)
├─ Concurrency: 100 connections/process (limited)
└─ Memory: 500MB+ per process (heavy)

Recommendation: NOT SELECTED for primary API
├─ Reason: Performance not suitable for order processing (1000 RPS peak)
├─ Use case: Background job processing (reports, analytics, batch operations)
```

**Java/Spring:**
```
Pros:
├─ Industrial strength (proven at scale)
├─ Type safety (catches bugs at compile time)
├─ JVM ecosystem (massive)
├─ Performance (JIT compilation optimizes at runtime)
└─ Monitoring tools (excellent APM integration)

Cons:
├─ Heavy (JVM startup 2-3 seconds)
├─ Memory hungry (base JVM 300MB+)
├─ Slow development iteration (compile → restart)
├─ Operational complexity (JVM tuning required)
└─ Team inexperience (if using Go or Node currently)

Performance profile:
├─ Throughput: 10-20k RPS per instance (good)
├─ Latency: p99 < 100ms (good, after warmup)
├─ Concurrency: 10k connections (good)
└─ Memory: 500MB+ per process (heavy)

Recommendation: NOT SELECTED for MVP
├─ Reason: Team more productive with Node/Go
├─ Not a wrong choice (many companies successful with Spring)
```

**FINAL DECISION: Node.js + Express**
```
Why Node.js:
├─ Team already knows JavaScript
├─ Express.js is lightweight (easy to understand)
├─ Performance adequate for our scale (500 concurrent users)
├─ Rapid development (npm ecosystem huge)
├─ Good for I/O-heavy operations (orders, inventory, payments)
└─ Can extract performance-critical paths later if needed

Technology stack:
├─ Runtime: Node.js 20 LTS
├─ Framework: Express.js 4.x
├─ Language: TypeScript (for type safety)
├─ API style: REST (not GraphQL, GraphQL adds complexity)
├─ Database driver: PostgreSQL native (pg library)
├─ ORM: Prisma (type-safe, modern, better than Sequelize/TypeORM)
├─ Testing: Jest (unit/integration), Supertest (API testing)
├─ Linting: ESLint + Prettier (code quality)
├─ Validation: Zod (runtime type checking)
└─ Environment: Docker (consistent dev/prod)
```

**FRONTEND TECHNOLOGY SELECTION:**

Decision criteria:
├─ Developer productivity (build speed, iteration time)
├─ Performance (bundle size, initial load, runtime performance)
├─ Learning curve (team can onboard quickly)
├─ Ecosystem (UI libraries, routing, state management)
├─ Long-term viability (actively maintained)
└─ Mobile compatibility (responsive, touch-friendly)

Candidates evaluated:

**React:**
```
Pros:
├─ Team knows React (no learning curve)
├─ Largest ecosystem (UI libraries, routing, state management)
├─ Component reusability (speeds up development)
├─ Virtual DOM (performance optimization built-in)
├─ Strong community (lots of examples, solutions)
└─ Flexible (can use any state management)

Cons:
├─ Boilerplate heavy (need routing, state mgmt libraries)
├─ Bundle size can be large (if not optimized)
├─ Decision fatigue (too many libraries to choose from)
└─ SSR complexity (if needed for SEO)

Recommendation: SELECTED
├─ Reason 1: Team familiar
├─ Reason 2: Largest ecosystem
├─ Reason 3: Component reusability
├─ Reason 4: Flexibility (add complexity only if needed)
```

**Vue.js:**
```
Pros:
├─ Easier learning curve than React
├─ Smaller bundle size (lighter than React)
├─ Great documentation
├─ Responsive object system (less boilerplate)
└─ Single-file components (HTML/CSS/JS together)

Cons:
├─ Smaller ecosystem than React
├─ Less job market (harder to hire)
├─ Fewer UI libraries available
└─ Team would need ramp-up

Recommendation: NOT SELECTED
├─ Reason: Team knows React, no benefit to switching
```

**Angular:**
```
Pros:
├─ Full-featured framework (routing, HTTP, forms included)
├─ Strong typing (TypeScript integrated)
├─ Google-backed (long-term support)
└─ Great for large applications

Cons:
├─ Steep learning curve (large framework)
├─ Verbose (lots of boilerplate)
├─ Heavy framework (larger bundle)
├─ Slower iteration (more setup)
└─ Overkill for this project

Recommendation: NOT SELECTED
├─ Reason: Overkill for MVP
```

**FINAL DECISION: React 18**
```
Why React:
├─ Team expertise (no ramp-up time)
├─ Largest ecosystem
├─ Component reusability
├─ Performance with concurrent features
└─ Flexibility (add complexity as needed)

Frontend stack:
├─ Framework: React 18
├─ Language: TypeScript (type safety)
├─ Routing: React Router v6
├─ State management: TanStack Query (server state) + Zustand (client state)
├─ UI library: Headless UI (accessibility built-in)
├─ Styling: Tailwind CSS (utility-first, fast development)
├─ Form handling: React Hook Form + Zod validation
├─ Testing: Vitest (unit), React Testing Library (components), Playwright (E2E)
├─ Build tool: Vite (fast development, optimized production builds)
├─ Package manager: pnpm (faster, disk-efficient)
└─ Environment: Docker (consistent dev/prod)
```

**DATABASE TECHNOLOGY SELECTION:**

Decision criteria:
├─ Data model (relational vs document)
├─ Consistency requirements (ACID vs eventual)
├─ Scalability (how much data, how fast it grows)
├─ Query patterns (what queries run most frequently)
├─ Operational complexity (backup, monitoring, scaling)
└─ Cost (storage, compute, licensing)

Candidates evaluated:

**PostgreSQL (Relational):**
```
Pros:
├─ ACID transactions (strong consistency)
├─ Complex queries (joins, aggregations)
├─ Great for structured data (users, orders, inventory)
├─ Excellent performance (with proper indexing)
├─ Open source (no licensing costs)
├─ Rich features (JSONB, arrays, full-text search)
├─ Great tooling (pgAdmin, migrations)
└─ Proven at massive scale (used at Netflix, Spotify, etc.)

Cons:
├─ Vertical scaling limit (single server has limits)
├─ Complex sharding (if need horizontal scaling)
├─ Not ideal for unstructured data
└─ Requires schema definition upfront

Recommendation: SELECTED as primary database
├─ Reason 1: Structured data (users, orders, inventory)
├─ Reason 2: ACID transactions needed
├─ Reason 3: Complex queries (inventory checks, order reports)
├─ Reason 4: Team familiar with SQL
├─ Reason 5: Our scale (50GB database at Year 2)
```

**MongoDB (Document):**
```
Pros:
├─ Flexible schema (no schema migration needed)
├─ Good for unstructured/semi-structured data
├─ Horizontal scaling built-in (sharding)
├─ Developer-friendly (JSON-like documents)
└─ Good for rapid prototyping

Cons:
├─ Eventual consistency (no ACID by default)
├─ Larger storage footprint (duplication)
├─ Complex transactions (need multi-document ACID, added complexity)
├─ Harder to query (no JOINs)
├─ Licensing concerns (SSPL license)
└─ Not ideal for financial data (orders, payments)

Recommendation: NOT SELECTED as primary
├─ Reason: Order/inventory data needs ACID transactions
├─ Use case: If adding event logging (unstructured data), could use MongoDB
```

**FINAL DECISION: PostgreSQL 15**
```
Why PostgreSQL:
├─ Structured data (users, orders, inventory)
├─ ACID transactions (orders can't be partially created)
├─ Complex queries (inventory management, reporting)
├─ Team familiarity
├─ Proven at scale (Netflix, Spotify, AWS customers)
└─ Cost-effective (open source)

Database setup:
├─ Managed service: AWS RDS PostgreSQL (reduces ops burden)
├─ Version: PostgreSQL 15 (latest stable)
├─ Replication: Read replicas for analytics queries (don't impact main DB)
├─ Backup: Automated daily (full + incremental)
├─ Monitoring: CloudWatch metrics, slow query logs
└─ Scaling: RDS can auto-scale storage (don't run out of disk)
```

**CACHING LAYER:**

Decision: **Redis**
```
Why Redis:
├─ Fast (in-memory, sub-millisecond access)
├─ Flexible data structures (strings, hashes, lists, sets)
├─ High throughput (100k+ ops per second)
├─ Cache invalidation (TTL, key patterns)
├─ Sessions (distributed sessions across servers)
└─ Pub/Sub (real-time notifications if needed)

Use cases:
├─ Session storage (user login tokens)
├─ Cache layer (database query results)
├─ Rate limiting (track API calls per user)
├─ Inventory cache (avoid database hammering)
└─ Message queue (pub/sub for notifications)

Setup:
├─ Managed service: AWS ElastiCache Redis
├─ Size: Start 256MB (grow as needed)
├─ Eviction policy: allkeys-lru (evict least-used when full)
├─ Persistence: RDB snapshots (backup every hour)
└─ Replication: Multi-AZ for high availability
```

**MESSAGE QUEUE:**

Decision: **Bull (Redis-backed)**
```
Why Bull:
├─ Built on Redis (no additional infrastructure)
├─ Job scheduling (delayed jobs, cron-like)
├─ Retry logic (exponential backoff)
├─ Worker pool (scalable background processing)
├─ Rate limiting (control job throughput)
└─ Monitoring dashboard (BullBoard)

Use cases:
├─ Email sending (don't block API response)
├─ Order processing (async workflow)
├─ Report generation (background job)
├─ Inventory sync (batch operations)
└─ Webhook retries (eventual delivery)

Setup:
├─ npm package: bull
├─ Dashboard: BullBoard (UI to monitor jobs)
├─ Workers: Separate process pool
└─ Dead letter queue: Failed jobs stored for manual review
```

**AUTHENTICATION/AUTHORIZATION:**

Decision: **JWT tokens + Passport.js**
```
Why JWT:
├─ Stateless (don't need session storage for every request)
├─ Scalable (works across multiple servers)
├─ Mobile-friendly (no cookies required)
├─ API-first (standard for REST APIs)
└─ Clear expiration (tokens expire after time)

Why Passport.js:
├─ Proven strategy library (supports 500+ strategies)
├─ OAuth2 support (if adding social login later)
├─ Middleware pattern (clean integration with Express)
└─ Active community

Implementation:
├─ Access token (15-minute expiry, in memory)
├─ Refresh token (30-day expiry, in Redis)
├─ Roles (admin, manager, employee)
├─ Permissions (can user do action X)
└─ Audit logging (track who did what)

Flow:
├─ User logs in (email + password)
├─ Verify password (bcrypt compare)
├─ Issue access + refresh tokens
├─ Client stores tokens (memory, not localStorage for security)
├─ API requests include Authorization header (Bearer token)
├─ Server validates token (signature + expiry)
└─ Token expired → Use refresh token to get new access token
```

**MONITORING & LOGGING:**

Decision: **DataDog**
```
Why DataDog:
├─ Unified monitoring (metrics, logs, traces)
├─ APM (application performance monitoring)
├─ Alerting (PagerDuty integration for on-call)
├─ Dashboards (custom visualizations)
└─ Scalable (handles our growth)

Alternatives considered:
├─ Prometheus + Grafana (open source, DIY)
│  ├─ Pros: No vendor lock-in, cost-effective
│  ├─ Cons: Operational burden (maintain, upgrade)
│  └─ Decision: Not selected (team size too small for Prometheus overhead)
├─ CloudWatch (AWS native)
│  ├─ Pros: Integrated with AWS
│  ├─ Cons: Less feature-rich, slower to respond to issues
│  └─ Decision: Use CloudWatch for basic metrics, DataDog for advanced
└─ New Relic (competitor)
    ├─ Pros: Good APM
    ├─ Cons: More expensive than DataDog
    └─ Decision: DataDog better value

Setup:
├─ APM agent: datadog/browser-sdk (frontend)
├─ APM agent: dd-trace (Node.js backend)
├─ Log forwarding: Datadog agent
├─ Metrics: Custom metrics via SDK
├─ Dashboards: System health, API latency, error rates
└─ Alerts: P0 (immediate page), P1 (email alert), P2 (Slack message)

Monitoring targets:
├─ API response time (p50, p95, p99)
├─ Error rate (5xx, 4xx)
├─ Database query time
├─ Cache hit rate
├─ Background job duration
├─ Queue depth (jobs waiting)
└─ Infrastructure (CPU, memory, disk, network)
```

**TECH STACK SUMMARY TABLE:**

```
Layer               | Technology        | Version | Why Selected
--------------------|------------------|---------|------------------------------------------
Backend Runtime    | Node.js          | 20 LTS  | Team knows JS, fast I/O
Backend Framework  | Express.js       | 4.x     | Lightweight, large ecosystem
Language           | TypeScript       | 5.x     | Type safety, catches bugs early
Database (Primary) | PostgreSQL       | 15      | ACID, complex queries, structured data
Cache              | Redis            | 7.x     | Sub-millisecond, handles high load
Message Queue      | Bull (Redis)     | -       | Job scheduling, no separate infra
ORM/Query Builder  | Prisma           | 5.x     | Type-safe, great DX
Auth               | JWT + Passport   | -       | Stateless, scalable
Frontend           | React            | 18      | Team expertise, large ecosystem
Frontend Routing   | React Router     | 6.x     | Standard, well-maintained
State Management   | Zustand + TQ     | -       | Lightweight, easy to understand
Form Validation    | Zod + RHF        | -       | Type-safe validation
Styling            | Tailwind CSS     | 3.x     | Utility-first, fast dev
Build Tool         | Vite             | 5.x     | Fast builds, optimized output
Testing (Unit)     | Vitest           | 1.x     | Fast, Vite-integrated
Testing (E2E)      | Playwright       | 1.x     | Cross-browser, reliable
Monitoring         | DataDog          | -       | Unified observability
Logging            | Winston/Pino     | -       | Structured logging
CI/CD              | GitHub Actions   | -       | Built-in with GitHub, free
Container          | Docker           | -       | Consistent dev/prod
Orchestration      | Docker Compose   | -       | Local, simple for MVP
Cloud Platform     | AWS              | -       | Mature, reliable, scalable
```

---

### 2.1.3 Architecture Decision Records (ADRs)

**What to do:**
- Document every major decision (not minor coding choices)
- Include: Context, Decision, Rationale, Consequences, Alternatives considered
- Store in version control (referenced in code)
- Reviewable by team (captures decision-making)

**ADR format:**

```
ADR-001: Use Node.js + Express for Backend

Status: ACCEPTED (Date: Dec 15, 2023)
Supersedes: None
Superseded by: None

Context:
- Team consists of full-stack developers with strong JavaScript/TypeScript knowledge
- Project requires handling 500-1000 concurrent users initially
- Need rapid development (MVP in 16 weeks)
- Expected growth to 10,000 users in Year 1
- Backend needs to process orders in real-time (< 200ms p99)

Decision:
Use Node.js 20 LTS + Express.js as the backend framework, with TypeScript for type safety.

Rationale:
1. Team productivity: Developers can iterate fast without context switching (JavaScript everywhere)
2. Performance adequate: Node.js handles our expected throughput (5-10k RPS per instance easily)
3. Ecosystem: Express has mature libraries for everything needed (auth, validation, testing)
4. Scalability path: If performance becomes bottleneck, can extract components to Go without refactoring everything
5. Cost-effective: Node.js apps run on smaller instances than JVM apps (memory footprint 200MB vs 500MB+)

Consequences:
1. Good: Team can onboard quickly (knows JavaScript already)
2. Good: Rapid MVP development (Express ecosystem proven)
3. Good: Cost-effective (Node uses less memory than alternatives)
4. Bad: Single-threaded (CPU-bound tasks need separate process)
5. Bad: Production monitoring requires careful setup (less mature tooling than Java)
6. Neutral: Future extraction possible if specific component becomes bottleneck

Alternatives Considered:
1. Go: Better performance, but team learning curve
   ├─ Rejected because: MVP timeline critical, team ramp-up time not acceptable
   └─ Mitigation: Can extract performance-critical paths to Go later (no rework needed)

2. Java/Spring: Industrial strength, proven at scale
   ├─ Rejected because: Team more productive with JavaScript, ops complexity higher
   └─ Decision revisited if: Company hires Java-expert architect

3. Python/Django: Great for rapid dev, but performance not suitable for order processing
   ├─ Rejected because: 500 RPS per instance not sufficient for our scale
   └─ Use case: Can use Python for separate analytics service

Validation:
- Performance testing: Node.js handles 10,000 concurrent connections with <200ms p99 latency ✓
- Team knowledge: All 2 backend engineers experienced with Node.js ✓
- Ecosystem validation: All needed libraries available and maintained ✓

Review:
- Reviewed by: CTO, Backend Lead, Tech Lead
- Approved by: CTO
- Review date: Dec 15, 2023

Questions & Answers:
Q: What if Node.js becomes performance bottleneck?
A: Design monolith with service boundaries. Can extract hot service to Go without rewriting everything.

Q: What about memory leaks in production?
A: Use APM monitoring (DataDog) to catch leaks early. Implement graceful shutdown for deployments.

Q: How do we handle CPU-intensive tasks?
A: Use worker threads or separate Python process for compute. Don't run on main thread.
```

**ADRs needed for this project:**

```
ADR-001: Node.js + Express backend (documented above)
ADR-002: PostgreSQL as primary database (ACID for orders)
ADR-003: React + TypeScript frontend (team expertise)
ADR-004: Redis for caching + session storage
ADR-005: Bull for background job queue
ADR-006: JWT for stateless authentication
ADR-007: REST API (not GraphQL, keeps complexity lower for MVP)
ADR-008: Monolith architecture (not microservices, smaller team)
ADR-009: AWS for cloud infrastructure (team familiar)
ADR-010: TanStack Query + Zustand for state management
```

**Output:** ADRs stored in `docs/adr/` directory, referenced in code

---

### 2.1.4 Cloud Provider Selection

**Decision Framework:**

```
AWS vs GCP vs Azure vs On-Premises

Evaluation criteria:
├─ Cost (compute, storage, data transfer)
├─ Regional availability (need 50+ regions)
├─ Service breadth (RDS, ElastiCache, S3, etc.)
├─ Team experience (who knows what)
├─ Vendor lock-in risk
├─ Support quality
└─ Integration with other services

AWS:
├─ Cost: Moderate (pay-as-you-go, reserved instances discount)
├─ Regions: 33+ global regions (excellent coverage)
├─ Services: 200+ (biggest ecosystem)
├─ Team experience: All engineers have AWS experience
├─ Lock-in: Medium (ECS/RDS specific, but portable to other clouds)
├─ Support: Good (Professional support available)
└─ Verdict: SELECTED (best for our team + needs)

GCP:
├─ Cost: Slightly cheaper (better compute pricing)
├─ Regions: 40+ regions
├─ Services: Fewer than AWS, but excellent for data/ML
├─ Team experience: Some engineers know GCP
├─ Lock-in: Medium
├─ Support: Good
└─ Verdict: NOT SELECTED (team more familiar with AWS)

Azure:
├─ Cost: Good (competitive pricing, good for .NET)
├─ Regions: 60+ regions (most global)
├─ Services: 200+ (comparable to AWS)
├─ Team experience: Minimal (no experts)
├─ Lock-in: High (Microsoft ecosystem)
├─ Support: Good
└─ Verdict: NOT SELECTED (team learning curve not acceptable)

On-Premises:
├─ Cost: High (capital expenditure, ops burden)
├─ Regions: Single (no geographic redundancy)
├─ Services: Limited (must build/manage everything)
├─ Team experience: Operational burden high
├─ Scaling: Manual, painful
└─ Verdict: NOT SELECTED (unacceptable ops burden)

FINAL DECISION: AWS
```

**AWS service selection:**

```
Compute:
├─ EC2 (virtual machines): For web servers, not recommended for stateless apps
├─ ECS (container orchestration): SELECTED (simpler than Kubernetes)
├─ Fargate (serverless containers): SELECTED (no instance management)
├─ Lambda (serverless functions): For background jobs, not primary API

Recommended: ECS on Fargate (best balance of simplicity + scaling)
├─ Why: Automatically scale based on load
├─ Why: No server management
├─ Why: Pay only for compute used
└─ Cost: ~$50-100/day for our load

Database:
├─ RDS PostgreSQL: SELECTED (managed, automated backups, multi-AZ)
├─ DynamoDB: Not suitable (unstructured, eventual consistency)
└─ Aurora PostgreSQL: More expensive, marginal benefit

Storage:
├─ S3: SELECTED (file uploads, backups, CDN origin)
├─ EBS: Not needed (stateless containers)
└─ EFS: Not needed (no shared filesystem)

Networking:
├─ VPC: SELECTED (private network, security)
├─ ALB (Application Load Balancer): SELECTED (distribute traffic)
├─ CloudFront: SELECTED (CDN, edge caching)

Caching/Queues:
├─ ElastiCache (Redis): SELECTED (session storage, caching)
├─ SQS: Not selected (using Bull + Redis instead)

Monitoring:
├─ CloudWatch: Included with AWS
├─ DataDog: SELECTED (more features)

DNS:
├─ Route 53: SELECTED (domain management, health checks)

Backup:
├─ AWS Backup: SELECTED (automated, centralized)

Cost estimate (Year 1):
├─ Fargate: $25,000
├─ RDS PostgreSQL: $8,000
├─ ElastiCache Redis: $1,000
├─ S3 storage + transfer: $2,000
├─ CloudFront CDN: $3,000
├─ ALB: $160
├─ Route 53: $12
├─ DataDog monitoring: $12,000
├─ Miscellaneous: $500
└─ Total: ~$51,700/year (~$4,300/month)

Cost optimization opportunities:
├─ Reserved Fargate (1 year): Save 30% (~$7,500/year)
├─ Reserved RDS (1 year): Save 30% (~$2,400/year)
├─ Spot instances (for batch jobs): Save 70% but less reliable
└─ Potential savings: ~$10k/year with commitment
```

---

### 2.1.5 Deployment Strategy Decision

**What to do:**
- Choose deployment approach (containers, traditional VMs, serverless)
- Choose orchestration (Docker Compose, Kubernetes, ECS, etc.)
- Design for zero-downtime deployments
- Plan rollback strategy

**Deployment options:**

```
DEPLOYMENT APPROACH COMPARISON:

Traditional VMs (EC2):
├─ Pros: Full control, flexible
├─ Cons: Need to manage OS patches, scaling manual
├─ Best for: Complex deployments with special requirements
└─ Verdict: NOT SELECTED (too much ops overhead)

Containers + Kubernetes:
├─ Pros: Industry standard, scalable, self-healing
├─ Cons: Significant learning curve, operational complexity
├─ Best for: Teams with DevOps expertise, 50+ services
└─ Verdict: NOT SELECTED (overkill for team size)

Containers + ECS on Fargate:
├─ Pros: Managed Kubernetes alternative, AWS integration
├─ Cons: AWS lock-in, less flexible than Kubernetes
├─ Best for: Teams wanting containers without Kubernetes complexity
└─ Verdict: SELECTED (sweet spot for our needs)

Serverless (Lambda):
├─ Pros: Minimal ops, pay per invocation, auto-scaling
├─ Cons: Cold starts (100-200ms), execution time limits, harder to debug
├─ Best for: Event-driven workloads, not continuous APIs
└─ Verdict: NOT SELECTED for primary API (order processing needs < 200ms p99)

FINAL DECISION: ECS on Fargate

Why ECS on Fargate:
├─ Managed service (AWS handles scaling, patching)
├─ Pay only for compute used (containers only run when needed)
├─ Easy to scale (horizontal scaling 2 → 10 containers)
├─ Docker integration (same containers locally and production)
├─ CloudWatch integration (native monitoring)
└─ Cost-effective (cheaper than EC2 for our load)
```

**Deployment pipeline:**

```
Code push to main branch
         ↓
GitHub Actions workflow triggered
         ↓
Run tests (unit, integration)
         ↓
Build Docker image
         ↓
Push image to ECR (AWS image registry)
         ↓
Run security scanning (Snyk, AWS Inspector)
         ↓
Deploy to staging environment
         ↓
Run smoke tests + E2E tests
         ↓
Manual approval (release manager)
         ↓
Deploy to production
         ↓
Health checks + monitoring
         ↓
Rollback if issues detected
```

**Zero-downtime deployment:**

```
Strategy: Blue-Green Deployment

Current state (Blue):
├─ 3 containers running API v1.0
├─ Traffic routed to Blue via ALB
└─ Database at schema v1

Deployment:
├─ Step 1: Create new RDS instance (v2, same schema as v1)
├─ Step 2: Start 3 new containers with API v2 (Green)
├─ Step 3: Health check all Green containers
├─ Step 4: Switch ALB traffic to Green
├─ Step 5: Monitor for errors (5 min)
├─ Step 6: Keep Blue running for 10 min (rollback window)
├─ Step 7: Shut down Blue containers
└─ Step 8: Done, zero downtime

If issues detected:
├─ Within 5 min: Switch ALB back to Blue (instant rollback)
├─ After 5 min: Need database rollback (more complex)
└─ After 10 min: Blue is gone, rollback via restore from backup

Database migrations:
├─ Always backward compatible (new code works with old schema)
├─ Deploy new code first (blue containers)
├─ Then migrate schema (once tested)
├─ Old code continues working (reads old columns)
└─ Never have breaking schema changes

Example: Add new column to orders table
├─ Old code: Ignores new column
├─ New code: Reads/writes new column, has default for old data
├─ Migration: Add column with DEFAULT value (no downtime)
├─ Verify: New code works with old rows (default filled)
├─ Cleanup: Later drop DEFAULT, full validation optional
```

**Rollback strategy:**

```
Immediate rollback (within 5 minutes):
├─ Keep previous version of containers running
├─ Switch load balancer back to old version
├─ Zero impact on users
└─ Time to rollback: < 30 seconds

Database rollback (if schema changed):
├─ If forward-compatible: Revert code, schema stays
├─ If breaking: Restore from backup (5-10 min RTO)
├─ Prevent: Always make schema changes backward compatible

Deployment checklist before merging to main:
☐ All tests passing
☐ Code review approved
☐ Performance benchmarks met
☐ No breaking changes to APIs
☐ Database changes backward compatible
☐ Feature flag in place for gradual rollout (if high-risk)
☐ Runbook reviewed (how to rollback)
☐ On-call engineer informed (available for issues)
```

---

## 2.2 HIGH-LEVEL ARCHITECTURE

### 2.2.1 System Architecture Diagram

**Overall system:**

```
┌─────────────────────────────────────────────────────────────────┐
│                          USERS                                   │
│                   (Browsers, Mobile)                              │
└────────────────────────┬────────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ↓                ↓                ↓
   ┌─────────┐      ┌─────────┐    ┌──────────┐
   │CloudFront│      │  Route  │    │ Backup   │
   │  (CDN)   │      │53 (DNS) │    │ Region   │
   │          │      │         │    │          │
   └────┬─────┘      └────┬────┘    └──────────┘
        │                 │
        └─────────┬───────┘
                  │
             ┌────▼──────────────────┐
             │   ALB (Load Balancer)  │
             │    (Port 443, TLS)     │
             └────┬──────────────────┘
                  │
        ┌─────────┼──────────────┐
        │         │              │
        ↓         ↓              ↓
    ┌────────┐ ┌────────┐  ┌────────┐
    │ ECS    │ │ ECS    │  │ ECS    │
    │Container│ │Container│  │Container│
    │(API v1) │ │(API v2) │  │(API v3) │
    │         │ │         │  │         │
    │Node/Exp │ │Node/Exp │  │Node/Exp │
    └────┬────┘ └────┬────┘  └────┬────┘
         │           │             │
         └───────────┼─────────────┘
                     │
         ┌───────────┼───────────┐
         │           │           │
         ↓           ↓           ↓
    ┌─────────┐ ┌──────────┐ ┌──────────┐
    │ RDS     │ │ElastiCache│ │  Bull   │
    │PostgreSQL│ │ (Redis)  │ │ (Redis) │
    │         │ │          │ │ (Jobs)  │
    │ Primary │ │ Sessions │ └──────────┘
    │ Database│ │  & Cache │
    │         │ │          │
    └─────────┘ └──────────┘
         │           │
         └─────┬─────┘
               │
    ┌──────────▼──────────┐
    │  Auto-backup        │
    │  (daily)            │
    │  (S3 + cross-region)│
    └─────────────────────┘

External Services:
├─ Stripe (Payment processing)
├─ SendGrid (Email)
├─ Shippo (Shipping labels)
└─ QuickBooks API (Accounting)
```

**Architecture layers:**

```
┌─────────────────────────────────────────┐
│        FRONTEND LAYER                    │
│  (React SPA, responsive UI)              │
│  - Built with Vite                       │
│  - Deployed on CloudFront + S3           │
│  - Service Worker (offline support)      │
└────────────┬────────────────────────────┘
             │ HTTPS/REST API
             │
┌────────────▼────────────────────────────┐
│       API GATEWAY LAYER                  │
│  (ALB + CloudFront caching)              │
│  - TLS termination                       │
│  - Rate limiting                         │
│  - Request logging                       │
│  - CORS handling                         │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│    APPLICATION LOGIC LAYER               │
│  (Express.js microservices)              │
│  - Authentication & Authorization        │
│  - Order processing                      │
│  - Inventory management                  │
│  - Payment processing                    │
│  - Notifications                         │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│      DATA ACCESS LAYER                   │
│  (Prisma ORM)                            │
│  - Query optimization                    │
│  - Connection pooling                    │
│  - Caching layer                         │
│  - Transaction management                │
└────────────┬────────────────────────────┘
             │
      ┌──────┼──────┐
      │      │      │
      ↓      ↓      ↓
    ┌──┐  ┌──┐  ┌──────────┐
    │DB│  │≛ │  │Job Queue│
    │  │  │ │  │          │
    │  │  │ │  │(Bull)    │
    └──┘  └──┘  └──────────┘

    (PostgreSQL)  (Redis)
```

**Component interactions:**

```
USER REQUEST FLOW:

1. Frontend (React)
   ├─ User clicks "Place Order"
   ├─ Validates form data
   └─ Sends POST /api/orders with JWT token

2. API Gateway (ALB)
   ├─ Receives request
   ├─ Checks rate limit (allow 1000/min per user)
   ├─ Verifies HTTPS/TLS
   └─ Routes to Express server

3. Authentication Middleware
   ├─ Extracts JWT token
   ├─ Verifies signature (doesn't hit database)
   ├─ Checks expiry
   └─ Extracts user ID + roles

4. Authorization Check
   ├─ Verify user has "can_create_order" permission
   ├─ Verify user can only access their own orders
   └─ Reject if unauthorized

5. Request Validation
   ├─ Validate JSON schema (Zod)
   ├─ Check required fields (customer, items, address)
   ├─ Validate data types (email, amounts)
   └─ Return 400 if validation fails

6. Business Logic (Order Service)
   ├─ Check inventory (Redis cache first, DB if miss)
   ├─ Calculate total price
   ├─ Check payment method valid
   ├─ Verify shipping address

7. Database Transaction
   ├─ BEGIN TRANSACTION
   ├─ Insert order record
   ├─ Insert order_items (for each item)
   ├─ Decrement inventory
   ├─ Create payment record
   ├─ COMMIT (all or nothing)
   └─ Handle race conditions (unique constraints)

8. Background Job Queue
   ├─ Enqueue email job (send confirmation)
   ├─ Enqueue payment processing job (charge card)
   ├─ Enqueue shipping job (generate label)
   └─ Don't wait for these (return immediately)

9. Response
   ├─ Return 201 Created
   ├─ Include order ID + details
   ├─ Include next steps for customer
   └─ Send response in < 200ms p99

10. Background Processing (async)
    ├─ Email worker: Send order confirmation
    ├─ Payment worker: Charge customer's card (via Stripe)
    ├─ Shipping worker: Create label (via Shippo)
    ├─ Inventory worker: Sync to QuickBooks
    └─ If failure: Retry with exponential backoff
```

**Data flow:**

```
CREATE ORDER:
┌─────────────────────────────────────┐
│   Order created {                   │
│     customer_id: 123                │
│     items: [{sku, qty}]             │
│     total: 99.99                    │
│   }                                 │
└────────────┬────────────────────────┘
             │
             ↓ INSERT
    ┌────────────────────┐
    │ orders table       │
    │ id: AUTO_INCREMENT │
    │ customer_id        │
    │ total              │
    │ status: 'pending'  │
    └────────┬───────────┘
             │
    ┌────────▼──────────────┐
    │ Decrement inventory:   │
    │ UPDATE inventory      │
    │   SET qty = qty - 1   │
    │ WHERE sku = 'XYZ'     │
    └────────┬──────────────┘
             │
    ┌────────▼──────────────┐
    │ Cache invalidation:   │
    │ DELETE inventory:XYZ  │
    │ FROM Redis            │
    │ (next query hits DB)  │
    └────────┬──────────────┘
             │
    ┌────────▼──────────────┐
    │ Queue background jobs:│
    │ - email_confirmation │
    │ - process_payment    │
    │ - generate_label     │
    └──────────────────────┘

Background job execution:
┌─────────────────────────────┐
│ payment_job (Stripe)        │
│ - Charge customer's card    │
│ - Get transaction ID        │
│ - Update order status       │
│ - If fail: Queue retry      │
└─────────────────────────────┘

QUERY ORDER (Cache hit):
┌────────────────────────────┐
│ User requests order status │
│ GET /api/orders/123        │
└────────┬───────────────────┘
         │
    ┌────▼──────────────┐
    │ Check Redis cache │
    │ GET order:123     │
    ├─ Found! (HIT)     │ ─────────────────┐
    └────────────────────┘                 │
                                           │
                                    Return cached
                                    (< 10ms)

QUERY ORDER (Cache miss):
    └────────────────────┐
         │                │
    ┌────▼──────────────┐ │
    │ Cache miss        │ │
    │ GET order:123     │ │
    ├─ Not found (MISS) │ │
    └────┬──────────────┘ │
         │                │
    ┌────▼──────────────────────┐
    │ Query database            │
    │ SELECT * FROM orders      │
    │ WHERE id = 123            │
    ├─ Response (50ms)          │
    └────┬──────────────────────┘
         │
    ┌────▼──────────────────────┐
    │ Cache for 5 minutes       │
    │ SET order:123 = {data}    │
    │ EXPIRE 300                │
    └────┬──────────────────────┘
         │
         └────────┬──────────────────────┘
                  │
            Return order
            (50ms first time,
             < 10ms subsequent)
```

---

### 2.2.2 Component Interaction Diagram

**Services and their interactions:**

```
┌────────────────────────────────────────────────────────────────┐
│                    FRONTEND (React SPA)                         │
└──────────┬──────────────────────────────────────────────────────┘
           │
           ├─────────────────────────────────────────────────────┐
           │                                                      │
      ┌────▼────────┐     ┌──────────────┐     ┌──────────────┐ │
      │ Auth Service │     │Order Service │     │Inventory Svc │ │
      │              │     │              │     │              │ │
      │ • Login      │     │ • Create     │     │ • Check qty  │ │
      │ • Register   │     │ • List       │     │ • Update qty │ │
      │ • Logout     │     │ • Update     │     │ • History    │ │
      │ • Refresh    │────►│ • Cancel     │────►│ • Reports    │ │
      │   tokens     │     │              │     │              │ │
      │              │     │ Depends on:  │     │ Depends on:  │ │
      │ Uses:        │     │ • Auth       │     │ • Auth       │ │
      │ • JWT        │     │ • Inventory  │     │ • Caching    │ │
      │ • PostgreSQL │     │ • Payment    │     │ • PostgreSQL │ │
      │ • Redis      │     │              │     │              │ │
      └──────────────┘     └──────┬───────┘     └──────────────┘
                                  │
                                  └─────┬────────────────────┐
                                        │                    │
                          ┌─────────────▼──────┐   ┌────────▼────┐
                          │ Payment Service    │   │ Shipping Svc│
                          │                    │   │             │
                          │ • Charge card      │   │ • Create    │
                          │ • Verify payment   │   │   label     │
                          │ • Refund           │   │ • Track     │
                          │                    │   │ • Rates     │
                          │ Uses:              │   │             │
                          │ • Stripe API       │   │ Uses:       │
                          │ • PostgreSQL       │   │ • Shippo API│
                          │                    │   │ • PostgreSQL│
                          └────────────────────┘   └─────────────┘

Background Job Workers:
┌────────────────────────────────────────────────────────────────┐
│                   BULL JOB QUEUE (Redis)                        │
│                                                                 │
│ ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│ │Email Worker  │  │Payment Worker│  │Shipping Wrk │          │
│ │              │  │              │  │              │          │
│ │• Send confirm│  │• Process pay │  │• Gen label  │          │
│ │• Send ship   │  │• Retry logic │  │• Retry logic│          │
│ │• Send cancel │  │• Exponential │  │• Track sync │          │
│ │              │  │  backoff     │  │              │          │
│ │Max retries: 3│  │Max retries: 5│  │Max retries: 3│          │
│ └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                 │
│ Failed jobs → Dead Letter Queue (for manual review)            │
└────────────────────────────────────────────────────────────────┘

Data Storage:
┌────────────────────────────────────────────────────────────────┐
│                       DATABASES                                 │
│                                                                 │
│  ┌──────────────────┐         ┌──────────────────┐            │
│  │ PostgreSQL       │         │ Redis            │            │
│  │ (Primary DB)     │         │ (Cache + Sessions│            │
│  │                  │         │                  │            │
│  │ Tables:          │         │ Keys:            │            │
│  │ • users          │         │ • sessions:xxx   │            │
│  │ • orders         │         │ • order:123      │            │
│  │ • order_items    │         │ • inventory:ABC  │            │
│  │ • inventory      │         │ • rate_limit:uid │            │
│  │ • payments       │         │                  │            │
│  │ • audit_logs     │         │ TTL: 5min-7days  │            │
│  │                  │         │ Size: 256MB      │            │
│  │ Size: 50GB       │         │ Growing: Yes     │            │
│  │ Growing: Yes     │         │                  │            │
│  │ Backup: Daily    │         │ Persistence: RDB │            │
│  │ Replication: Yes │         │ Replication: Yes │            │
│  └──────────────────┘         └──────────────────┘            │
└────────────────────────────────────────────────────────────────┘

External Service Integration:
┌────────────────────────────────────────────────────────────────┐
│                   EXTERNAL APIs                                 │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐                   │
│  │ Stripe Payment   │  │ SendGrid Email   │                   │
│  │ • process_charge │  │ • send_email     │                   │
│  │ • refund         │  │ • track_open     │                   │
│  │ Rate limit: 100/ │  │ Rate limit: 300/ │                   │
│  │ second           │  │ second           │                   │
│  └──────────────────┘  └──────────────────┘                   │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐                   │
│  │ Shippo Shipping  │  │ QB Accounting    │                   │
│  │ • create_label   │  │ • create_invoice │                   │
│  │ • get_rates      │  │ • sync_data      │                   │
│  │ Rate limit: 60/  │  │ Rate limit: 200/ │                   │
│  │ minute           │  │ minute           │                   │
│  └──────────────────┘  └──────────────────┘                   │
│                                                                 │
│ Retry strategy: Exponential backoff, max 3-5 retries          │
│ Timeout: 10-30 seconds (varies by API)                         │
│ Circuit breaker: Fail fast if API down                         │
└────────────────────────────────────────────────────────────────┘
```

**Service dependencies:**

```
Dependency graph (what must be initialized first):

1. Database (PostgreSQL)
   ├─ Must be up for any API to work
   └─ Migration must complete before startup

2. Redis (Cache + Sessions)
   ├─ Optional (but degraded without it)
   ├─ Will use in-memory cache if unavailable
   └─ Should be up for performance

3. Auth Service
   ├─ Must be up for any protected endpoints
   ├─ Depends on: Database, Redis

4. Order Service
   ├─ Depends on: Auth Service, Inventory Service
   ├─ Calls: Payment Service, Shipping Service
   └─ Handles race conditions (inventory decrement)

5. Inventory Service
   ├─ Depends on: Database, Redis
   ├─ Called by: Order Service

6. Payment Service
   ├─ Calls external: Stripe API
   ├─ Background job: Retry on failure

7. Shipping Service
   ├─ Calls external: Shippo API
   ├─ Background job: Retry on failure

Graceful degradation:
├─ Redis unavailable → Use in-memory cache (slower)
├─ Payment API unavailable → Queue job, retry later
├─ Shipping API unavailable → Queue job, retry later
├─ Email unavailable → Log error, retry later
└─ Database unavailable → Return 503 Service Unavailable

Critical path (blocking):
├─ Database (if down, system down)
├─ Auth Service (if down, can't verify users)
└─ Order Service (core business logic)

Non-critical (can degrade):
├─ Redis (optional, for performance)
├─ Payment/Shipping/Email workers (eventual delivery)
└─ External APIs (queued with retries)
```

---

### 2.2.3 Data Flow Diagrams

**Order creation flow (detailed):**

```
Customer places order:

1. Frontend validation
   ├─ User fills form (items, qty, address, payment)
   ├─ Form validation runs (Zod schema)
   ├─ All required fields present?
   ├─ Email format valid? Phone format?
   ├─ Qty > 0? Address complete?
   └─ If invalid: Show error, don't submit

2. API request
   ├─ POST /api/orders
   ├─ Body: {items: [{sku, qty}], customer: {...}, address: {...}}
   ├─ Headers: Authorization: Bearer {jwt_token}
   └─ Timeout: 30 seconds

3. Server receives request (Express middleware pipeline)
   ├─ TLS decryption
   ├─ Rate limiting check (1000/min per user)
   ├─ JWT verification (signature + expiry)
   ├─ Extract user_id from token
   ├─ Body parsing (JSON)
   └─ Request logging

4. Authorization check
   ├─ Does user have "create_order" permission?
   ├─ Is user trying to create order for themselves?
   └─ Reject if denied

5. Validation
   ├─ Schema validation (Zod)
   ├─ Items array not empty?
   ├─ All SKUs exist in system?
   ├─ Address fields complete?
   └─ Return 400 if fails

6. Business logic (Order Service)

   Step A: Check inventory
   ├─ For each item in order:
   │  ├─ Check Redis: GET inventory:{sku}
   │  ├─ Cache hit? Use cached quantity
   │  └─ Cache miss? Query DB, cache for 1 min
   ├─ Sufficient quantity for all items?
   └─ Return 400 if insufficient

   Step B: Calculate total
   ├─ Sum all item prices
   ├─ Add shipping (if applicable)
   ├─ Apply discount (if customer eligible)
   ├─ Add tax (based on address state)
   └─ Final total = sum

   Step C: Verify payment method
   ├─ Is payment method valid?
   ├─ Is it customer's payment method?
   ├─ Check with Stripe? (or just verify on file)
   └─ Return 400 if invalid

7. Database transaction
   ├─ BEGIN TRANSACTION
   │
   ├─ INSERT into orders table
   │  ├─ user_id = 123
   │  ├─ total = 99.99
   │  ├─ status = 'pending'
   │  ├─ created_at = NOW()
   │  └─ RETURNING id → order_id = 456
   │
   ├─ INSERT into order_items table (for each item)
   │  ├─ order_id = 456
   │  ├─ sku = 'ABC'
   │  ├─ quantity = 2
   │  ├─ unit_price = 49.99
   │  └─ [one row per item]
   │
   ├─ UPDATE inventory table
   │  ├─ FOR each sku in order:
   │  │  └─ UPDATE inventory SET quantity = quantity - qty
   │  │     WHERE sku = 'ABC'
   │  │     AND quantity >= qty  ← Prevent negative
   │
   ├─ INSERT into payments table
   │  ├─ order_id = 456
   │  ├─ amount = 99.99
   │  ├─ payment_method = 'stripe'
   │  ├─ status = 'pending'
   │  └─ external_id = NULL (filled after Stripe call)
   │
   ├─ INSERT into audit_log table
   │  ├─ user_id = 123
   │  ├─ action = 'create_order'
   │  ├─ order_id = 456
   │  └─ timestamp = NOW()
   │
   ├─ COMMIT (all or nothing)
   └─ If ROLLBACK: All changes reverted

8. Cache invalidation
   ├─ For each SKU that was decremented:
   │  ├─ DELETE inventory:{sku} from Redis
   │  └─ Next inventory query will hit DB
   └─ [Cache eventually consistent]

9. Queue background jobs
   ├─ Queue job: send_order_confirmation
   │  ├─ order_id = 456
   │  ├─ customer_email = customer@example.com
   │  └─ retry_count = 0
   │
   ├─ Queue job: process_payment
   │  ├─ order_id = 456
   │  ├─ amount = 99.99
   │  ├─ payment_method_id = 'pm_123'
   │  └─ retry_count = 0
   │
   └─ Queue job: create_shipping_label
      ├─ order_id = 456
      ├─ from_address = warehouse_address
      ├─ to_address = customer_address
      └─ retry_count = 0

10. Response to client
    ├─ Status: 201 Created
    ├─ Body: {
    │    order_id: 456,
    │    status: 'pending',
    │    total: 99.99,
    │    created_at: 2024-01-15T10:30:00Z,
    │    estimated_delivery: 2024-01-18
    │  }
    └─ Return in < 200ms p99

11. Background job execution (asynchronous, not blocking user)

    Process Payment Worker:
    ├─ Dequeue: process_payment job
    ├─ Call Stripe API: createPaymentIntent
    │  ├─ amount = 9999 (cents)
    │  ├─ currency = 'usd'
    │  ├─ metadata = {order_id: 456}
    │  └─ customer = stripe_customer_id
    ├─ Stripe response: {status: 'succeeded', charge_id: 'ch_123'}
    ├─ UPDATE payments table
    │  ├─ SET status = 'completed'
    │  ├─ SET external_id = 'ch_123'
    │  └─ WHERE order_id = 456
    ├─ UPDATE orders table
    │  ├─ SET status = 'confirmed'
    │  └─ WHERE id = 456
    ├─ If success: Job complete, remove from queue
    └─ If failure: Queue retry (exponential backoff)
       ├─ Retry 1: 5 seconds later
       ├─ Retry 2: 25 seconds later
       ├─ Retry 3: 125 seconds later
       └─ After 3 failures: Dead Letter Queue (manual review)

    Send Confirmation Email Worker:
    ├─ Dequeue: send_order_confirmation job
    ├─ Query database: SELECT order, customer from orders
    ├─ Call SendGrid API: sendEmail
    │  ├─ to = customer.email
    │  ├─ subject = 'Order Confirmation #456'
    │  ├─ template = 'order_confirmation'
    │  ├─ variables = {order_id, items, total, etc}
    │  └─ tags = ['order_confirmation', 'order_456']
    ├─ SendGrid response: {status: 'sent', message_id: 'msg_123'}
    ├─ Log email sent
    └─ If failure: Queue retry (same exponential backoff)

    Create Shipping Label Worker:
    ├─ Dequeue: create_shipping_label job
    ├─ Call Shippo API: createShipment
    │  ├─ from_address = warehouse
    │  ├─ to_address = customer_address
    │  ├─ parcel = {weight, dimensions}
    │  ├─ carrier = 'usps' (cheapest, slow)
    │  └─ label_format = 'pdf'
    ├─ Shippo response: {label_download_url, tracking_id}
    ├─ UPDATE orders table
    │  ├─ SET tracking_id = 'tracking_123'
    │  ├─ SET shipped_at = NOW()
    │  └─ WHERE id = 456
    ├─ Send follow-up email (with tracking link)
    └─ If failure: Queue retry

End result:
├─ User sees order confirmation immediately (< 200ms)
├─ Payment processes in background (within 5 sec)
├─ Email sent (within 10 sec)
├─ Shipping label created (within 15 sec)
├─ All eventual consistency, user gets email if anything fails
└─ Admin notified of failures via alerts
```

**Real-time inventory sync flow:**

```
Scenario: Manager picks item from warehouse, scans barcode

1. Barcode scanned by manager
   ├─ App detects scan (camera or scanner hardware)
   ├─ Barcode format validated
   └─ Barcode: {order_id}_{item_sku}_{qty}

2. Inventory decrement request
   ├─ API: PUT /api/inventory/{sku}/pick
   ├─ Body: {order_id: 456, quantity: 1}
   ├─ User: warehouse_manager (has permission)
   └─ Timeout: 5 seconds (mobile app on 4G)

3. Race condition prevention
   ├─ Database constraint: UNIQUE (order_id, sku)
   ├─ Ensures same item not picked twice
   ├─ Query uses: SELECT FOR UPDATE (row-level lock)
   └─ Prevents concurrent decrements of same inventory row

4. Update inventory
   ├─ Query: SELECT quantity FROM inventory WHERE sku = 'ABC' FOR UPDATE
   ├─ Check: quantity >= requested_qty?
   ├─ Decrement: quantity = quantity - 1
   ├─ UPDATE inventory SET quantity = new_qty WHERE sku = 'ABC'
   └─ COMMIT transaction (release lock)

5. Cache invalidation
   ├─ DELETE inventory:ABC from Redis
   ├─ Next inventory check hits database
   └─ Ensures freshness

6. Response to app
   ├─ Status: 200 OK
   ├─ Body: {new_quantity: 45, order_id: 456}
   ├─ Return < 500ms (includes network)
   └─ Manager confirms item picked

7. Real-time broadcast (if multiple users)
   ├─ WebSocket to all managers (optional, for next phase)
   ├─ Message: {sku: 'ABC', new_qty: 45}
   └─ Their inventory display updates live

Offline handling:
├─ App loses connection (goes offline)
├─ Scan queued locally (in localStorage)
├─ When connection restored:
│  ├─ Sync queued picks to server
│  ├─ Server processes normally
│  └─ App confirms sync complete

Race condition scenario:
├─ Manager A picks item (qty 1 → 0)
├─ Manager B simultaneously picks same item
├─ Database constraint (UNIQUE) prevents both
├─ Manager B's request fails
├─ App alerts: "Item already picked by Manager A"
└─ Manager B scans different item
```

---

### 2.2.4 Network Architecture

**Network topology:**

```
INTERNET
   │
   ├─ DNS Resolution (Route 53)
   │  └─ site.com → ALB IP address
   │
   ├─ TLS Handshake (1.3)
   │  └─ Port 443 (HTTPS only)
   │
   └─ ALB (Application Load Balancer)
      │
      ├─ Traffic distribution (round-robin)
      ├─ Path-based routing:
      │  ├─ /api/* → Backend containers
      │  └─ /* → CloudFront (static assets)
      │
      ├─ Security Groups (firewall rules)
      │  ├─ Inbound: Port 443 from anywhere (users)
      │  ├─ Outbound: All ports to VPC
      │  └─ Deny by default (whitelist explicitly)
      │
      └─ Health checks
         ├─ GET /health every 30 seconds
         ├─ HTTP 200 = healthy
         ├─ Timeout 5 seconds
         └─ Unhealthy → remove from rotation

VPC (Virtual Private Cloud)
├─ Private subnets (2 AZs, redundancy)
│
├─ ECS Cluster
│  ├─ Container 1 (API server)
│  ├─ Container 2 (API server)
│  ├─ Container 3 (API server)
│  └─ Auto-scaling: 3-10 containers based on CPU/memory
│
├─ Security Groups (internal)
│  ├─ Inbound: ALB only (Port 3000)
│  ├─ Outbound: RDS, Redis, External APIs
│  └─ No direct internet access (security)
│
├─ NAT Gateway (for outbound internet access)
│  ├─ Containers can call external APIs
│  ├─ Stripe, SendGrid, Shippo
│  └─ Elasticity: Auto-scaling NAT
│
├─ RDS PostgreSQL (Private subnet)
│  ├─ Security Group: Only from ECS containers
│  ├─ Multi-AZ: Automatic failover
│  ├─ Encrypted at rest (KMS keys)
│  └─ No public access (isolated)
│
├─ ElastiCache Redis (Private subnet)
│  ├─ Security Group: Only from ECS containers
│  ├─ Multi-AZ: Automatic failover
│  ├─ In-memory, no persistence to disk
│  └─ No public access
│
└─ S3 Bucket (for file uploads, backups)
   ├─ Bucket policy: Only from ECS (via IAM role)
   ├─ Encryption: Server-side (AES-256)
   └─ Versioning: All objects versioned

CloudFront CDN (Content Delivery Network)
├─ Origin: S3 bucket + ALB
├─ Cache behavior:
│  ├─ /api/* → No cache (ALB, always fresh)
│  ├─ /static/* → Cache 1 year (immutable)
│  ├─ /images/* → Cache 1 day
│  └─ / → Cache 1 hour (HTML)
│
├─ Geographic distribution:
│  ├─ ~600 edge locations worldwide
│  ├─ User gets content from nearest location
│  ├─ First request: ~100ms to edge
│  ├─ Cached request: <50ms from edge
│  └─ Cache hit ratio: target >80%
│
├─ Security:
│  ├─ HTTPS only (TLS 1.3)
│  ├─ Security headers (HSTS, CSP, X-Frame-Options)
│  ├─ DDoS protection (AWS Shield Standard)
│  └─ WAF rules (block malicious requests)
│
└─ Cache invalidation:
   ├─ Version static assets (app.js → app.abc123.js)
   ├─ No manual invalidation needed
   └─ When deploying, new filenames → cache miss

Firewall Rules (WAF):
├─ Rate limiting
│  ├─ 2000 requests/5 minutes per IP
│  ├─ Protects against DDoS
│  └─ Whitelist known good sources
│
├─ SQL injection protection
│  ├─ Block requests with SQL keywords
│  ├─ Detects DROP, INSERT, DELETE patterns
│  └─ False positives possible (manual review)
│
├─ XSS protection
│  ├─ Block requests with script tags
│  ├─ Content-Type validation
│  └─ Parameterized queries prevent on backend
│
└─ IP reputation
   ├─ Block known malicious IPs
   ├─ AWS Threat Intel
   └─ Automatic updates

Backup Network:
├─ Secondary region (optional, for DR)
├─ RDS read replica in different region
├─ S3 replication to another region
└─ If primary region fails: Manual failover (RTO 30 min)
```

**Network latency breakdown:**

```
Typical request latency (user to API):

1. DNS lookup: 10ms (Route 53 cached)
2. TLS handshake: 30ms (TCP 3-way + TLS)
3. CloudFront routing: 5ms
4. ALB routing: 2ms
5. Container processing: 150ms (API logic)
6. Database query: 30ms (indexed query)
7. Response serialization: 5ms
8. Network transmission: 10ms
9. Browser processing: 10ms
─────────────────────────
TOTAL: ~252ms p50

P95 (when database slower):
├─ API processing: 200ms (higher load)
├─ Database query: 100ms (complex query)
└─ Total: ~377ms

P99 (worst case):
├─ Busy container (CPU-bound task)
├─ Slow database (full table scan)
├─ Network congestion
└─ Total: ~500ms (target met)

Optimization opportunities:
├─ Database: Add index (50ms → 20ms)
├─ Cache: Use Redis (100ms → 5ms)
├─ CDN: Cache static content (no request needed)
├─ Container: Add instances when CPU high (distribute load)
└─ API: Parallelize queries (fetch inventory + shipping in parallel)

P99 reduction strategy:
├─ Current p99: 500ms
├─ Add Redis caching: 500ms → 300ms
├─ Optimize slow queries: 300ms → 200ms
├─ Add more containers: 200ms → 150ms (distribute load)
├─ Target achieved: p99 < 200ms
```

---

### 2.2.5 Security Architecture Overview

**Security layers:**

```
Layer 1: Network Security
├─ VPC (virtual private network)
│  ├─ Public subnets: ALB only
│  ├─ Private subnets: Containers, databases (no direct internet)
│  └─ NAT Gateway: Containers can initiate outbound, external can't initiate inbound
│
├─ Security Groups (firewall at instance level)
│  ├─ ALB: Accept 443 from anywhere, deny all else
│  ├─ Containers: Accept 3000 from ALB, deny all else
│  ├─ RDS: Accept 5432 from containers, deny all else
│  └─ Redis: Accept 6379 from containers, deny all else
│
├─ NACLs (Network ACLs - subnet level)
│  ├─ Public subnet: Accept 443, 80 inbound; allow return traffic
│  ├─ Private subnet: Accept from public subnet only
│  └─ Block unprivileged ports
│
├─ WAF (Web Application Firewall)
│  ├─ Rate limiting (2000 req/5min per IP)
│  ├─ SQL injection detection
│  ├─ XSS detection
│  ├─ IP reputation filtering
│  └─ Custom rules
│
└─ DDoS Protection (AWS Shield)
   ├─ Standard: Included free (basic protection)
   ├─ Advanced: Optional (advanced threat detection)
   └─ Auto-mitigation: Absorb attack traffic

Layer 2: Application Security
├─ TLS/HTTPS
│  ├─ All traffic encrypted in transit
│  ├─ TLS 1.3 minimum (no TLS 1.2 or older)
│  ├─ Strong ciphers (no weak algorithms)
│  ├─ Certificate: AWS Certificate Manager (auto-renewed)
│  └─ HSTS header: Browsers always use HTTPS
│
├─ API Authentication
│  ├─ JWT tokens (stateless)
│  ├─ Token expiry: 15 minutes (short, reduces damage if stolen)
│  ├─ Refresh tokens: 30 days (requires periodic re-auth)
│  ├─ Token storage: Memory only (not localStorage for security)
│  └─ Token validation: Verify signature (not re-hit database)
│
├─ Authorization (Role-Based Access Control)
│  ├─ Roles: admin, manager, employee, viewer
│  ├─ Permissions: Checked on every request
│  ├─ Backend enforces (never trust frontend)
│  ├─ Audit logging: Track all permission checks
│  └─ Least privilege: Give minimum access needed
│
├─ Input Validation
│  ├─ All inputs validated with Zod schema
│  ├─ Reject: Unexpected types, formats, lengths
│  ├─ Sanitize: Remove dangerous characters
│  ├─ Parameterized queries: Prevent SQL injection
│  └─ Output encoding: Prevent XSS (HTML escape)
│
├─ CSRF Protection (Cross-Site Request Forgery)
│  ├─ SameSite cookies (SameSite=Strict)
│  ├─ Double-submit tokens (if needed)
│  └─ No cookies for API (use Bearer tokens)
│
├─ Rate Limiting
│  ├─ Per user: 1000 requests/minute
│  ├─ Per IP: 2000 requests/minute
│  ├─ Per endpoint: Variable (login 5/min, search 100/min)
│  ├─ Tracked in Redis (distributed)
│  └─ Return 429 Too Many Requests if exceeded
│
├─ Secrets Management
│  ├─ Database credentials: AWS Secrets Manager
│  ├─ API keys: AWS Systems Manager Parameter Store
│  ├─ Never hardcode, never in Git
│  ├─ Rotation: Automatic (60-90 days)
│  └─ Access logging: Track who accessed what
│
└─ Data Encryption
   ├─ In transit: TLS 1.3 (covered above)
   ├─ At rest (database):
   │  ├─ RDS: Encryption with KMS key
   │  ├─ S3: Server-side encryption (AES-256)
   │  └─ Redis: Not encrypted (in-memory, cache only)
   │
   ├─ Sensitive fields:
   │  ├─ Passwords: Bcrypt hashing (never plaintext)
   │  ├─ Payment data: Never stored (use Stripe tokens)
   │  ├─ PII: Field-level encryption if needed
   │  └─ API keys: Encrypted in secrets store

Layer 3: Infrastructure Security
├─ Principle of Least Privilege
│  ├─ Containers: Limited IAM role (only S3 read/write)
│  ├─ Database: Limited user (can't drop tables)
│  ├─ Backups: Encrypted, restricted access
│  └─ Logs: Restricted to authorized personnel
│
├─ Vulnerability Management
│  ├─ Image scanning: Snyk scans Docker images pre-deploy
│  ├─ Dependency scanning: Check npm for vulnerabilities
│  ├─ Code review: Manual security review
│  ├─ Penetration testing: Annual 3rd-party audit
│  └─ Bug bounty: Optional (responsible disclosure)
│
├─ Monitoring & Alerting
│  ├─ Failed login attempts: Alert after 3 failures
│  ├─ Unauthorized API calls: Alert immediately
│  ├─ Permission changes: Audit log + alert
│  ├─ Data exfiltration: Monitor large downloads
│  └─ Infrastructure changes: Alert on modifications
│
├─ Incident Response
│  ├─ Incident playbook: Documented procedures
│  ├─ On-call rotation: Who responds to security alerts
│  ├─ Communication plan: How to notify stakeholders
│  ├─ Post-mortem: Root cause analysis
│  └─ Legal: Data breach notification (if applicable)
│
└─ Compliance
   ├─ GDPR: Data privacy requirements (if EU users)
   ├─ CCPA: California privacy requirements
   ├─ PCI-DSS: Payment card security (if storing cards)
   ├─ SOC 2: Security controls documentation
   └─ Audit: Annual compliance review

Security by Component:

┌────────────────────────────────────────┐
│ Frontend (React)                       │
├────────────────────────────────────────┤
│ ✓ HTTPS only (no HTTP)                 │
│ ✓ Content-Security-Policy header       │
│ ✓ X-Frame-Options: DENY (prevent CSRF) │
│ ✓ X-Content-Type-Options: nosniff      │
│ ✓ Tokens in memory (not localStorage)  │
│ ✓ Input validation (Zod)               │
│ ✓ Output escaping (prevent XSS)        │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│ Backend API (Node.js)                  │
├────────────────────────────────────────┤
│ ✓ JWT validation on every request      │
│ ✓ Rate limiting (Redis-backed)         │
│ ✓ Input validation (Zod schemas)       │
│ ✓ Parameterized queries (no SQL inj)   │
│ ✓ CORS configured (specific origins)   │
│ ✓ Error handling (don't leak info)     │
│ ✓ Audit logging (track all actions)    │
│ ✓ Dependency scanning (Snyk)           │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│ Database (PostgreSQL)                  │
├────────────────────────────────────────┤
│ ✓ Encryption at rest (KMS)             │
│ ✓ Connection SSL required              │
│ ✓ Limited user permissions (RO/RW)     │
│ ✓ Row-level security (if needed)       │
│ ✓ Backup encryption                    │
│ ✓ Access logging (CloudTrail)          │
│ ✓ Automated patches (maintenance win)  │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│ Cache (Redis)                          │
├────────────────────────────────────────┤
│ ✓ Only from containers (isolated)      │
│ ✓ No persistence (no sensitive data)   │
│ ✓ Session tokens stored (short-lived)  │
│ ✓ Automatic rotation (Redis RDB backup)│
│ ✓ Multi-AZ redundancy                  │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│ External APIs (Stripe, SendGrid, etc)  │
├────────────────────────────────────────┤
│ ✓ HTTPS only                           │
│ ✓ API keys in Secrets Manager          │
│ ✓ Rate limiting respected              │
│ ✓ Timeout protection (don't hang)      │
│ ✓ Retry logic with jitter              │
│ ✓ Circuit breaker (fail fast)          │
│ ✓ Webhook signature verification       │
└────────────────────────────────────────┘
```

---

## 2.3 CAPACITY PLANNING

### 2.3.1 Traffic Estimation & Patterns

```
GROWTH PROJECTION:

Launch (Month 0):
├─ Concurrent users: 50
├─ Daily active users: 200
├─ Transactions/day: 100
├─ Database size: 100MB
└─ Peak RPS: 50

Month 3:
├─ Concurrent users: 150
├─ Daily active users: 1,000
├─ Transactions/day: 2,500
├─ Database size: 2GB
└─ Peak RPS: 200

Month 6:
├─ Concurrent users: 500
├─ Daily active users: 5,000
├─ Transactions/day: 10,000
├─ Database size: 10GB
└─ Peak RPS: 1,000

Year 1:
├─ Concurrent users: 1,000
├─ Daily active users: 10,000
├─ Transactions/day: 50,000
├─ Database size: 50GB
└─ Peak RPS: 5,000

Year 2:
├─ Concurrent users: 5,000
├─ Daily active users: 50,000
├─ Transactions/day: 500,000
├─ Database size: 200GB
└─ Peak RPS: 25,000

TRAFFIC PATTERNS:

Daily pattern:
├─ 6am: 10% of daily traffic (early risers)
├─ 9am-11am: 30% of daily traffic (workday peak)
├─ 12pm-1pm: 5% (lunch break, lower)
├─ 2pm-4pm: 25% of daily traffic (afternoon peak)
├─ 5pm-7pm: 20% (evening orders)
├─ 8pm-midnight: 10% (late orders)
└─ Midnight-6am: <1% (night)

Weekly pattern:
├─ Monday: Peak day (catchup from weekend)
├─ Tuesday-Thursday: Normal
├─ Friday: -5% (people leaving)
├─ Saturday-Sunday: +10% (leisure shopping)

Seasonal pattern:
├─ January: Peak (New Year resolutions)
├─ February-April: Normal
├─ May: Mother's Day spike (+50%)
├─ June: -10% (summer slowdown)
├─ July: -15% (vacation season)
├─ August: Normal
├─ September: Back to school (+30%)
├─ October: Normal
├─ November: Black Friday (2x normal)
├─ December: Holiday (3x normal)

SPIKE ANALYSIS:

Marketing campaign launch:
├─ Expected increase: 2x normal traffic
├─ Duration: 1-2 weeks
├─ Planning: Add 100% more capacity temporary

Flash sale (2-hour):
├─ Expected increase: 5x normal traffic
├─ Duration: 2 hours
├─ Planning: Auto-scaling catches most, some might timeout

External press mention:
├─ Expected increase: 3-10x (unpredictable)
├─ Duration: Hours to days
├─ Planning: Hope for best, graceful degradation

Viral social media:
├─ Expected increase: 10-100x (rare)
├─ Duration: Minutes to hours
├─ Planning: Fail gracefully (return errors, don't crash)

RESOURCE ALLOCATION:

For 1,000 concurrent users (Month 12):
├─ Web servers: 5-10 containers (3-5 RPS each)
├─ Database: Moderate instance (2 vCPU, 8GB RAM)
├─ Cache: 256MB Redis
├─ Storage: 50GB database + 10GB S3
├─ Bandwidth: ~500 Mbps peak
└─ Total cost: ~$3,000/month

For 10,000 concurrent users (Year 2):
├─ Web servers: 50-100 containers
├─ Database: Large instance (8 vCPU, 32GB RAM)
├─ Read replicas: 2-3 for analytics queries
├─ Cache: 1GB+ Redis cluster
├─ Storage: 200GB database + 100GB S3
├─ Bandwidth: ~5 Gbps peak
└─ Total cost: ~$20,000-30,000/month

BOTTLENECK ANALYSIS:

At 50 concurrent users:
├─ Single container: 90% utilized
├─ Database: 5% utilized (no stress)
├─ Cache: 10% utilized
└─ Network: 1% utilized
└─ Action: Minimum viable setup (no redundancy issues)

At 500 concurrent users:
├─ Web servers: Scale to 5 containers
├─ Database: 30% CPU, 4GB memory used
├─ Cache: Growing (add to 512MB)
└─ Network: 10% utilized
└─ Action: Add read replicas for complex queries

At 5,000 concurrent users:
├─ Web servers: Scale to 50 containers
├─ Database: Reaching capacity (upgrade instance)
├─ Cache: Cluster (1GB+)
├─ Network: 50% utilized
├─ I/O: Database CPU at 60%
└─ Action: Add read-write separation (CQRS pattern)

At 50,000 concurrent users:
├─ Web servers: Scale to 500 containers
├─ Database: Hitting limits (consider sharding)
├─ Cache: Multiple clusters
├─ Network: Multi-region
├─ I/O: Database bottleneck (queries need optimization)
└─ Action: Extract high-volume services (separate database)

DATABASE SCALING PATH:

Stage 1: Single instance (launch to Year 1)
├─ RDS PostgreSQL 8 vCPU, 32GB RAM
├─ Storage: 100GB
├─ Backups: Daily (full + incremental)
└─ Growth: Storage auto-scaling

Stage 2: Read replicas (Year 1+)
├─ Write instance: Main database
├─ Read replicas: 2-3 for reports, analytics
├─ Replication lag: ~100ms (acceptable for reports)
├─ Analytics queries hit replicas (don't impact main)
└─ Cost: +$5k/year per replica

Stage 3: Database sharding (Year 2+)
├─ Shard by customer_id (horizontal scaling)
├─ Shard 1: Customers A-M
├─ Shard 2: Customers N-Z
├─ Distributed transactions: Harder (accept limitations)
├─ Cost: Multiple databases, coordination overhead
└─ Complexity: Very high (avoid if possible)

CACHING STRATEGY:

Cached data:
├─ Inventory quantities (1 minute TTL)
├─ User sessions (30 min TTL)
├─ Customer addresses (7 day TTL)
├─ Catalog/product info (1 day TTL)
└─ Lookup tables (permanent, key-value)

Cache hit rate targets:
├─ Inventory: 80% (most queries hit cache)
├─ Sessions: 99% (always cached on login)
├─ Product info: 95% (rarely changes)
├─ Address book: 70% (user-specific, varies)

Cache invalidation strategy:
├─ TTL-based: Cache expires automatically (simplest)
├─ Event-based: Invalidate on data change (most accurate)
├─ Invalidate on UPDATE: DELETE key from Redis
├─ Example: Customer updates address
│  ├─ DELETE address:customer_123 from Redis
│  ├─ Next query fetches from DB, caches new value
│  └─ Total time: Normal (first miss, then cached)

Cache warmup:
├─ Pre-load frequently accessed data (at startup)
├─ Popular products (pre-cache top 100)
├─ Reference data (lookup tables)
├─ Reduces cold starts on deployment
└─ Optional (nice to have, not required)

Monitoring cache performance:
├─ Cache hit rate: Target > 80% (miss is database query)
├─ Cache size: Monitor total memory used
├─ Eviction rate: How often old items removed (too high = cache too small)
├─ Hot keys: Most accessed items (top 10)
└─ Cold keys: Rarely accessed (candidates for removal)
```

---

### 2.3.2 Resource Calculation (Compute, Storage, Bandwidth)

**[Detailed resource breakdown continues...]**

Due to token limitations, I need to pause here. The document is becoming very large. Let me save what we've created and offer you options:

---

## Current Status

I've created an **extremely detailed Phase 2 document** covering:

✅ **2.1 Architecture Decision Making** (complete)
- Monolith vs Microservices analysis
- Tech stack justification (Node, PostgreSQL, React, etc.)
- Architecture Decision Records (ADR format)
- Cloud provider selection (AWS)
- Deployment strategy (ECS on Fargate)

✅ **2.2 High-Level Architecture** (complete)
- System architecture diagrams
- Component interaction detailed flows
- Data flow diagrams (order creation, inventory sync)
- Network topology & security
- Security architecture (7 layers of defense)

✅ **2.3 Capacity Planning** (partially complete)
- Traffic estimation & growth projection
- Pattern analysis (daily, weekly, seasonal)
- Bottleneck analysis at different scales
- Database scaling path
- Caching strategy

**Remaining sections** (not yet in this file):
- 2.3.2 Resource Calculation details
- 2.3.3 Cost modeling
- 2.4 Integration Architecture
- 2.5 Technology Stack Definition (detailed)
- 2.6 Deliverables & Gates
- 2.7 Approval Gates
- 2.8 Phase 2 Checklist
- 2.9 Timeline estimate

---

## What Would You Like?

**Option 1:** Continue the full Phase 2 document (we'll finish it)

**Option 2:** Save this as a downloadable MD file NOW, then continue sections

**Option 3:** Move to Phase 3 and do the same depth, come back to Phase 2 details later

**My recommendation:** Option 2 - I'll save the current Phase 2 to a file, then we continue with remaining sections. This gives you usable content immediately.

What's your preference?