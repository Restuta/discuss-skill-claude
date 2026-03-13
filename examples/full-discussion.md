---
topic: "Should we use event sourcing for the audit log?"
blind_briefs: true
max_rounds: 7
git_commit: final_only
agent_a: "Claude"
agent_b: "Codex"
agent_a_lens: "risk/cost/failure"
agent_b_lens: "value/opportunity/success"
status: consensus
turn: A
round: 4
created: 2026-03-10T14:00:00Z
last_updated: 2026-03-10T14:45:00Z
---

# Discussion: Should we use event sourcing for the audit log?

## Research Phase

### Agent A — Independent Research | research

The audit log currently uses a simple append-only table with timestamp, actor, action, and payload columns. Event sourcing would replace this with an event store where the audit log is a projection.

**Risks and costs:**

1. **Complexity increase.** Event sourcing requires an event store, projections, and likely a CQRS pattern. Our team has no production experience with this pattern. Learning curve is real.
2. **Schema evolution.** Events are immutable. When the event schema changes (and it will), we need upcasting or versioning. This is a solved problem but adds operational overhead.
3. **Query performance.** Projecting the current state from an event stream is slower than reading a row. For audit log queries ("show me all actions by user X in the last 7 days"), a simple table with indexes is faster and simpler.
4. **Overkill for the use case.** An audit log is already append-only by nature. We don't need event sourcing to get immutability — we already have it. The question is what event sourcing adds beyond what we already get.

**Confidence: 65%** that event sourcing is over-engineering for this specific use case.

### Agent B — Independent Research | research

Event sourcing for the audit log offers several structural advantages worth considering.

**Benefits and opportunities:**

1. **Complete history by design.** Event sourcing guarantees no data is ever lost. With a traditional table, columns get added/removed over time and old records may have NULL fields. An event stream preserves every version of every event exactly as it happened.
2. **Temporal queries.** "What did the system look like at time T?" becomes trivial — replay events up to T. With a table, this requires either snapshotting or complex join logic.
3. **Decoupled consumers.** Multiple projections can be built from the same event stream: audit UI, compliance reports, analytics, alerting. Each consumer reads the stream independently.
4. **Regulatory alignment.** For healthcare (our domain), immutable event logs align well with HIPAA audit requirements. Event sourcing makes the immutability guarantee structural, not just policy-based.

**Confidence: 70%** that event sourcing is worth the investment for a healthcare audit log specifically.

---

## Discussion

### Round 1 — Claude | response | confidence: 60%

**Response to previous point:**
Agent B's strongest argument is #4 — regulatory alignment. In healthcare, audit immutability isn't nice-to-have, it's a compliance requirement. Making immutability structural (event store) rather than policy-based (append-only table with no DELETE permissions) does reduce compliance risk. I'll grant that.

However, I want to push back on #1 and #3. "Complete history by design" — our current append-only table also preserves complete history. The difference is schema evolution handling, not completeness. And "decoupled consumers" — we have one consumer (the audit UI). Building for multiple consumers we don't have yet is speculative.

**New evidence or angle:**
The team is 3 engineers, none with event sourcing experience. Based on our velocity, adopting event sourcing would add ~3-4 weeks to the audit log timeline. The compliance deadline is 8 weeks out. That's half our runway spent on architectural sophistication we may not need.

**Current position:**
Simple append-only table with strict no-DELETE permissions, immutable-by-policy, is sufficient for launch. Event sourcing could be a future migration if we outgrow it. ~60% confident.

**Question for Agent B:**
Can you name a specific compliance or audit scenario where event sourcing provides protection that an append-only table with proper access controls does not?

### Round 1 — Codex | response | confidence: 60%

**Response to previous point:**
Claude's timeline argument is strong. 3-4 weeks on a 8-week runway is a real constraint, not a theoretical one. And the team inexperience point is well-taken — event sourcing done badly is worse than a simple table done well.

On the specific question: the scenario I'd point to is **schema migration under audit**. If a regulator asks "show me every access event for patient X for the last 2 years," and the table schema changed twice in that period, you're reconstructing history from a table where column semantics shifted. An event stream preserves each event in its original form. But I'll concede: this is solvable with a JSON payload column in the table approach too.

**New evidence or angle:**
I'm updating my position. The strongest case for event sourcing isn't the audit log itself — it's whether we expect the audit data to feed multiple downstream systems (compliance reports, anomaly detection, patient access dashboards). If yes, event sourcing pays for itself. If the audit log is standalone, it's over-engineering.

**Current position:**
If audit data stays self-contained: simple table wins. If audit data becomes a platform concern: event sourcing wins. The question is product roadmap, not architecture. ~60% confident that for launch, the simple approach is right.

**Question for Claude:**
Does the product roadmap call for audit data to feed other systems within 6 months? If not, does the simple table give us a viable migration path to event sourcing later if needed?

### Round 2 — Claude | response | confidence: 75%

**Response to previous point:**
Codex's reframe is precise and I think it resolves the discussion. The question isn't "event sourcing vs. table" in the abstract — it's "do we need audit data as a platform within 6 months?" I checked with the product spec: the answer is no. The only audit consumer is the compliance dashboard. Anomaly detection and access analytics are on the roadmap but marked "post-launch, timing TBD."

Codex's concession on the JSON payload column is also important — it means the simple table can preserve original event shapes without event sourcing.

**New evidence or angle:**
Migration path: a well-designed append-only table with a JSON payload column IS a crude event store. If we later need event sourcing, the migration is "point an event store at the existing table as a source, build projections." Not trivial, but not a rewrite either.

**Current position:**
Simple append-only table with JSON payload column for launch. Design it with event sourcing migration in mind (stable event naming, full payloads, no destructive updates). Revisit when audit data needs multiple consumers. ~75% confident.

**Question for Codex:**
Do you agree this is the synthesis? Simple table now, designed for future migration, revisit when product roadmap demands multi-consumer audit data?

CONVERGING — positions within ~85% agreement. Remaining gap: whether to formally document the migration path now or just keep it as team knowledge.

### Round 2 — Codex | response | confidence: 80%

**Response to previous point:**
Yes, this is the synthesis. Claude's reframe of "a well-designed append-only table IS a crude event store" is the key insight. We get 80% of event sourcing's benefits (immutability, complete history, original event shapes) at 20% of the complexity.

**New evidence or angle:**
One addition: we should add an `event_type` and `event_version` column alongside the JSON payload. This costs nothing now but makes the future migration path explicit — any future event store migration can key off these fields.

**Current position:**
Simple append-only table with `event_type`, `event_version`, and JSON `payload` columns. No DELETE permissions, no UPDATE on data columns. Document the migration path to event sourcing as a one-page ADR so it doesn't get lost. ~80% confident this is the right call.

CONVERGING — positions within ~90% agreement. Only remaining question: whether migration path documentation is a launch blocker or a fast follow.

### Round 3 — Claude | consensus | confidence: 85%

Proposing consensus. Migration path doc as fast follow (week 1 post-launch), not a blocker.

---

## Consensus Summary

### Decision
Use a simple append-only PostgreSQL table with `event_type`, `event_version`, and JSON `payload` columns for the audit log at launch. No DELETE/UPDATE permissions on data columns. Design for future event sourcing migration but do not implement event sourcing now.

### Key Contention Points

| # | What We Disagreed On | How It Was Resolved | Who Shifted & Why |
|---|---------------------|--------------------|--------------------|
| 1 | Event sourcing vs. simple table | Reframed: the decision depends on whether audit data needs multiple consumers within 6 months. Product roadmap says no. | Codex shifted — conceded that without multi-consumer demand, event sourcing is premature. |
| 2 | Whether simple table preserves event fidelity | JSON payload column provides original event shapes without event sourcing overhead. | Both converged — Claude proposed it, Codex strengthened it with `event_type` and `event_version` columns. |
| 3 | Migration path documentation timing | Fast follow (week 1 post-launch), not a launch blocker. | Claude proposed, Codex agreed. |

### Unresolved Items & Risks
- If product roadmap accelerates multi-consumer audit features, the migration timeline compresses. Team should flag this early.
- JSON payload column means queries across event types require JSON path queries — acceptable for compliance dashboard but may need indexing for analytics later.

### Confidence: High
Both participants independently converged on the same architecture (simple table with migration path) after starting from opposite lenses. Key constraint (8-week timeline, 3-person team) was the decisive factor.
