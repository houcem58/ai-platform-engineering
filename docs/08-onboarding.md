# 08 — Onboarding Guide

**For:** New engineers joining the AI & Data Platform team  
**Owner:** AI & Data Platform Lead  
**Last updated:** 2025-Q4

---

## Before Day 1

Your tech lead will send you a pre-onboarding email with:

- Access request form for internal systems (submit at least 3 days before start)
- Reading list: this repo's docs, the two ADRs, and the working-with-me guide
- Your first week schedule (1:1s, team intro, environment setup)

---

## Week 1 Goals

By end of Week 1 you should be able to:

- [ ] Access all systems (see Access Checklist below)
- [ ] Run the local development environment for your workstream
- [ ] Understand how your workstream's components fit in the overall platform architecture
- [ ] Read and understand the two ADRs in `docs/adr/`
- [ ] Know who to ask for what (tech leads, on-call, cross-workstream contacts)
- [ ] Attend your first sprint planning and daily standup

You are **not** expected to ship anything in Week 1. Understanding comes first.

---

## Access Checklist

| System | Access type | Requested via |
|---|---|---|
| GitHub (org + team repos) | Contributor | Your tech lead → IT |
| MLflow Tracking Server (internal URL) | Read + experiment creation | Tech lead |
| PostgreSQL Data Warehouse | Read-only (workstream schema) | Tech lead → DBA |
| Kafka cluster | Consumer group for your workstream | MLOps Lead |
| Grafana dashboards | Read-only | MLOps Lead |
| Prometheus (internal) | Read-only | MLOps Lead |
| Docker registry (internal) | Pull + push for your workstream | MLOps Lead |
| Jira project | Full access | Tech lead → IT |

Flag any missing access to your tech lead by Day 2. Do not wait until you need it.

---

## Environment Setup

### 1. Clone the relevant repos

```bash
# Platform engineering docs
git clone https://github.com/houcem58/ai-platform-engineering.git

# Your workstream repo (tech lead will share the internal URL)
git clone <internal-repo-url>
```

### 2. Set up Python environment

```bash
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Configure MLflow

```bash
export MLFLOW_TRACKING_URI=http://<internal-mlflow-host>:5000
mlflow experiments list           # should show existing experiments
```

### 4. Verify Kafka access

```bash
# Test your consumer group (your tech lead provides group name)
python -c "from kafka import KafkaConsumer; print('Kafka OK')"
```

### 5. Run the smoke test for your workstream

Each workstream has a `scripts/smoke_test.py` that validates end-to-end connectivity. Run it and share the output with your tech lead.

---

## Team Norms

**Standup** — 15 min daily, async-first (Slack update by 09:00; video only when blockers need discussion). Format: what I did yesterday / what I'm doing today / any blockers.

**Asking for help** — blockers raised immediately, not at standup. If you're blocked for more than 2 hours, ping your tech lead directly. There is no award for struggling in silence.

**Code review** — you are expected to review PRs within the SLA (see `docs/07-engineering-standards.md`). Reviews are a team responsibility, not optional extras.

**Experiments** — all experiments logged to MLflow from Day 1. If it's not in MLflow, it didn't happen. This is not bureaucracy; it is how we avoid repeating experiments and justify model promotions.

**Documentation** — ADRs written before implementation, not after. If you're about to make a significant architecture decision, open a PR with the ADR first and get it reviewed.

---

## First Story Guidelines

Your first story should be:

- **Small** — ideally under 3 story points
- **Self-contained** — touching only your workstream, no cross-team dependencies
- **In the main path** — not a test or docs task, but something that ships to the codebase

Your tech lead will help you pick the right first story. Do not self-assign. After you complete your first story, schedule a 30-minute retro with your tech lead to talk through what felt easy, what was unclear, and what you'd do differently.

---

## Key Contacts

| Role | Responsibility | How to reach |
|---|---|---|
| Your tech lead | Day-to-day delivery, technical decisions, PR reviews | Direct message |
| Platform Lead (me) | Cross-team issues, career growth, architecture decisions | Weekly 1:1 or DM for urgent |
| MLOps Lead | MLflow, Kafka, Docker, Grafana, on-call setup | Direct message |
| Data Engineering Lead | PostgreSQL, dbt, data access, feature store | Direct message |
| On-call engineer (this week) | P1/P2 production incidents | Incident channel (pinned in Slack) |

---

## Reading List (Week 1)

1. `docs/01-team-structure.md` — org design and my operating model
2. `docs/02-ai-governance-framework.md` — model card and ethics requirements
3. `docs/03-mlops-platform-standards.md` — experiment tracking and model versioning rules
4. `docs/06-platform-architecture.md` — system overview (read this before touching any infrastructure)
5. `docs/adr/adr-001-mlops-platform-toolchain.md` — why MLflow
6. `docs/adr/adr-002-inference-serving-strategy.md` — why FastAPI
7. `docs/07-engineering-standards.md` — code review, commit conventions, DoD
8. [working-with-me.md](https://github.com/houcem58/houcem58/blob/main/working-with-me.md) — how I work as a manager
