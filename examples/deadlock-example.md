---
topic: "Should we migrate from REST to GraphQL for the patient-facing API?"
blind_briefs: true
max_rounds: 7
git_commit: none
agent_a: "Claude"
agent_b: "Codex"
agent_a_lens: "risk/cost/failure"
agent_b_lens: "value/opportunity/success"
status: deadlock
turn: A
round: 5
created: 2026-03-11T09:00:00Z
last_updated: 2026-03-11T10:20:00Z
---

# Discussion: Should we migrate from REST to GraphQL for the patient-facing API?

## Research Phase

### Agent A — Independent Research | research

**Risks and costs of migrating to GraphQL:**

1. **Migration scope.** 23 REST endpoints serve the patient app. Each must be reimplemented as GraphQL resolvers, tested, and deployed. Minimum 4-6 weeks of work.
2. **Team expertise.** 2 of 3 backend engineers have zero GraphQL experience. Learning curve during a critical delivery period.
3. **N+1 query problem.** Without careful DataLoader implementation, GraphQL resolvers will generate significantly more database queries than the current REST endpoints.
4. **Caching complexity.** REST responses cache trivially with HTTP caching (ETags, Cache-Control). GraphQL POST requests require application-level caching (persisted queries, response normalization).
5. **Monitoring gaps.** Our current observability stack (Datadog APM) has first-class REST support. GraphQL requires custom instrumentation for per-resolver tracing.

**Confidence: 70%** this migration is not justified at this stage.

### Agent B — Independent Research | research

**Benefits of migrating to GraphQL:**

1. **Mobile performance.** The patient app currently makes 3-5 REST calls per screen. GraphQL would reduce this to 1 request per screen, cutting latency by 40-60% on cellular connections.
2. **Frontend autonomy.** The mobile team currently files backend tickets for every new data shape they need. GraphQL lets them query exactly what they need without backend changes.
3. **Type safety.** GraphQL schema generates TypeScript types automatically. The current REST API has no generated types — the mobile team maintains manual type definitions that drift from reality.
4. **Future API surface.** We're adding provider-facing and payer-facing apps within 12 months. GraphQL's single schema serves all three audiences with field-level permissions, vs. maintaining 3 separate REST APIs.

**Confidence: 75%** the migration is worth the investment for the 12-month horizon.

---

## Discussion

### Round 1 — Claude | response | confidence: 65%

**Response to previous point:**
Agent B's mobile performance argument (#1) is the strongest point. 3-5 REST calls per screen on cellular is genuinely painful, and reducing to 1 round-trip is measurable user-facing improvement. I'll grant that this is a real problem that needs solving.

However, the 40-60% latency improvement claim is speculative without measurement. GraphQL requests are larger and more complex to parse server-side. The improvement might be 20% or it might be 50% — we don't know.

**New evidence or angle:**
There's a third option neither of us addressed: **BFF (Backend for Frontend) pattern.** A thin aggregation layer that combines multiple REST calls into single endpoints specifically shaped for each mobile screen. Gets the "one request per screen" benefit without rewriting the entire API layer.

**Current position:**
BFF solves the mobile performance problem at ~20% of the migration cost. GraphQL may still be the right long-term answer, but BFF is a better next step. ~65% confident.

**Question for Agent B:**
Does the BFF pattern address your mobile performance concern? If so, does GraphQL still justify its migration cost?

### Round 1 — Codex | response | confidence: 65%

**Response to previous point:**
The BFF proposal is reasonable for the performance concern in isolation. But it doesn't address frontend autonomy (#2) or type safety (#3). A BFF is another backend surface to maintain — every new screen requires a new BFF endpoint, which recreates the "mobile team files backend tickets" problem.

The BFF also doesn't help with the multi-app future (#4). We'd need a BFF per client app, which is 3 aggregation layers to maintain.

**New evidence or angle:**
Incremental migration is possible. We don't have to rewrite all 23 endpoints at once. We can run GraphQL alongside REST, migrate screen-by-screen, and deprecate REST endpoints as we go. Apollo Router supports this pattern. This addresses Claude's scope concern.

**Current position:**
Incremental GraphQL migration alongside REST, starting with the highest-traffic patient screens. Not a full rewrite. ~65% confident.

**Question for Claude:**
Does incremental migration (GraphQL + REST coexisting) change your cost/risk assessment? The scope drops from "rewrite 23 endpoints" to "add GraphQL for 3-4 key screens first."

### Round 2 — Claude | response | confidence: 55%

**Response to previous point:**
Incremental migration does improve the risk profile. "Add GraphQL for 3-4 screens" is a 2-week experiment, not a 6-week rewrite. That's a reasonable investment to validate the benefits.

But I have a deeper concern: running GraphQL + REST simultaneously is not free. It means maintaining two API paradigms, two sets of auth middleware, two monitoring configurations, two documentation systems. The "incremental" framing undersells the operational cost of dual-stack.

**New evidence or angle:**
The real question might be timing, not technology. GraphQL is probably right for this product eventually. The question is whether migrating now (with 3 engineers, pre-launch, no GraphQL expertise) is better than migrating in 6 months (with presumably more engineers, post-launch stability, and time to learn).

**Current position:**
I'm genuinely uncertain. Both paths have real costs. ~55% confident in "wait 6 months" but I can see the other side. This might be a values/priorities question more than a technical one.

**Question for Codex:**
If we wait 6 months, what's the concrete cost? Is the mobile performance pain bad enough that waiting 6 months materially hurts user retention or acquisition?

### Round 2 — Codex | response | confidence: 60%

**Response to previous point:**
Claude's timing reframe is fair. The dual-stack operational cost is real and I underweighted it.

On the concrete cost of waiting: the mobile team reports that the multi-request pattern causes visible loading spinners on ~15% of page transitions. We don't have data on whether this affects retention, but it's a known UX pain point. Whether it's "6-month urgent" or "nice to fix" depends on growth targets we don't have visibility into.

**New evidence or angle:**
There's a middle ground I haven't proposed: **fix the mobile performance problem now with BFF, AND budget GraphQL migration for Q3 when we'll have more engineers and post-launch stability.** This gets the immediate UX win without the dual-stack risk.

**Current position:**
I'm split. BFF-now-GraphQL-later is pragmatic but means building a BFF we'll throw away in 6 months. GraphQL-now is riskier but avoids throwaway work. ~60% confident in BFF-first, but the throwaway cost bothers me.

DIVERGING — core disagreement on whether BFF is pragmatic or wasteful. Claude's crux: team capacity and timing. My crux: throwaway work and compounding tech debt.

### Round 3 — Claude | response | confidence: 55%

**Response to previous point:**
Codex's honesty about the throwaway concern is well-placed. Building a BFF we discard in 6 months is ~2 weeks of engineer time wasted. That's real. But the alternative (GraphQL now) risks 4-6 weeks of distraction during pre-launch, with a team that's never done it.

**New evidence or angle:**
Nothing new. I keep circling the same tradeoff: short-term pragmatism (BFF) vs. long-term correctness (GraphQL). The answer depends on information we don't have: growth targets, hiring timeline, and whether the mobile UX pain is retention-affecting.

**Current position:**
Still ~55% on BFF-first. But I acknowledge this is a judgment call, not a technical analysis. The right person to make this call is the product lead, with input on growth targets.

DEADLOCKED — we agree on the technical tradeoffs but disagree on the timing/priority weighting. The crux is business context (growth targets, hiring plan) that neither of us has.

### Round 3 — Codex | response | confidence: 55%

**Response to previous point:**
Agreed that we've reached the limit of what technical analysis can resolve. We both understand the tradeoffs. The remaining disagreement is about business priorities we can't evaluate.

DEADLOCKED — same assessment as Claude. Recommending human review with specific decision criteria.

---

## Consensus Summary

### Decision
**DEADLOCKED.** Both participants agree GraphQL is likely the right long-term architecture. The disagreement is on timing: migrate now (higher risk, no throwaway work) vs. BFF now + GraphQL in Q3 (lower risk, ~2 weeks throwaway work). The decision depends on business context neither participant has.

### Key Contention Points

| # | What We Disagreed On | How It Was Resolved | Who Shifted & Why |
|---|---------------------|--------------------|--------------------|
| 1 | REST vs. GraphQL (original framing) | Reframed to timing question — both agree GraphQL is right eventually. | Both shifted — Claude proposed BFF as third option, Codex proposed incremental migration. |
| 2 | BFF vs. incremental GraphQL | **UNRESOLVED.** BFF is safer but creates throwaway work. Incremental GraphQL avoids waste but adds dual-stack operational cost. | Neither shifted — genuine tradeoff with no clear winner without business context. |
| 3 | Whether mobile UX pain is urgent | **UNRESOLVED.** 15% of page transitions show spinners, but retention impact is unknown. | Both acknowledged the gap — no data available. |

### Unresolved Items & Risks
- Growth targets and hiring timeline determine the right answer — product lead should weigh in
- If mobile retention data shows spinner-correlated churn, urgency increases and GraphQL-now becomes more justified
- If Q3 hiring doesn't materialize, the "migrate later with more engineers" assumption breaks

### Confidence: Medium
Strong technical analysis from both sides. Deadlock is genuine and well-reasoned — not a failure of discussion but an accurate identification that the decision requires business input. The human should decide based on: (1) Is 15% spinner rate affecting retention? (2) Will Q3 headcount increase happen? (3) How much pre-launch risk is acceptable?
