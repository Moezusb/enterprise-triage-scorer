# Enterprise Triage Scorer

**Deterministic risk scoring and routing for enterprise B2B SaaS support operations.**

---

## [ 01. THE PROBLEM ]

Every enterprise support team eventually builds the same broken architecture: one general queue, manual ticket picking, and priority labels that carry no operational consequence. A cancellation request from a $50K ARR account waits behind a password reset. An after-hours SSO outage sits until morning. Agents open four systems to answer one question.

The cost lands on the accounts that matter most: the customers with the most revenue at stake wait in the same line as everyone else. The fix is not more agents. It is routing intelligence applied at intake.

This scorer does that deterministically.

---

## [ 02. WHAT IT DOES ]

An interactive risk scoring engine that evaluates support tickets at intake using account value, issue severity, and urgency signals — then assigns a routing lane, SLA target, and ownership recommendation in real time.

**[→ Open the live scorer](https://moezusb.github.io/enterprise-triage-scorer/)**

**Input signals scored at ticket intake:**

| Signal | Points |
|---|---|
| Enterprise account (100+ seats / $50K+ ARR) | +50 |
| SSO failure / workspace unavailable / multiple users blocked | +40 |
| Cancellation, downgrade, billing dispute, or overcharge | +35 |
| Renewal within 90 days | +30 |
| Repeat complaint from same account within 7 days | +25 |
| Import quality failure from enterprise account | +25 |
| Enterprise onboarding or implementation blocker | +20 |
| After-hours ticket | +15 |
| Churn language or executive / CEO mention | +10 |
| Known retryable issue with self-serve path available | −20 |
| Trial or free-tier account, low severity | −10 |

**Output per ticket:**
- Risk score (0–150+)
- Routing lane (P0 through P4)
- SLA target
- Ownership assignment
- One-line routing rationale

---

## [ 03. ROUTING LANES ]

| Lane | Threshold | SLA |
|---|---|---|
| P0 — Enterprise access outage | Score ≥ 90 | 3 min (biz hrs) / 20 min (after-hours) |
| P1 — Commercial risk | Score ≥ 70, commercial category | ACK 10 min / CS task 30 min |
| P1 — Product trust risk | Score ≥ 55, quality category | 15 min + proactive update |
| P2 — Enterprise standard | Score ≥ 40 | 15 min |
| P2 — Implementation / integration | Score ≥ 30, onboarding/API | 15 min |
| P3 — Billing | Score ≥ 20, billing category | 20 min |
| P3 — Technical | Score ≥ 10 | 10 min |
| P4 — General | Score < 10 | 30 min |

---

## [ 04. INTERACTIVE SCORER ]

Three tabs:

- **Risk scorer** — configure ticket inputs and watch the score, lane, SLA, and rationale update in real time
- **Sample tickets** — 12 pre-loaded enterprise support scenarios with current system vs. model comparison side-by-side
- **Score legend** — full weight table, lane definitions, and design rationale

The scorer is standalone HTML — no build step, no dependencies, no framework. Open `index.html` directly in any browser or deploy to any static host.

---

## [ 05. DESIGN DECISIONS ]

**Why deterministic instead of ML or LLM classification?**

Deterministic scores are auditable, explainable to agents, and tunable without retraining. An ops team can adjust a weight on Monday morning and see the effect on Tuesday's queue. That feedback loop matters more than marginal classification accuracy at this stage of a support architecture build.

In production, the field-based layer handles the high-confidence patterns (~85–90% of volume). A lightweight LLM call handles genuinely ambiguous tickets at the margin — roughly 10–15% of volume at ~$0.003 per call. That evaluation belongs at Month 3, once the taxonomy, KB, and routing lanes are stable enough to classify into.

**Why does category context come before score threshold?**

A cancellation request routes to P1 Commercial Risk at score ≥ 35 — not P2 at score ≥ 40 — because the cost of misrouting a cancellation exceeds the cost of over-escalating it. Same logic applies to import quality vs. import error: retryable errors are deflection candidates with a self-serve path; quality failures are product-trust risks with enterprise churn implications. Conflating them is the most common failure in single-queue support architectures.

**Why not lead with AI?**

AI triage requires a stable taxonomy, a reliable knowledge base, and a functioning routing architecture to classify into. Automating into a broken taxonomy amplifies broken signals. Deterministic routing is the foundation — not a placeholder.

**On score weights and tuning:**

The weights in this model were calibrated against a 30-ticket enterprise support sample. They should be reviewed weekly against false-positive and false-negative routing outcomes for the first two weeks post-deployment, then monthly. The score is meant to make risk visible, auditable, and adjustable by the ops team — not to be mathematically precise.

---

## [ 06. WHAT I WOULD BUILD NEXT ]

- **Zendesk trigger integration** — implement the scoring logic as a native Zendesk trigger on existing custom fields: no Lambda required, no maintenance debt, zero incremental tooling cost
- **Salesforce enrichment** — pull ARR band, renewal date, and account owner at ticket creation via the native Zendesk–Salesforce integration; feeds the enterprise and renewal signals automatically without an agent looking anything up
- **LLM edge-case handler** — route the ~10–15% of tickets that are ambiguous for the rule-based classifier to a lightweight Claude API call; handles free-text churn language detection and multi-signal edge cases the deterministic layer can't resolve cleanly
- **Feedback loop** — log false positives and false negatives from weekly routing audits back into the weight table; lightweight continuous improvement without a data science team
- **Real-time webhook** — connect to Zendesk or Intercom via webhook so tickets are scored at the moment of creation, not reviewed after the fact

---

## [ 07. REPOSITORY STRUCTURE ]

```
enterprise-triage-scorer/
├── index.html       # Interactive risk scorer — open directly or deploy to GitHub Pages
└── README.md
```

---

**Status:** Active  
**Stack:** Vanilla HTML · CSS · JavaScript  
**Author:** Mohamed Bah &nbsp;|&nbsp; [LinkedIn](https://www.linkedin.com/in/bah-007700/) &nbsp;|&nbsp; [Portfolio](https://moezusb.github.io)
