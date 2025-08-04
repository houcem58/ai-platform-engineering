# 07 — Engineering Standards

**Owner:** AI & Data Platform Lead  
**Scope:** All workstreams — AI Engineering, Data Engineering, Software Engineering, MLOps/DevOps  
**Last reviewed:** 2025-Q4

---

## Code Review Standards

### Requirements (non-negotiable)

- Every PR requires at least **2 approvals**: one from the same workstream, one from a different workstream
- The cross-workstream review exists specifically to catch interface and integration issues — it is not optional
- No PR merges without CI passing (tests green, linting green, docs check green)
- PR description must include: what changed, why, how to test, any rollback considerations

### Review SLA

| PR size | Review SLA |
|---|---|
| < 100 lines changed | 4 hours |
| 100–500 lines | 1 business day |
| > 500 lines | 2 business days (decompose if possible) |

Tech leads are responsible for enforcing review SLAs within their workstream. Stale PRs are escalated to me at the weekly cross-team sync.

### What reviewers check

- **Correctness:** does the code do what the description says?
- **Observability:** are new code paths logged? Are metrics emitted for new inference paths?
- **Rollback:** can this change be reverted without data loss or service disruption?
- **Standards:** does it follow the conventions in this document?

---

## Branch and Commit Conventions

### Branch naming

```
feature/<workstream>/<short-description>    # new functionality
fix/<workstream>/<short-description>        # bug fix
chore/<workstream>/<short-description>      # maintenance, docs, refactor
model/<project>/<model-name>-v<N>           # model training runs
```

Examples:
```
feature/data-eng/add-olist-streaming-adapter
fix/mlops/drift-monitor-false-positives
model/project-alpha/xgboost-v3
```

### Commit message format

```
<type>(<scope>): <subject>

<body — optional, explains why not what>

<footer — references: closes #N, breaking changes>
```

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `model`

Examples:
```
feat(data-eng): add PSI feature drift metrics to monitoring pipeline
fix(mlops): correct threshold calibration for retail dataset
docs(adr): add ADR-003 for federated coordinator selection
model(project-alpha): promote XGBoost v3 to staging (F1=0.847 vs v2=0.831)
```

---

## Architecture Decision Records (ADRs)

Every significant technical decision must be documented as an ADR in `docs/adr/` before implementation begins.

### What requires an ADR

- Technology selection (framework, library, infrastructure component)
- Cross-workstream architecture changes
- Any decision that affects the data contract between components
- Security or compliance-relevant decisions

### ADR template

```markdown
# ADR-NNN — [Decision Title]

**Status:** [Proposed | Accepted | Deprecated | Superseded by ADR-NNN]
**Date:** YYYY-QN
**Author:** [Name]
**Reviewers:** [Names — minimum: all 4 tech leads + Platform Lead for cross-workstream decisions]

## Context
[What problem are we solving and why now?]

## Options Considered
| Option | Pro | Con | Decision |
|---|---|---|---|
| ... | | | ✅/❌ |

## Decision
[What we chose and the specific configuration/implementation details.]

## Rationale
[Why this option over the others — specific to our constraints.]

## Consequences
**Easier:** [What this makes easier]
**Harder:** [What this makes harder]
**Off the table:** [What we explicitly ruled out]

## Review Trigger
[When to revisit this decision: scale thresholds, dependency changes, new requirements]
```

---

## Observability Requirements

Every new service, model endpoint, or data pipeline must include on day one:

| Component | Required observability |
|---|---|
| FastAPI inference endpoint | `/metrics` (Prometheus), `/health` (model version + status), request latency histogram |
| Training pipeline | MLflow experiment run with all params, metrics, and artefacts logged |
| Data pipeline (Kafka/Spark) | Record count, error count, and processing latency per batch |
| dbt model | dbt tests passing in CI; freshness check configured |
| Background job | Completion status, duration, and failure reason logged |

Observability is part of the Definition of Done — a feature is not done if there is no way to know it is working in production.

---

## Definition of Done (Platform-wide)

A story is **Done** when all of the following are true:

- [ ] CI pipeline green (tests, linting, docs check)
- [ ] Peer review approved by own workstream + one other workstream
- [ ] Runbook updated if operational behaviour changed
- [ ] Monitoring configured (metrics emitted, Grafana dashboard updated if new component)
- [ ] Model card updated if a model was retrained or promoted
- [ ] ADR written if a significant architecture decision was made
- [ ] No open comments on the PR

Tech leads are accountable for enforcing this within their workstream. I do not accept stories as done without confirmation that all criteria are met.

---

## Security Standards

| Standard | Requirement |
|---|---|
| Secrets | Never in code, never in MLflow run params, never in logs. Use Docker Compose secrets or environment variables. |
| Dependencies | `pip-audit` run in CI on every push. Critical CVEs block merge. |
| Audit logs | All inference predictions logged with input hash (not raw input), model version, and user/system identifier. |
| Data access | Engineers access only their workstream's data sources. Cross-workstream data access requires Platform Lead approval. |
| Classified data | Any model processing data above a defined classification level requires Compliance Officer sign-off before deployment. |

---

## On-Call and Incident Protocol

| Priority | Definition | Response time | Escalation |
|---|---|---|---|
| P1 — Production down | Inference API returning errors; data pipeline stopped | Immediate (on-call engineer) | Platform Lead notified within 15 min |
| P2 — Degraded | p95 latency > SLA; drift alert firing | 30 minutes | Tech lead notified; Platform Lead if unresolved in 2h |
| P3 — Non-critical | Monitoring gap; non-critical job failure | Next business day | Tech lead |

On-call rotation: one engineer per workstream per week. On-call engineers carry a runbook for every production component in their workstream.

Post-incident: every P1/P2 incident requires a written incident report within 48 hours (see `docs/adr/` format adapted for incidents). Root cause, timeline, corrective actions, owner, deadline.
