# 07 — Technical Architecture Design
**Purpose**: Make high-level architecture decisions (build vs. buy, service boundaries, integration strategy) that de-risk delivery and set crisp guardrails.  
**Inputs**: PRD/FRDs, non-functional requirements (performance, security, availability), compliance constraints, budget & timelines.  
**Outputs**: Target-state architecture view, Architecture Decision Records (ADRs), build-vs-buy matrix with recommendations, integration map, SLOs/SLIs, cost & capacity assumptions, phased roadmap.

## Scope (What this covers)
- Architecture principles & constraints
- Target-state architecture (context → container → component at coarse granularity)
- Build vs. buy (including open source vs. managed service)
- Service & data boundaries; upstream/downstream integration map
- Non-functionals: availability, performance, scalability, security, privacy, compliance
- SLOs/SLIs & error budgets
- Capacity & throughput assumptions; initial sizing
- Cost model (order-of-magnitude) & total cost of ownership (TCO)
- Risks, trade-offs, and phased migration/expansion plan

## Checklist
- Architecture principles (3–7 bullets)
- Context diagram (system, users, key externals—textual description ok)
- Container diagram (apps/services/data stores; protocols)
- Data flow (critical paths; PII flow & residency)
- Build vs. buy matrix & decision
- Integration inventory (systems, contracts, auth, rate limits, SLAs)
- Security posture (authN/Z model, secrets, encryption at rest/in transit, threat surfaces)
- SLOs/SLIs (latency, availability, quality, cost) + alerting posture
- Capacity plan assumptions (QPS, concurrency, storage growth)
- Cost estimate v0 (infra + vendors) and cost levers
- ADRs authored for the top decisions (status: proposed/accepted)
- Phase plan: MVP → v1 → scale (with architecture deltas)

## Templates

### A) Architecture Principles (example)
1. Prefer **managed** over self-hosted when it removes undifferentiated heavy lifting.  
2. Keep **stateful** components minimal and well-bounded.  
3. **Backwards-compatible** interfaces; contracts first.  
4. **Security by default**: least-privilege, zero trust between services.  
5. **Observability first**: logs/metrics/traces for every critical path.

### B) Build vs. Buy Matrix
| Capability | Options | Pros | Cons | Est. Cost (12 mo) | Effort (S/M/L) | Risk | Recommendation |
|---|---|---|---|---:|---|---|---|
| {{Search}} | Build, {{Vendor A}}, {{Vendor B}} | … | … | $… | M | Privacy/lock-in | **Buy {{Vendor A}}** |
| {{Data Warehouse}} | {{Open Source}}, {{Cloud Managed}} | … | … | $… | S | Ops | **Buy Managed** |

### C) Integration Inventory
| System | Purpose | Interface | Auth | Rate/SLA | Owner | Notes |
|---|---|---|---|---|---|---|
| {{CRM}} | Leads sync | REST /webhooks | OAuth | 5 rps / 99.9% | Sales Ops | … |

### D) SLOs/SLIs
| Service | SLI | SLO Target | Alert Threshold | Measure Source |
|---|---|---:|---:|---|
| {{API}} | p95 latency | ≤ 300 ms | ≥ 80% of SLO breach 1h | Traces |
| {{Worker}} | Success rate | ≥ 99.5% | 99.2% for 15m | Metrics |

### E) Capacity & Cost Assumptions
- **Traffic model**: {{X}} QPS peak, {{Y}} QPS p95, burst factor {{B}}.  
- **Data growth**: {{N}} GB/day logs, {{M}} GB/day OLTP, {{W}} GB/day analytics.  
- **Cost levers**: storage tiering, reserved instances, query pruning, cache hit rate.

### F) ADR Template
`# ADR-{{id}}: {{Decision Title}}`
- Status: Proposed | Accepted | Superseded by ADR-{{id}}
- Context: {{Problem, constraints, forces}}
- Options: {{Option A}}, {{Option B}}, {{Option C}}
- Decision: {{Chosen option and why}}
- Consequences: {{Positive/negative trade-offs, follow-ups}}
- References: {{Links, benchmarks, spikes}}


## Deliverables
- **Target-state diagrams** (textual descriptions acceptable in docs)
- **ADRs** for: primary datastore, messaging, deployment model, identity, multi-tenancy, analytics stack, search/recommendations (if relevant)
- **Build-vs-Buy pack** with recommendation & exit strategy (avoid lock-in)
- **Integration map** & contracts to be specified (OpenAPI/GraphQL stubs)
- **SLO deck** with alerting posture & error budgets
- **Phase plan** (MVP → v1 → scale), with decision “reopen” triggers

**Definition of Done**: ADRs accepted; build-vs-buy decisions ratified with costs; target-state and phase plan approved by PM/TL/Sec; SLOs documented; risks logged with owners and mitigations.

---

### AI Prompt
You are a principal software architect. Produce a **Technical Architecture Design** for **{{system}}** focused on high-level decisions and build-vs-buy trade-offs.

**Include:**
1) Architecture principles & constraints  
2) Context/container/component (coarse) diagrams — textual descriptions are fine  
3) Build-vs-buy matrix with TCO (12 months), effort, risks, and recommendation (incl. exit/mitigation for vendor lock-in)  
4) Service & data boundaries; critical data flows (incl. PII handling, residency)  
5) Integration inventory (interfaces, auth, SLAs) and an API contract stub list  
6) Security posture (authN/Z model, encryption, secrets, threat surfaces)  
7) SLOs/SLIs & error budgets; observability plan  
8) Capacity assumptions & initial sizing; cost model v0 with key cost levers  
9) ADRs (for datastore, messaging, deployment, identity, analytics, search/ML if relevant) using the provided ADR template  
10) Phase plan (MVP → v1 → scale) with which decisions may be revisited

Output everything in **Markdown**, using the tables and templates above. Keep it concise but decision-oriented.

**Technologies under consideration**
- ... 
- ... 