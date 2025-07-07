# 06 — Platform Architecture Overview

**Status:** Current · Last reviewed: 2025-Q4  
**Owner:** AI & Data Platform Lead (Houcem Hammami)

---

## Architecture Principles

1. **Separation of concerns** — experiment tracking (MLflow) is decoupled from production serving (FastAPI containers). One system failing does not take down the other.
2. **Auditability by default** — every prediction logged to PostgreSQL audit table; every model promotion recorded in MLflow registry; every deployment tracked in GitHub Actions.
3. **Self-hosted first** — all platform components run on-premise. No hard cloud dependency. Data classification requirements preclude SaaS tooling for core ML infrastructure.
4. **Standard interfaces** — all inference APIs follow the same FastAPI contract (health, predict, model-version header). Any workstream can call any model the same way.

---

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                                  │
│  Kafka topics · PostgreSQL sources · File ingestion · External APIs  │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ streaming + batch
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      DATA PLATFORM LAYER                             │
│                                                                      │
│   Apache Kafka (3-broker)     Apache Spark (batch)                  │
│        │                           │                                  │
│        ▼                           ▼                                  │
│   dbt transformation layer (PostgreSQL DW)                           │
│        │                                                              │
│        └──────► Feature Store (PostgreSQL feature tables)            │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ features + training data
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      ML PLATFORM LAYER                               │
│                                                                      │
│   MLflow Tracking Server ────────────────────────────────────────   │
│   (PostgreSQL backend + NFS artefact store)                          │
│        │                                                              │
│        │  experiment runs → model registry                            │
│        ▼                                                              │
│   MLflow Model Registry                                              │
│   (None → Staging → Production → Archived)                          │
│        │                                                              │
│        │  CI gate + Platform Lead sign-off required for Production   │
│        ▼                                                              │
│   GitHub Actions CI/CD Pipeline                                      │
│   (training pipeline + deployment pipeline)                          │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ containerised model deployment
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     INFERENCE SERVING LAYER                          │
│                                                                      │
│   FastAPI + Uvicorn (one container per model)                       │
│   ├── GET /health          → model version + status                 │
│   ├── POST /predict        → prediction + model_version header       │
│   └── Audit middleware     → all requests logged to PostgreSQL       │
│                                                                      │
│   Docker Compose orchestration (Kubernetes migration: Phase 2)       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ predictions + metrics
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     MONITORING LAYER                                  │
│                                                                      │
│   Prometheus (metrics scraping)                                      │
│   Grafana (dashboards: latency, prediction distribution, drift)      │
│   PSI drift alerts → retraining trigger → CI pipeline               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Component Inventory

| Component | Technology | Owner | Purpose |
|---|---|---|---|
| Stream broker | Apache Kafka 3.x (3-broker cluster) | MLOps Lead | Real-time event ingestion across all projects |
| Batch processing | Apache Spark | Data Engineering Lead | Large-scale ETL and feature computation |
| Data warehouse | PostgreSQL 15 | Data Engineering Lead | Central analytical store; dbt transformation target |
| Transformation | dbt-postgres 1.8 | Data Engineering Lead | SQL transformation layer with tests and lineage |
| Feature store | PostgreSQL feature tables | Data Engineering Lead | Versioned features for model training and serving |
| Experiment tracking | MLflow (self-hosted, PostgreSQL backend) | MLOps Lead | All experiments logged; parameter/metric/artefact tracking |
| Artefact store | NFS shared mount | MLOps Lead | Model artefacts shared across all workstreams |
| Model registry | MLflow Registry | Platform Lead | Staged promotion: None → Staging → Production |
| CI/CD | GitHub Actions | MLOps Lead | Training and deployment pipelines; CI gate for model promotion |
| Inference serving | FastAPI + Uvicorn (containerised) | Software Engineering Lead | One container per production model; audit middleware |
| Orchestration | Docker Compose | MLOps Lead | Container lifecycle management; Phase 2: Kubernetes |
| Metrics | Prometheus + Grafana | MLOps Lead | Latency, prediction distribution, drift alerting |
| Audit log | PostgreSQL audit table | Software Engineering Lead | All predictions logged: input hash, model version, latency |

---

## Data Flow — Training Path

```
1. Data Engineering: raw sources → Kafka / PostgreSQL
2. Data Engineering: dbt transformation → feature tables
3. AI Engineering: model training → MLflow experiment run (all params + metrics + artefacts)
4. AI Engineering: candidate model → MLflow Registry (Staging)
5. CI gate: GitHub Actions runs evaluation suite on Staging model
6. Platform Lead: reviews CI results + model card → promotes to Production
7. Deployment pipeline: pulls Production model → builds FastAPI container → deploys
```

## Data Flow — Inference Path

```
1. Downstream system → POST /predict on FastAPI endpoint
2. FastAPI: loads model (Production stage) at startup from MLflow Registry
3. FastAPI: runs prediction → returns result + model_version header
4. Audit middleware: logs request to PostgreSQL (input hash, model version, prediction, latency, ts)
5. Prometheus: scrapes /metrics → Grafana alerts if latency p95 > 200ms
6. Drift monitor: tracks PSI on prediction distribution → triggers retraining if threshold exceeded
```

---

## Security Architecture

| Control | Implementation |
|---|---|
| Data access | Role-based: engineers access only their workstream's data sources |
| Model promotion | Requires CI gate pass + Platform Lead sign-off — no self-promotion |
| Audit logging | All predictions logged; audit table append-only (no delete privileges for app users) |
| Secret management | Environment variables via Docker Compose secrets; no secrets in code or MLflow runs |
| Network | All components on internal network; inference APIs not publicly exposed |
| Classified data | Compliance Officer sign-off required before any model processes classified data |

---

## Known Limitations

| Limitation | Impact | Mitigation |
|---|---|---|
| NFS artefact store is a single point of failure | MLflow artefacts unavailable if NFS fails | Daily NFS backup; runbook for recovery |
| Docker Compose orchestration scales manually | Container restarts require manual intervention beyond ~10 models | Phase 2: Kubernetes migration |
| Single MLflow instance | Potential bottleneck beyond 25 engineers / 50 concurrent experiments | Horizontal scaling plan documented for Phase 2 |
| No GPU infrastructure | GPU-intensive models (LLMs) require off-platform training | Evaluate cloud GPU burst for Phase 2 |

---

## Architecture Decision Records

Key decisions documented in `docs/adr/`:

- **ADR-001** — MLflow chosen over W&B/DVC/Neptune/Kubeflow for MLOps toolchain
- **ADR-002** — FastAPI chosen over MLflow serving/TorchServe/Triton/BentoML for inference
