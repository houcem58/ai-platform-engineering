# 02 — AI Governance Framework

Every model deployed to this platform is governed by the framework below. Governance is not an audit process — it is built into the delivery workflow. A model that cannot pass these gates does not reach production, regardless of accuracy metrics.

---

## Governance Principles

1. **No model reaches production without a model card.** Accuracy is one metric. Bias, data provenance, failure modes, and limitations are others. All of them are documented before deployment.
2. **Explainability is an operational requirement, not a research concern.** If an analyst cannot understand why the model produced a prediction, the model is not production-ready for our environment.
3. **Every deployment has a tested rollback.** Tested means: the rollback was executed in staging and confirmed to restore the previous behaviour within SLA.
4. **Compliance review is a gate, not a checkbox.** Security Officer sign-off is required for any model processing classified data. This is not waivable under delivery pressure.
5. **Monitoring is not optional.** A model without drift detection and prediction distribution monitoring is not deployed.

---

## Model Lifecycle

```
Research & Experimentation
    │  MLflow experiment tracked
    │  Baseline established
    ▼
Model Development
    │  Training reproducible (seed fixed, data versioned)
    │  Evaluation on hold-out set
    │  Bias assessment completed
    ▼
Model Card Drafted
    │  See template below
    ▼
CI Validation Gate
    │  Automated tests: accuracy ≥ threshold
    │  Latency test: p95 ≤ SLA
    │  Security scan: no credential leakage in artefact
    ▼
Staging Deployment
    │  Rollback tested in staging
    │  Monitoring configured
    │  Tech lead sign-off
    ▼
Compliance Review
    │  Security Officer reviews model card
    │  Data sovereignty confirmed
    │  Classification check
    ▼
Production Deployment
    │  MLflow registry: None → Staging → Production
    │  Deployment logged to audit trail
    │  Alerts active
    ▼
Production Monitoring
    │  Data drift: daily check (PSI threshold)
    │  Prediction distribution: weekly review
    │  Latency: p95 monitored continuously
    │  Manual review: monthly model performance meeting
```

---

## Model Card Template

Every model in production has a completed model card in the MLflow registry and in the project repository.

```markdown
## Model Card — [Model Name] v[X.Y]

### Model Details
- **Architecture:** [e.g. XGBoost classifier / DistilBERT fine-tuned]
- **Training date:** YYYY-MM-DD
- **MLflow run ID:** [run_id]
- **Author:** [name]

### Intended Use
- **Primary use:** [what decision this model supports]
- **Out-of-scope uses:** [explicit list of what it should NOT be used for]

### Training Data
- **Source:** [data source, version]
- **Size:** [N samples, date range]
- **Preprocessing:** [steps applied]
- **Known biases in training data:** [explicit statement]

### Evaluation
| Metric | Value | Threshold | Status |
|---|---|---|---|
| Accuracy / F1 | | ≥ 0.85 | |
| Precision | | ≥ 0.80 | |
| Recall | | ≥ 0.80 | |
| p95 latency | | ≤ 200ms | |

- **Evaluation dataset:** [held-out set details]
- **Known failure modes:** [explicit description]

### Fairness Assessment
- Evaluated across: [subgroups assessed]
- Performance gap between subgroups: [value]
- Mitigation applied: [if any]

### Explainability
- Method: [SHAP / LIME / attention weights]
- Available at: [endpoint or notebook reference]

### Compliance
- Data classification: [level]
- Security Officer sign-off: [name, date]
- Data sovereignty: [confirmed / N/A]

### Monitoring
- Drift detection: PSI on key features, daily
- Prediction distribution alert threshold: [value]
- Latency alert: p95 > [Xms] → PagerDuty

### Rollback
- Previous production version: [version]
- Rollback command: `mlflow models serve --model-uri models:/[name]/[version]`
- Rollback tested: [date, who tested]
```

---

## AI Ethics Checklist

Completed before any model is promoted from Staging to Production:

| Question | Required answer |
|---|---|
| Does the model make decisions that affect individuals or operations? | If yes → explainability required |
| Was the training data collected with appropriate authorisation? | Yes — documented in model card |
| Has the model been evaluated for bias across relevant subgroups? | Yes — fairness assessment in model card |
| Are the model's limitations documented and communicated to users? | Yes — model card + user documentation |
| Is there a human review process for high-stakes predictions? | Yes — defined in operational runbook |
| Can the model be rolled back within SLA if it degrades? | Yes — tested in staging |
| Is the model monitored for drift in production? | Yes — monitoring configured before deployment |

Any "No" or "Not yet" blocks the promotion to Production. This is not a soft gate.

---

## Compliance Requirements

### Data classification

All models processing data above a certain classification level require:
- Security Officer review of training data provenance
- Confirmation that no classified data is embedded in model weights or artefacts
- Storage of model artefacts in the approved, access-controlled repository only

### Data sovereignty

For models trained or serving predictions on data from distributed nodes:
- Raw data must never leave its node during training (federated approach required)
- Gradient/weight transmission only between nodes
- Differential privacy applied where required by compliance framework (ε calibrated per dataset)

### Audit trail

- Every model inference logged to audit schema (timestamp, model version, input hash, prediction)
- Audit logs retained 24 months
- Access to audit logs: Security Officer and Platform Lead only

---

## Governance Metrics

Reviewed monthly at the AI governance meeting (Platform Lead + Security Officer + tech leads):

| Metric | Target | Reviewed |
|---|---|---|
| % of production models with current model card | 100% | Monthly |
| % of models with drift monitoring active | 100% | Monthly |
| % of deployments with tested rollback | 100% | Per deployment |
| Model performance below threshold (flagged) | 0 | Monthly |
| Open compliance findings | 0 unresolved >30 days | Monthly |
| Mean time to rollback (when triggered) | ≤ 30 min | Per incident |
