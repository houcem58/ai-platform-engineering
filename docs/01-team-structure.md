# 01 — Team Structure & Operating Model

## Organisational Design

The AI Platform team is structured around **four technical disciplines** reporting to four tech leads, who report directly to me. This two-level structure keeps the management span manageable (4 direct reports) while enabling 15 people to operate with low coordination overhead.

```
AI & Data Platform Lead (Houcem Hammami)
│
├── AI Engineering Lead
│   ├── Senior AI Engineer × 2
│   ├── AI Engineer × 1
│   └── ML Research Engineer × 1
│
├── Data Engineering Lead
│   ├── Senior Data Engineer × 2
│   └── Data Engineer × 1
│
├── Software Engineering Lead
│   ├── Senior Software Engineer × 1
│   └── Software Engineer × 1
│
└── MLOps / DevOps Lead
    ├── Senior MLOps Engineer × 1
    └── DevOps Engineer × 1
```

**Total: 15 engineers + 1 Platform Lead**

---

## Roles and Responsibilities

### Tech Leads (×4)

Tech leads own their workstream's technical quality, team health, and delivery. They are not just senior engineers — they are responsible for:

- Technical decisions within their workstream (ADR-level decisions consulted with me)
- Sprint-level delivery: story breakdown, estimation, unblocking the team
- Code review culture: PR review SLA enforced within their workstream
- 1:1s with their direct reports (weekly)
- Growth plans for each engineer in their workstream (quarterly)
- Escalating cross-team dependencies and blockers to me

**What tech leads do not own:** cross-project roadmap, stakeholder communication, budget, hiring decisions, platform architecture across workstreams. These stay with me.

### My role as Platform Lead

| Responsibility | Cadence |
|---|---|
| Cross-project dependency management | Weekly cross-project sync |
| Platform architecture decisions | On demand, documented as ADRs |
| Stakeholder communication | Weekly (operational), Monthly (executive) |
| Tech lead 1:1s | Weekly (30 min each) |
| Team health monitoring | Sprint retrospective + quarterly |
| Hiring and headcount | Ongoing |
| Platform roadmap | Quarterly planning |
| AI governance sign-off | Per release |
| Budget and vendor management | Monthly |

---

## Cross-Project Coordination

With 4 concurrent projects running, I operate a lightweight **Programme Board** in addition to each project's sprint board:

| Forum | Frequency | Participants | Purpose |
|---|---|---|---|
| Daily standup | Daily, 15 min | Tech leads only | Cross-project blockers; my action items |
| Sprint planning sync | Every 2 weeks | All leads + me | Capacity allocation, shared dependency check |
| Cross-project weekly | Weekly, 45 min | All leads + me | Portfolio status, risk, roadmap |
| Stakeholder sync | Weekly, 30 min | Me + stakeholders | Delivery status, decisions needed |
| Executive review | Monthly | Me + Director | Portfolio health, outcomes, roadmap |
| Retrospective (portfolio) | End of each phase | All leads + me | Cross-project lessons, process improvement |

**Key principle:** Tech leads own their project standups. I attend only to unblock, not to observe.

---

## Growth Framework

Every engineer on the team has a growth plan anchored to three levels:

| Level | Focus | How I support it |
|---|---|---|
| **Developing** (0–2 yrs in role) | Technical depth, code quality, delivery fundamentals | Paired with senior; structured ADR reviews; explicit feedback monthly |
| **Proficient** (2–5 yrs) | System design, cross-team influence, quality ownership | Leading design sessions; presenting at Sprint Reviews; mentoring junior |
| **Senior / Lead** (5+ yrs) | Architecture, tech lead readiness, strategic thinking | Cross-project ownership; stakeholder exposure; hiring panels |

**Promotions:** I advocate for promotion when the engineer is already operating at the next level — not as a reward for tenure. I document the evidence and present it to the Director.

---

## Hiring Approach

For AI/ML and Data Engineering roles, I use a **four-stage process:**

1. **CV screen** (me) — 30 min: technical depth, real systems, evidence of ownership
2. **Technical screen** (tech lead) — 45 min: system design or data modelling problem relevant to the role
3. **Culture and working style** (me) — 45 min: how they handle ambiguity, conflict, failure; how they give feedback
4. **Reference check** — I call references directly; I do not accept written references only

**What I'm looking for:** Evidence of ownership ("I built this, it failed, here's what I learned") over credentials. I am less interested in a perfect CV than in someone who has shipped something real and knows what went wrong.

---

## Team Health

Team health is measured at every sprint retrospective using a 6-dimension survey (same framework as [agile-ai-delivery-playbook](https://github.com/houcem58/agile-ai-delivery-playbook)). Any dimension below 3.5 for two consecutive sprints triggers a specific intervention from me — not a "let's discuss in the next retro."

**Current team health:** All dimensions above 4.0 as of last measurement.
