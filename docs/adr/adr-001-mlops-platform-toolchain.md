# ADR-001 — MLOps Platform Toolchain Selection

**Status:** Accepted  
**Date:** 2025-Q1 (Platform inception)  
**Author:** Houcem Hammami  
**Reviewers:** MLOps Lead, AI Engineering Lead, Data Engineering Lead

---

## Context

The AI Platform team (15 engineers, 4 concurrent projects) needs a shared MLOps toolchain. With 4 projects running in parallel, inconsistent tooling would create: incompatible experiment formats, duplicate infrastructure, and knowledge silos between teams. One toolchain must serve all projects.

Requirements:
- Experiment tracking across all 4 projects
- Model registry with staged promotion (None → Staging → Production → Archived)
- Works on-premise (no cloud dependency)
- All 4 tech leads must be able to operate it without MLOps specialist support
- Open source (no per-seat licence that scales with team)

---

## Options Considered

| Option | Experiment tracking | Model registry | Self-hosted | Team familiarity | Decision |
|---|---|---|---|---|---|
| **MLflow (chosen)** | ✅ Full | ✅ Full staged registry | ✅ Docker | Medium — 3/4 leads with prior exposure | ✅ Selected |
| **Weights & Biases** | ✅ Excellent UI | ✅ | ❌ SaaS-only | Low | ❌ Rejected — SaaS, per-seat cost |
| **DVC + custom registry** | ✅ Git-native | ⚠️ Manual tagging | ✅ | Low | ❌ Rejected — registry feature too manual for 4 concurrent projects |
| **Neptune.ai** | ✅ | ✅ | ❌ SaaS | None | ❌ Rejected — SaaS, no team familiarity |
| **Kubeflow** | ✅ | ✅ | ✅ | None | ❌ Rejected — Kubernetes dependency not yet in place; over-engineered for current scale |

---

## Decision

**MLflow** self-hosted on-premise with:
- PostgreSQL backend (reuses existing DW infrastructure — no new DB service)
- NFS artefact store (shared across all project teams)
- Docker Compose deployment managed by MLOps Lead
- Single MLflow instance serving all 4 projects (namespaced by experiment prefix: `{project}/{model_name}`)

**Supplementary:** GitHub Actions for CI/CD pipeline; Prometheus + Grafana for production model monitoring (separate from MLflow — MLflow tracks experiments, not production runtime metrics).

---

## Rationale

W&B and Neptune were rejected on SaaS and cost grounds — on-premise is a hard requirement given data classification constraints, and per-seat cost at 15 engineers is not justifiable for tooling.

Kubeflow was rejected because it requires Kubernetes, which is currently in progress (see ADR-002). Kubeflow is the right answer at 30+ engineers with Kubernetes in place; it is over-engineered now.

DVC was rejected because its model registry (Git tags + DVC remote) requires too much manual discipline for 4 parallel teams. MLflow's staged promotion model is more suitable for a team operating a formal governance process (Staging → Production requires CI gate and Platform Lead sign-off).

MLflow was chosen because: (a) 3/4 tech leads have used it before — lowest onboarding cost; (b) PostgreSQL backend reuses existing infrastructure; (c) staged registry maps directly to our governance process; (d) it scales to the 4-project portfolio without modification.

---

## Consequences

**Easier:**
- All teams share one experiment UI — cross-project model comparisons are possible
- Staged promotion integrates directly with CI pipeline (`mlflow models transition-stage` in GitHub Actions)
- No additional infrastructure to provision (PostgreSQL already exists; NFS already exists)

**Harder:**
- NFS artefact store is a single point of failure for model artefacts — must be monitored and backed up
- MLflow server is a new operational dependency — runbook required; MLOps Lead owns availability
- Large model artefacts (LLMs) will stress NFS — Phase 2 should evaluate object storage (MinIO)

**Off the table:**
- Per-project experiment tracking in different tools — all projects use MLflow or raise an ADR to justify the exception

---

## Review Trigger

Revisit when: (a) Kubernetes is in production (Kubeflow becomes viable), (b) team grows beyond 25 engineers (MLflow single-instance may become a bottleneck), (c) NFS reliability becomes an operational concern, (d) a project requires a model format MLflow does not support natively.
