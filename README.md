<div align="center">

# AI Platform Engineering

### Engineering Management Playbook — AI & Data Platform Lead

**15 engineers · 4 concurrent projects · 40% processing improvement · 2 releases/month**

[![Docs](https://img.shields.io/badge/Docs-Playbook-blue)](docs/)
[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-blue)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green)](LICENSE)

</div>

---

> Engineering management playbook for running a 15-person AI & Data Platform team delivering production-grade AI systems. Covers team structure, AI governance, MLOps standards, release management, and the operating model used to deliver 4 concurrent AI and data engineering projects.
>
> Role: **AI & Data Platform Lead** · 2025–Present

---

## Team at a Glance

| Discipline | Head count | Scope |
|---|---|---|
| AI Engineering | 5 | Model development, fine-tuning, federated learning, LLM systems |
| Data Engineering | 4 | Streaming pipelines, dbt transformations, data platform |
| Software Engineering | 3 | APIs, integrations, internal tooling |
| DevOps / MLOps | 3 | CI/CD, infrastructure, model deployment, monitoring |
| **Total** | **15** | End-to-end AI platform delivery |

**Reporting line:** Direct reports: 4 tech leads. Tech leads manage their workstreams.  
**My role:** Architecture governance · Stakeholder management · Delivery across 4 concurrent projects · Platform roadmap · Hiring and team growth.

---

## Platform Outcomes (2025–Present)

| Outcome | Metric |
|---|---|
| Data processing time reduction | **40%** — distributed processing + pipeline optimisation |
| Deployment frequency | **2 releases/month** — CI/CD + MLOps automation |
| Projects delivered to production | **4 AI and Data Engineering projects** |
| Infrastructure availability | **≥ 99%** sustained |
| Post-deployment incidents | **0 critical** since CI/CD full adoption |

---

## What's in This Playbook

```
docs/
├── 01-team-structure.md          Team org, tech leads, growth framework
├── 02-ai-governance-framework.md AI governance: model cards, fairness, compliance
├── 03-mlops-platform-standards.md MLOps practices: versioning, CI/CD, monitoring
├── 04-release-management.md      Release process: 2x/month cadence, rollback
├── 05-project-portfolio.md       4 active projects: scope, status, outcomes
├── 06-platform-architecture.md   System architecture overview
├── 07-engineering-standards.md   Coding standards, ADRs, review SLAs
├── 08-onboarding.md              Week 1 guide for new engineers
├── adr/
│   ├── adr-001-...               Architecture decisions
│   └── adr-002-...
```

---

## AI Governance Principles

Every model deployed to this platform has:

- **Model card** — evaluation metrics, training data provenance, known limitations, fairness considerations
- **Explainability** — SHAP values or equivalent for decisions affecting operations
- **Monitoring** — data drift detection, prediction distribution alerts, p95 latency SLA
- **Rollback** — every production model has a tested rollback path; zero models deployed without it
- **Compliance review** — Security Officer sign-off for models processing classified data

See full framework: [docs/02-ai-governance-framework.md](docs/02-ai-governance-framework.md)

---

## Engineering Management Approach

**Decision-making:** Delegated to tech leads within agreed architecture. Escalated to me when cross-team or cross-project. Documented as ADRs regardless of level.

**Delivery cadence:** 2-week sprints per workstream, coordinated at programme level. Monthly cross-project sync for dependency management and roadmap alignment.

**Quality bar:** No story is done without: CI green, peer review from another workstream, runbook updated, monitoring configured. Non-negotiable.

**Team growth:** Each tech lead owns a growth plan for their workstream. Quarterly competency reviews. I do not wait for annual cycles to give feedback.

→ **[Working with me →](https://github.com/houcem58/houcem58/blob/main/working-with-me.md)**

---

## Stack

| Layer | Technologies |
|---|---|
| AI/ML | PyTorch · HuggingFace Transformers · XGBoost · scikit-learn · LangChain |
| MLOps | MLflow · GitHub Actions · Docker · Prometheus · Grafana |
| Data Engineering | Apache Kafka · Apache Spark · dbt · PostgreSQL · DuckDB |
| APIs | FastAPI · Uvicorn |
| Infrastructure | Docker Compose · Kubernetes (in progress) · Terraform |
| Languages | Python · SQL |

---

## Author

**Houcem Hammami** — AI & Data Platform Lead

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://linkedin.com/in/houcem-hammami)
[![GitHub](https://img.shields.io/badge/GitHub-houcem58-black)](https://github.com/houcem58)
[![Email](https://img.shields.io/badge/Email-houcem0508%40gmail.com-red)](mailto:houcem0508@gmail.com)

---

## License

Copyright 2025–2026 Houcem Hammami. Licensed under the Apache License, Version 2.0.
