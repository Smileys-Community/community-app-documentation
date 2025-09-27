# 11 — Quality & Security Plan
**Purpose**: Shift-left quality.  
**Inputs**: Tech docs.  
**Outputs**: Test strategy, perf budget, SAST/DAST plan, threat model.

## Checklist
- Unit/integration/contract/E2E test strategy
- Test data management
- Perf budget & load test plan
- Security: SAST/DAST/dep scan; threat model
- Quality gates in CI

**Definition of Done**: Quality gates running, baseline tests pass.

### AI Prompt
You are a QA/Sec lead. Create a Quality & Security Plan for **{{system}}**.  
Include: test pyramid, coverage targets, E2E critical paths, data seeding, performance budgets & load test scenarios, security controls (SAST/DAST/SBOM, authz tests), and CI quality gates. Output in Markdown with checklists.

