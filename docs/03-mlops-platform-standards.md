# 03 — MLOps Platform Standards

Standards for the AI platform's MLOps practices. Every team uses these. Deviations require an ADR.

---

## Experiment Tracking

**Tool:** MLflow (self-hosted)  
**Rule:** Every training run is tracked. No untracked experiments in production code paths.

Every tracked experiment must log:
- All hyperparameters (use `mlflow.log_params(vars(args))` — no selective logging)
- Training and validation metrics at each epoch/iteration
- Final evaluation metrics on the hold-out set
- Model artefact
- Training data version (hash or dataset ID)
- Random seed
- Environment: `mlflow.log_artifact(requirements.txt)`

**Why this matters:** An untracked run cannot be reproduced. An irreproducible model cannot be audited, rolled back, or compared. All three are production requirements here.

---

## Model Versioning

MLflow model stages:

| Stage | Meaning | Who promotes |
|---|---|---|
| `None` | Experiment / under development | Anyone |
| `Staging` | Passed CI validation; ready for review | CI pipeline (automated) |
| `Production` | Approved for live inference | Platform Lead sign-off |
| `Archived` | Superseded; kept for audit | Platform Lead |

**Promotion rules:**
- `None → Staging`: CI pipeline green (accuracy ≥ threshold, latency ≤ SLA, security scan pass)
- `Staging → Production`: Model card complete, compliance review done, rollback tested, Platform Lead approval
- `Production → Archived`: Only when a new version is promoted to Production; never deleted

**Naming convention:** `{domain}_{task}_{architecture}` (e.g., `ops_alert_classification_xgboost`, `logistics_demand_forecast_distilbert`)

---

## CI/CD for Models

Every model training and deployment job runs through GitHub Actions.

### Training pipeline CI

```yaml
# Triggered on: push to model/training/** or manual dispatch
jobs:
  train-and-validate:
    steps:
      - Checkout + environment setup
      - Data validation: schema check, freshness check, completeness ≥ 98%
      - Training run (MLflow tracked)
      - Evaluation: assert metrics ≥ thresholds (fail build if not)
      - Security scan: no credentials in artefact
      - Latency test: p95 inference ≤ SLA on benchmark dataset
      - Promote to Staging in MLflow registry (if all pass)
      - Post model card draft to PR
```

### Deployment pipeline CI

```yaml
# Triggered on: Production promotion approved in MLflow
jobs:
  deploy-to-production:
    steps:
      - Pull Production model from MLflow registry
      - Deploy to staging environment
      - Smoke test: 10 inference requests, all return valid predictions
      - Deploy to production (blue/green)
      - Health check: /health endpoint returns 200
      - Verify monitoring alerts are active
      - Log deployment to audit trail
      - Notify on-call: deployment complete
```

**Rollback:** Automated rollback if health check fails. Manual rollback: `mlflow models serve --model-uri models:/name/[previous_version]` — tested in staging for every model before production promotion.

---

## Inference API Standards

All model inference is served via FastAPI. Standards:

| Requirement | Standard |
|---|---|
| Health endpoint | `GET /health` → `{"status": "ok", "model_version": "..."}` |
| Prediction endpoint | `POST /predict` with structured request/response schema |
| Model version in response | Always included — every prediction response includes the model version that produced it |
| Input validation | Pydantic schema validation on all inputs; reject invalid payloads with 422 |
| Audit logging | Every prediction logged: timestamp, model version, input hash, prediction, confidence |
| Latency SLA | p95 ≤ 200ms for synchronous inference |
| Error handling | Structured error responses; no stack traces in production API responses |
| Rate limiting | Applied at API gateway level |

---

## Monitoring & Alerting

### Data drift monitoring

- **Method:** Population Stability Index (PSI) on key input features
- **Frequency:** Daily batch job against last 7 days of inference data vs. training distribution
- **Thresholds:** PSI > 0.1 → warning alert; PSI > 0.2 → P2 alert (model review required)
- **Action:** P2 drift alert triggers model performance review within 48h

### Prediction distribution monitoring

- **What:** Distribution of predicted classes / regression output
- **Frequency:** Weekly review (automated report to tech lead)
- **Threshold:** >15% shift in predicted class distribution vs. baseline → investigation

### Latency monitoring

- **Metrics:** p50, p95, p99 latency per model endpoint
- **Alert:** p95 > SLA for 3 consecutive minutes → P2 alert
- **Dashboard:** Grafana; link in runbook

### Retraining triggers

| Trigger | Action |
|---|---|
| PSI > 0.2 on any key feature | Initiate retraining pipeline |
| Model accuracy below threshold on validation set | Initiate retraining pipeline |
| Manual trigger by tech lead | Initiate retraining pipeline |
| Scheduled: monthly | Evaluate whether retraining is needed |

---

## Feature Engineering Standards

- Features are defined in code, not in notebooks. Notebooks are for exploration only.
- Feature definitions are versioned alongside model code.
- No feature is computed differently in training vs. inference (training-serving skew = production bug).
- Feature store pattern: shared feature definitions imported as a library, not copy-pasted.

---

## Reproducibility Checklist

Before any model is promoted to Staging, the following must be true:

- [ ] Random seed fixed and logged
- [ ] Training data version recorded (hash or dataset ID in MLflow)
- [ ] All dependencies pinned in `requirements.txt` or `pyproject.toml`
- [ ] Training run can be re-executed from the MLflow run ID and produce metrics within 1% of logged values
- [ ] Model artefact loaded from MLflow, not from a local file path
