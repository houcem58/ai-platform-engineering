# ADR-002 — Inference Serving Strategy

**Status:** Accepted  
**Date:** 2025-Q1  
**Author:** Houcem Hammami  
**Reviewers:** AI Engineering Lead, Software Engineering Lead, MLOps Lead

---

## Context

The platform serves predictions from multiple models to downstream systems and dashboards. We need a consistent inference serving approach across all 4 projects. Requirements: p95 latency ≤ 200ms for synchronous requests, on-premise deployment, model version in every response, audit logging of all predictions.

---

## Options Considered

| Option | Latency | Versioning | Audit | Complexity | Decision |
|---|---|---|---|---|---|
| **FastAPI + Uvicorn (chosen)** | ✅ <100ms overhead | ✅ Custom header | ✅ Middleware | Low | ✅ Selected |
| MLflow built-in serving (`mlflow models serve`) | ✅ | ⚠️ Via env var | ❌ Not built-in | Very low | ❌ Rejected — no audit logging; limited middleware support |
| TorchServe | ✅ | ✅ | ⚠️ Plugin needed | High | ❌ Rejected — PyTorch-only; overkill for mixed model types |
| Triton Inference Server | ✅ Excellent | ✅ | ⚠️ | Very high | ❌ Rejected — GPU-optimised; adds NVIDIA dependency; team has no Triton experience |
| BentoML | ✅ | ✅ | ✅ | Medium | ⚠️ Considered — good option but team has no experience; FastAPI known to all |

---

## Decision

**FastAPI + Uvicorn** as the standard inference server for all models, with:
- Pydantic input/output schemas enforced on every endpoint
- Model version loaded from MLflow registry at startup (`Production` stage)
- Model version returned in every prediction response header and body
- Audit middleware: every request logged to PostgreSQL audit table (input hash, model version, prediction, latency, timestamp)
- Health endpoint: `GET /health` returning model version and status
- Containerised: each model served in its own Docker container, orchestrated via Docker Compose

---

## Rationale

FastAPI was chosen over MLflow built-in serving because: (a) audit logging is a compliance requirement — MLflow serving has no audit middleware; (b) FastAPI gives full control over request/response schemas, versioning headers, and middleware; (c) every engineer on the team already knows FastAPI.

BentoML is a strong alternative and should be reconsidered in Phase 2 if the number of models in production grows beyond 10 (BentoML's runner architecture is better suited to multi-model serving).

Triton and TorchServe are GPU-serving frameworks — appropriate when GPU utilisation efficiency is the bottleneck. We are not at that scale.

---

## Consequences

**Easier:**
- Every engineer can read, debug, and extend any inference API — FastAPI is universal knowledge on the team
- Audit logging is centralised in middleware — not a per-model implementation concern
- Blue/green deployments work naturally with Docker containers

**Harder:**
- Each model requires its own FastAPI service — operational overhead grows linearly with model count
- No built-in batching — models requiring batch inference need custom implementation
- Container orchestration complexity grows with model count — Kubernetes migration (Phase 2) will address this

**Off the table:**
- Deploying any model without the standard audit middleware
- Returning predictions without the model version in the response

---

## Review Trigger

Revisit when: (a) production model count exceeds 10 (BentoML or Triton may be warranted), (b) Kubernetes is in production (enables model serving at scale with resource isolation), (c) a model requires GPU-optimised batch inference that FastAPI + Uvicorn cannot meet within the latency SLA.
