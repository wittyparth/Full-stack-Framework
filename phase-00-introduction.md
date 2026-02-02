# PHASE 0: PROJECT INITIATION

## Overview
This phase determines if the project should exist and establishes the foundation for all subsequent decisions. A poorly executed Phase 0 cascades failures through the entire project lifecycle.

***

## 0.1 PROJECT FOUNDATION

### 0.1.1 Stakeholder Identification & Mapping

**What to do:**
- Identify all decision-makers (who approves spending, timeline, scope)
- Identify all users (end-users, administrators, operators)
- Identify technical stakeholders (DevOps, Security, DBA teams)
- Map influence vs interest for each stakeholder
- Document approval chains (what decisions require whose sign-off)

**Critical details:**
- Stakeholders who can kill the project after it starts must be identified now
- Power dynamics matter (CEO wants feature X, but CFO controls budget)
- Technical stakeholders often have veto power over architecture decisions
- Document escalation path for blocking decisions

**Output:** Stakeholder map with roles, influence level, contact info, concerns

***

### 0.1.2 Business Objectives Definition

**What to do:**
- Why are we building this? (Revenue growth, cost reduction, user retention, market entry, competitive response)
- What problem does it solve? (Specific, measurable problem)
- Who benefits? (Which user segment, which stakeholder)
- What's the business impact? (Revenue increase, cost savings, time saved, risk reduction)
- Timeline pressure? (Nice to have vs must-have by date X)

**Critical details:**
- "We want an app" is not an objective. "We want to increase user retention by 15% in Q2 through real-time notifications" is
- Distinguish between wants and needs (stakeholder may want mobile-first, but need is "works on all devices")
- Document hidden agendas (political reasons for project)
- Understand if this is replacing existing system (migration complexity)
- Determine if this is greenfield or extending existing product

**Output:** Business objectives document (1-2 pages max, crystal clear)

***

### 0.1.3 Success Criteria Establishment

**What to do:**
- Define what "done" looks like (quantifiably measurable)
- Set KPIs (Key Performance Indicators) the project must hit
- Document ROI expectations (what are we spending vs what will we gain)
- Set quality baselines (uptime SLA, error rates, performance targets)
- Define user adoption targets (% of users who use feature, engagement metrics)

**Critical details:**
- Each success criterion must be measurable (not "fast" but "p99 latency < 200ms")
- Success criteria drive all architectural decisions downstream
- Budget constraints are success criteria (if you can only spend $X, architecture changes)
- Failure to define this causes scope creep (nothing is ever "done")
- Success criteria must be agreed by all stakeholders before Phase 1

**Examples:**
- "System must handle 10,000 concurrent users with p99 latency < 500ms"
- "Achieve 99.9% uptime SLA"
- "ROI breakeven within 18 months"
- "Achieve 40% user adoption within 6 months of launch"
- "Reduce customer support tickets by 30%"

**Output:** Success criteria document with measurable targets and who owns measurement

***

### 0.1.4 Budget & Timeline Constraints

**What to do:**
- Total budget allocated (hard cap, not estimate)
- Timeline hard deadline (if flexible, that's different strategy)
- Resource constraints (can't hire, must use existing team)
- Dependency constraints (must integrate with system X by date Y)
- Constraints on changes (can we pivot after starting, or is scope locked)

**Critical details:**
- Budget includes all costs (people, infrastructure, tools, external services)
- Timeline includes contingency? (80% allocated, 20% buffer is realistic)
- Fixed timeline + fixed scope + fixed budget = impossible (pick two)
- Understand what's truly fixed vs what's negotiable
- Cost per feature matters for trade-off decisions (feature A costs $X vs Y)
- External dependencies are schedule killers (if you depend on API from Partner Co that launches Q3, your timeline is constrained)

**Output:** Constraints document with clear negotiation points

***

### 0.1.5 Team Composition & Roles

**What to do:**
- Required roles (Frontend, Backend, DevOps, QA, Product Manager, Designer)
- Current team assessment (do we have people, what's their level)
- Gaps (what expertise is missing, need to hire or contract)
- Team experience with similar projects (have we done this before)
- Onboarding timeline (how long until new people are productive)
- Time allocation (is this 100% commitment or 20% side project)

**Critical details:**
- "Senior engineer" from Startup X is not same capability as "Senior engineer" from Google (assess specifically)
- Timeline includes ramp-up time for new team members (3-6 weeks to be productive)
- Team morale/turnover risk (if key person leaves mid-project, plan breaks)
- Communication overhead scales with team size (7 people = 21 communication channels)
- Remote vs co-located affects communication/decision-making speed
- Distributed team across timezones complicates synchronous work

**Output:** Team composition document with capability assessment and ramp-up plan

***

## 0.2 FEASIBILITY ANALYSIS

### 0.2.1 Technical Feasibility Assessment

**What to do:**
- Can we actually build this with available technology? (Is it technically possible)
- Do we have team capability to build it? (Can our people do this)
- Integration complexity (how many external systems must we integrate with)
- Legacy system constraints (are we forced to integrate with 15-year-old monolith)
- Proof of Concept needs (should we build a spike/prototype first)
- Technology decisions (if we choose tech X instead of Y, does it become feasible)

**Critical details:**
- "Feasible" is not the same as "easy" (we can build anything, question is cost/time/quality)
- Unknown unknowns kill timelines (if you've never done distributed caching before, plan 2-3 weeks to learn)
- Integration with external APIs/systems is often the bottleneck (their API sucks, they don't support what we need, rate limits)
- Database performance at scale is often underestimated (works fine with 1000 rows, breaks at 100 million)
- Security/compliance requirements often make "simple solution" impossible (e.g., "must be GDPR compliant" means data residency, deletion policies, audit trails all built-in)

**Questions to ask:**
- Have we built something like this before?
- What's the riskiest technical component (database queries, real-time sync, machine learning model accuracy)?
- If we can't use tech X (because team doesn't know it), can we use tech Y?
- What's the minimum viable integration (can we launch without some integrations)?

**Output:** Technical feasibility assessment with risk flags

***

### 0.2.2 Market & Competitive Analysis

**What to do:**
- Does a market exist for this? (Are people actually willing to pay/use this)
- What competitors exist? (How are we different, better, worse)
- Customer discovery (talk to 10-20 potential users, not hypothetical)
- Feature comparison (competitors have X, Y, Z; we're planning A, B, C - is that competitive)
- Market timing (is market ready, or are we early/late)
- Pricing comparison (what can we charge vs what competitors charge)

**Critical details:**
- "We asked internal stakeholders" is not market research (they're biased)
- Talk to actual target users outside your organization
- Competitors offer signal (if competitors exist, market exists; if no competitors, might be no market)
- Features competitors have but you're not building are either (a) not valuable or (b) a gap you'll lose on
- Market saturation affects pricing and customer acquisition cost

**Output:** Market analysis document with competitive positioning

***

### 0.2.3 Risk Identification Matrix

**What to do:**
- What could go wrong? (Think big: key person leaves, technology choice fails, market rejects product, integration takes 6 months instead of 2 weeks)
- How likely is each risk? (High/Medium/Low probability)
- What's the impact if it happens? (Timeline slip of 2 weeks vs project kills vs company goes bankrupt)
- Risk = Probability × Impact (prioritize addressing high-probability + high-impact risks)
- Mitigation strategy (what can we do now to reduce probability or impact)

**Common risks in software projects:**
- **Team risk**: Key person departure, skill gaps, communication issues
- **Technical risk**: Technology choice doesn't scale, integration complexity underestimated, security/performance assumptions prove wrong
- **Timeline risk**: Scope creep, external dependencies slip, team onboarding takes longer than planned
- **Market risk**: User adoption lower than expected, competitive response faster than expected, requirements change post-launch
- **External risk**: Third-party API changes, cloud provider outage, regulatory changes
- **Dependency risk**: Partner company delivers their piece late, external API becomes unavailable, infrastructure provider capacity issues

**Mitigation examples:**
- Key person risk → Document critical knowledge, cross-train, have backup plan
- Technology risk → Proof of concept before full commitment, avoid unproven tech
- Timeline risk → 20% contingency buffer, start Phase 2 architecture early, have fallback options
- External dependency → Start conversations early, have fallback solution, don't let critical path depend on external group

**Output:** Risk register with probability, impact, mitigation for each risk

***

### 0.2.4 Resource Availability Check

**What to do:**
- Do required people exist in organization? (Can we dedicate Frontend engineer 100% for 6 months)
- External resource needs? (Do we need to hire, contract, or outsource any component)
- Equipment/tools/infrastructure? (Do we have cloud budget, development licenses, monitoring tools)
- Opportunity cost (what project are we pulling people from, what's the impact)
- Competing projects (if company has 3 projects starting simultaneously, we can't staff all equally)

**Critical details:**
- "We'll hire" adds 6-12 weeks onboarding before they're productive
- Contractors ramp slower than employees (code review overhead, cultural onboarding)
- Tools/infrastructure have procurement time (can't get enterprise contract in 2 days)
- External resources (cloud, APIs) have costs that compound
- Resource allocation reveals project priority (are we really committed, or is this low priority)

**Output:** Resource plan with availability, hiring timeline, costs

***

### 0.2.5 Go/No-Go Decision

**What to do:**
- Based on 0.1-0.2.4, is this project a "Go" or "No-Go"?
- If Go: what assumptions must hold true (risk register items we can't afford to happen)
- If No-Go: why not, and what would need to change to make it viable
- Document decision maker and approval
- If Go, what's the success bar for Phase 1 (do we re-evaluate mid-Phase 1)

**Critical details:**
- This is a formal gate, not consensus (someone decides, others accept
- Go with conditions is valid ("Go if we can hire 2 engineers by Q2, else No-Go")
- Changing this decision mid-project is expensive (sunk cost fallacy kills projects)
- If technical infeasibility is discovered, Go → No-Go is acceptable

**Output:** Go/No-Go decision document signed by authority

***

## 0.3 DELIVERABLES & GATES

### Deliverable 1: Project Charter
**Contains:**
- Business objective (1 paragraph, clear)
- Success criteria (3-5 measurable targets)
- Team composition (roles, names, capability level)
- Budget and timeline (fixed numbers)
- High-level risks (top 5-10)
- Constraints (technical, business, resource)
- Go/No-Go decision

**Quality check:** Can a new person read this and understand what project is trying to do?

***

### Deliverable 2: Feasibility Report
**Contains:**
- Technical feasibility assessment (is it buildable)
- Proof of concept decisions (do we need to spike anything)
- Market analysis (competitive position, customer need validation)
- Risk register (probability, impact, mitigation)
- Resource availability (can we staff this)
- Assumptions (what must be true for success)

**Quality check:** Would you fund this project based on this report?

***

### Deliverable 3: Stakeholder & Constraint Document
**Contains:**
- Stakeholder map (who decides what, approval chains)
- Budget breakdown (where does the money go)
- Timeline constraints (hard dates, dependencies)
- Technical constraints (legacy systems, must-haves, integration requirements)
- Non-negotiable requirements (what cannot be cut)

**Quality check:** Can engineering team see what they're working within?

***

## 0.4 APPROVAL GATES

### Gate 1: Business Sponsorship Sign-Off
**Criteria:**
- Executive sponsor agrees with success criteria
- Budget is formally approved
- Timeline is realistic per sponsor
- Resource commitment is confirmed (people allocated)

**Who approves:** Business sponsor, CFO (budget), CEO (strategic alignment)

***

### Gate 2: Technical Feasibility Thumbs-Up
**Criteria:**
- Technical lead confirms "we can build this"
- Architecture decisions are preliminary but clear (monolith vs micro, tech stack direction)
- Risk assessment is thorough (no hidden gotchas)
- Proof of concept is planned if technology is unproven

**Who approves:** CTO, Chief Architect, Technical Lead

***

### Gate 3: Team Capability Assessment
**Criteria:**
- Team has required expertise or path to get it
- Hiring plan is realistic (if needed)
- Ramp-up time is included in timeline
- Key person dependencies are mitigated

**Who approves:** Engineering Manager, HR (if hiring needed)

***

### Gate 4: Formal Go/No-Go Decision
**Criteria:**
- All deliverables are complete
- All concerns from feasibility analysis are addressed (mitigated or accepted)
- Stakeholders are aligned
- Resources are committed
- Timeline is approved

**Who approves:** Project sponsor, executive stakeholder, technical lead (consensus)

***

## 0.5 COMMON PITFALLS & HOW TO AVOID

| Pitfall | Why It Happens | How to Avoid |
|---------|---------------|------------|
| **Vague success criteria** | Stakeholders assume everyone agrees on "success" | Write measurable targets: "p99 latency < 200ms", not "fast" |
| **Underestimated timeline** | Optimism bias, ignoring ramp-up time | Add 20-30% contingency, use historical data from similar projects |
| **Hidden agenda stakeholders** | Politics/turf wars not surfaced early | Stakeholder interviews should ask "what are you worried about" |
| **Technical feasibility ignored** | Saying "yes" to everything | Force hard feasibility assessment, spike risky tech early |
| **Resource commitment uncertain** | "We'll allocate people later" | Get formal commitment now, understand opportunity cost |
| **No risk register** | Assuming "nothing bad will happen" | Identify top 10 risks, probability and impact, mitigation strategy |
| **Market validation skipped** | Building what we think users want | Talk to 10+ actual potential users, ask if they'd pay |
| **External dependencies unclear** | Partner company doesn't deliver | Map all external dependencies, confirm timelines early |
| **Scope not bounded** | "We'll add that during project" | Document what's in MVP, what's cut, what's deferred |
| **Decision-maker unclear** | Who actually has authority to approve/change things | Explicit approval chains, decision log |

***

## 0.6 PHASE 0 CHECKLIST

- [ ] Stakeholder map created with approval chains
- [ ] Business objectives documented and clear
- [ ] Success criteria are measurable and agreed
- [ ] Budget and timeline are formal constraints
- [ ] Team composition assessed, gaps identified
- [ ] Technical feasibility validated
- [ ] Proof of concept planned (if needed)
- [ ] Market analysis completed
- [ ] Risk register documented with top 10 risks
- [ ] Resource availability confirmed
- [ ] Project charter signed by sponsor
- [ ] Feasibility report completed
- [ ] All approval gates passed
- [ ] Go/No-Go decision recorded
- [ ] Transition plan to Phase 1 ready

***

## 0.7 ESTIMATED DURATION

**Timeline: 1-2 weeks**

- Stakeholder identification & interviews: 2-3 days
- Feasibility analysis & market research: 3-4 days
- Risk assessment & team planning: 2-3 days
- Documentation & approval: 2-3 days
- **Total: 9-13 days**

**Parallel:** Some activities can overlap (feasibility analysis while scheduling interviews)

***

## 0.8 TRANSITION TO PHASE 1

**When Phase 0 is complete:**
- Formal Go decision is made
- Resources are committed and available
- Stakeholders are aligned
- Risks are documented and mitigated
- Team understands constraints and success criteria

**Phase 1 begins when:** All gates are passed and sponsor gives explicit approval to proceed

***

**END OF PHASE 0**

This phase is your insurance policy. Rushing through it causes cascading failures in Phases 1-12. A good Phase 0 takes 1-2 weeks and saves 4-8 weeks of chaos later.