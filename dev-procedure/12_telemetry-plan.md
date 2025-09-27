# 12 — Data & Telemetry Plan
**Purpose**: Measure product & system health.  
**Inputs**: PRD metrics, SLOs.  
**Outputs**: Event schema, dashboards, alerts.

## Checklist
- Analytics events (names, props, semantics)
- Product funnels (activation, retention)
- Observability (logs/metrics/traces)
- SLOs + alerts & runbooks
- PII handling & retention

**Definition of Done**: Dashboards/alerts live in staging.

### AI Prompt
Act as a data/observability lead. Produce a Data & Telemetry Plan for **{{product}}**.  
Deliver: event schema (table), funnel definitions, key dashboards, SLIs/SLOs with alert thresholds, logging/trace strategy, and data governance (PII, retention). Return in Markdown.
