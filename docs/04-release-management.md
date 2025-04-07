# 04 — Release Management

The platform ships **2 releases per month** — one per 2-week sprint. This cadence is only possible because the release process is fully automated. Manual deployment steps are a single-point-of-failure and a delivery bottleneck; we eliminated them.

---

## Release Cadence

| Sprint week | Activity |
|---|---|
| Week 1, Days 1–9 | Development, PR reviews, CI green |
| Week 2, Day 9 | Release candidate cut: `git tag v{YYYY.MM.N}-rc1` |
| Week 2, Day 9 | Release candidate deployed to staging; smoke tests run |
| Week 2, Day 9 | Sprint Review — stakeholder demo against staging |
| Week 2, Day 10 | Release approved → production deployment |
| Week 2, Day 10 | Post-deployment monitoring: 1-hour heightened watch |
| Week 2, Day 10 | Sprint Retrospective |

**Release is not tied to a date — it is tied to the CI pipeline being green and the Sprint Review acceptance.** If Sprint Review reveals a blocking issue, the release is held one sprint. This has happened once; the sprint was extended by 3 days, not the release forced through.

---

## Release Types

| Type | Frequency | Process |
|---|---|---|
| **Sprint release** | Every 2 weeks | Full CI/CD pipeline, staging validation, Sprint Review demo, production deployment |
| **Hotfix** | As needed | Branch from production tag, minimal change, accelerated CI (skip non-blocking tests), immediate deployment, post-mortem |
| **Model release** | As needed (decoupled from code release) | ML CI pipeline, model card, compliance review, blue/green deployment |

Model releases are **decoupled from code releases**. A new model version can be promoted to production without a code change — it goes through the ML CI pipeline independently.

---

## Release Pipeline Detail

```
Developer → PR → Code Review (24h SLA, cross-workstream)
    │
    ▼
CI Pipeline (GitHub Actions)
    ├── Lint + type check
    ├── Unit tests
    ├── Integration tests
    ├── Security scan (secret detection + dependency CVE)
    ├── Docker image build
    └── Push to container registry
    │
    ▼
Staging Deployment (automated on main branch merge)
    ├── Deploy to staging environment
    ├── Smoke tests (10 key API paths)
    ├── Performance test (p95 latency assertion)
    └── Data quality check (schema + freshness)
    │
    ▼
Sprint Review gate (manual — stakeholder acceptance)
    │
    ▼
Production Deployment (manual trigger by MLOps Lead after approval)
    ├── Blue/green deployment (traffic shifted 10% → 50% → 100% over 10 min)
    ├── Health check at each traffic step
    ├── Automated rollback if error rate > 0.5% during ramp
    └── Deployment logged to audit trail
    │
    ▼
Post-Deployment Monitoring (1-hour heightened watch)
    ├── Error rate dashboard
    ├── Latency p95
    └── Prediction distribution (for model-serving endpoints)
```

---

## Rollback Protocol

Rollback is treated as a **normal operation**, not an emergency. Every release has a documented and tested rollback path before it goes to production.

| Trigger | Rollback type | Time to rollback |
|---|---|---|
| Automated: error rate > 0.5% during blue/green ramp | Automatic traffic reversion | < 2 min |
| Manual: P1 incident post-deployment | MLOps Lead executes rollback | < 15 min |
| Model degradation | MLflow: revert to previous Production model | < 30 min |

**Rollback is not a failure — it is the safety net working.** We track rollbacks, learn from them, and do not penalise teams for triggering them. We do penalise deploying without a tested rollback path.

---

## Release Notes

Every release has a release note published in the internal wiki and linked from the GitHub release tag. Format:

```markdown
## Release v{YYYY.MM.N} — {Sprint name}

**Date:** YYYY-MM-DD
**Deployed by:** [MLOps Lead name]

### What's in this release
- [Feature 1 — user-facing description]
- [Feature 2]
- [Bug fix 1]

### Models updated
- [Model name] → v[X.Y] (see model card for changes)

### Breaking changes
- [None / description]

### Rollback
- Previous version: v{YYYY.MM.N-1}
- Rollback command: [documented]

### Monitoring
- Heightened watch period: until [datetime]
- On-call: [name]
```

---

## Release Metrics

Reviewed at each monthly executive review:

| Metric | Target | Current |
|---|---|---|
| Releases per month | 2 | 2 ✅ |
| Release success rate (no rollback) | ≥ 90% | 100% (last 6 months) |
| Mean time to deploy (staging → production) | ≤ 4h | ~2h ✅ |
| Mean time to rollback (when triggered) | ≤ 15 min | < 15 min ✅ |
| Critical incidents post-deployment | 0 | 0 ✅ |
| Sprint Review acceptance rate | 100% | 100% ✅ |
