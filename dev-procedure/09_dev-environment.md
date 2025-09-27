# 9 — DevEx & Environments
**Purpose**: Make shipping easy & safe.  
**Inputs**: Tech docs.  
**Outputs**: Repos, CI/CD, IaC, secrets, staging/prod, feature flags.

## Checklist
- Repo structure & branch strategy
- CI: build/test/scan; CD: promote dev→stg→prod
- IaC for environments; secrets management
- Observability bootstrap
- “Hello world” deploy

**Definition of Done**: Green pipeline; deploy succeeds end-to-end.

### AI Prompt
You are a platform engineer. Write a DevEx setup plan for **{{stack}}**.  
Cover: repo/branching, CI/CD stages, testing gates, IaC modules, secrets strategy, environment topology (dev/stage/prod), feature flags, baseline observability (logs/metrics/traces), and a “hello world” deployment checklist. Output Markdown with checklists.
