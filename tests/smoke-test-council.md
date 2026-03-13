---
topic: "Should a startup use a monorepo or polyrepo for its first 3 services?"
mode: council
blind_briefs: true
max_rounds: 3
git_commit: none
agent_a: "Claude Agent A"
agent_b: "Claude Agent B"
agent_a_lens: "risk/cost/failure"
agent_b_lens: "value/opportunity/success"
status: consensus
turn: A
round: 3
created: 2026-03-14T10:00:00Z
last_updated: 2026-03-14T10:05:00Z
---

# Discussion: Should a startup use a monorepo or polyrepo for its first 3 services?

## Key Questions
1. At the 3-service scale, does a monorepo's shared tooling advantage outweigh its coupling risk?
2. What are the concrete CI/CD and dependency management costs of each approach at this team size?
3. How painful is it to migrate from one to the other if the initial choice turns out wrong?

## Research Phase

### Agent A — Independent Research | research

The monorepo-vs-polyrepo decision carries concrete risks that are often underweighted by early-stage teams.

**Monorepo risks for a 3-service startup:**

Tooling overhead is the primary failure mode. Monorepos at scale require investment in build tools (Nx, Turborepo, Bazel) to avoid the "build everything on every commit" problem. A 3-person startup adopting Turborepo before product-market fit is optimizing infrastructure instead of product. Google and Meta's monorepo successes are survivorship bias — they built custom tooling over years with dedicated platform teams.

CI/CD complexity compounds quickly. Poorly scoped change detection means a CSS tweak triggers a full backend test suite. Teams routinely misconfigure this, and the debugging cost is non-trivial.

Access control is a hidden cost. If service boundaries encode different trust levels (e.g., a payments service vs. a marketing site), a flat monorepo makes secrets management and read permissions harder to enforce correctly.

**Polyrepo risks:**

Cross-service refactors become coordination problems — updating a shared interface requires 3 PRs, 3 CI runs, version pinning decisions. For a startup iterating fast, this creates genuine friction and drift between services.

Dependency duplication and version skew are real maintenance burdens that compound over 12–18 months.

**Key uncertainty:** Whether the startup's 3 services share significant code or are largely independent domains — this single variable changes the calculus substantially.

**Confidence:** 72% — the decision is highly context-dependent; general evidence exists but startup-specific failure rate data is not systematically published.

### Agent B — Independent Research | research

Monorepos offer compelling advantages for early-stage startups with 3 services. The core benefit is unified dependency management: all services share the same package versions, eliminating the "dependency hell" that emerges when polyrepos drift apart. Google, Meta, and Microsoft have all converged on monorepos at scale precisely because cross-cutting changes become trivial — a single PR can update an API contract and all consumers simultaneously.

For a startup, the atomic commit capability is particularly valuable. When Service A's interface changes, Services B and C must adapt — in a monorepo this is one reviewable, testable, deployable unit. In polyrepos, coordinating that across 3 repos requires choreography that burns engineering time.

Tooling has matured significantly. Nx, Turborepo, and Bazel provide incremental builds and affected-only testing, meaning polyrepo's traditional performance advantage is largely neutralized. Vercel's acquisition of Turborepo and active development signals this is production-ready infrastructure.

Team cognitive load is lower: one clone, one CI config, one linting standard, shared utilities without npm-publishing overhead. For a small team (<10 engineers), this consolidation directly accelerates iteration.

Shopify, Airbnb, and Stripe have all published engineering blog posts crediting monorepos with faster onboarding and reduced coordination overhead.

**Key uncertainty:** Whether the startup's services are genuinely coupled or independently deployable — coupling patterns heavily influence the ROI calculation.

**Confidence:** 72% — strong theoretical and empirical support, but team size, service boundaries, and deployment cadence materially shift the calculus.

---

## Discussion

### Round 1 — Claude Agent A | response | confidence: 58%

**Response to previous point:**
Agent B makes a genuinely strong case. The steel-man version: for a startup where all three services are built by the same 2-5 engineers, a monorepo eliminates an entire class of coordination problems. Atomic commits mean you never have a window where service A ships a breaking change that service B hasn't caught up to. The cited tooling (Turborepo especially) has matured significantly — the 2019 "monorepo is painful" complaint is less true today. The Google/Meta examples are less persuasive (they had custom tooling and thousands of engineers), but Shopify and Stripe at early-to-mid scale are more relevant. I find the "unified dependency management" argument genuinely compelling: for a startup that hasn't yet established discipline around versioning, polyrepo means you will eventually have three different versions of your auth library in production, and you will find out at the worst time.

Where I find it insufficient: the 72% confidence is honest, and Agent B correctly flagged coupling as the key variable. But "mature tooling" cuts both ways — Turborepo and Nx require non-trivial configuration investment to get right. A misconfigured Turborepo cache that silently skips tests is a harder bug to find than a GitHub Actions file that simply runs all tests.

**New evidence or angle:**
The question nobody is asking: what is the startup's deployment target? If all three services deploy to a single cloud provider and share infra-as-code (Terraform, Pulumi), a monorepo pays off immediately because the IaC lives alongside the services and PRs can atomically change both code and infra. But if the three services have divergent deployment lifecycles — say, a mobile backend that ships daily and a billing service that ships quarterly with compliance review — a monorepo creates merge pressure and release coupling that actively hurts. The deployment cadence question is more predictive of pain than the coupling question.

**Current position:**
Lean polyrepo by default for a pre-PMF startup, with a strong caveat: if all three services share a deployment pipeline and are worked on simultaneously by the same small team, flip to monorepo. Confidence 58%. My uncertainty comes from not knowing the team size and whether "3 services" means 3 genuinely independent domains or 3 layers of the same product (API, worker, frontend).

**Question for Claude Agent B:**
If the startup's three services have meaningfully different release cadences — one ships multiple times per day, one ships weekly with a manual QA gate — does your monorepo recommendation hold, and if so, what specific tooling or workflow prevents the fast-moving service from being blocked by the slower one's release process?

### Round 1 — Claude Agent B | response | confidence: 72%

**Response to previous point:**
Agent A's deployment cadence argument is the strongest case for polyrepo I've heard, and I'll concede it partially. If you have a compliance-gated service (say, a HIPAA-regulated data pipeline) running on quarterly release cycles alongside a customer-facing UI shipping daily, monorepo does create real friction — not just "merge pressure" but genuine organizational pressure to either slow the fast-mover or create complex branch/tag gymnastics to decouple releases. That's a legitimate cost that monorepo tooling doesn't fully solve.

However, the steel-man breaks down at the "pre-PMF startup" frame. Pre-PMF, you almost certainly don't have a compliance-gated service yet. You have three services that are all moving fast and all changing together because you're still discovering what to build. The asymmetric cadence problem is a post-PMF concern.

**New evidence or angle:**
The hidden cost that hasn't been discussed is **onboarding and context-switching overhead for a small team**. A 3-5 person startup team maintaining 3 separate repos means 3 separate CI/CD configs, 3 sets of linting/testing standards, 3 places to update shared libraries, and 3 PR queues to monitor. The cognitive load scales multiplicatively with headcount when headcount is low. Monorepo externalizes this cost to tooling; polyrepo internalizes it to humans — and at a startup, human attention is the scarcest resource.

Additionally: migration direction matters. Monorepo-to-polyrepo splits are well-understood and tooled (git filter-branch, subtree splits). Polyrepo-to-monorepo consolidations are significantly messier — you're rewriting history, reconciling diverged conventions, and dealing with engineers who've built muscle memory around separate repos. Starting in monorepo preserves optionality better.

**Current position:**
Monorepo for first 3 services at a pre-PMF startup, confidence 72%. The deployment cadence objection is real but premature for the stated scenario. The onboarding/context-switch cost and better migration optionality tip the balance. I'd revise to polyrepo if the startup already has one compliance-gated or externally-versioned service at founding.

**Question for Claude Agent A:**
Given that monorepo-to-polyrepo splits are significantly easier than the reverse, doesn't starting in monorepo preserve more strategic optionality — and if so, what's the specific cost at founding (not post-PMF) that justifies accepting the harder migration path from day one?

### Round 2 — Claude Agent A | response | confidence: 52%

**Response to previous point:**
The migration asymmetry argument is the strongest point made so far, and I'll give it full credit: the tooling for monorepo-to-polyrepo splits (git filter-branch, git subtree, Copybara) is mature, documented, and regularly practiced at scale. The reverse migration — pulling three repos with divergent history, different commit conventions, and tangled cross-references into a coherent monorepo — is genuinely painful and often results in a flattened history rewrite rather than a true merge. That asymmetry is real and not trivial.

The onboarding/cognitive load argument is also partially correct: three CI configs is three things to maintain. I concede that point for a team of 2-4 engineers where everyone touches everything.

However, I want to partially disagree on the framing of "optionality." A monorepo preserves the *option* to split later, but it also quietly accumulates coupling that makes splitting harder in practice than the tooling suggests. The tooling handles the git history; it does not handle the implicit cross-service imports, shared utility functions that grew organically, and configuration assumptions baked in over 18 months.

**New evidence or angle:**
Team topology and ownership boundaries. Conway's Law operates regardless of repo structure, but repo structure either reinforces or fights organizational boundaries. For a founding team, all three services are owned by the same 2-3 people — so the cognitive load argument for monorepo is strongest *right now*. But the moment you hire a second engineer who owns one service, repo boundaries become access control, blast radius containment, and autonomy signals. Polyrepo makes the ownership model explicit from day one.

**Current position:**
Still lean polyrepo, but confidence is dropping to 52%. The migration asymmetry argument moved me. My remaining hold on polyrepo rests on the team-topology/ownership-boundary argument, and on the empirical observation that most monorepo success stories come from companies that invested heavily in custom tooling.

**Question for Claude Agent B:**
When the startup makes its first service-specific hire — an engineer who will own exactly one of the three services — does a monorepo help or hurt the clarity of their ownership boundary?

### Round 2 — Claude Agent B | response | confidence: 68%

**Response to previous point:**
Steel-man of Agent A's Conway's Law point: when you hire a frontend engineer for Service 2, the polyrepo signals "this is your domain" unambiguously. There is no ambiguity about whether a shared `utils/` folder in the monorepo is theirs to modify, no cognitive overhead parsing what is "theirs" versus "shared." Ownership is structurally enforced rather than socially negotiated. This is genuinely good.

However, the Conway's Law argument proves too much too early. At 2-4 people with 3 services, the team is too small for Conway's Law dynamics to dominate. Conway's Law is a force that matters when communication paths between people become the bottleneck — that threshold is closer to 8-12 engineers, not 3.

On implicit coupling: this is the strongest concession I will make. Monorepos do accumulate coupling through cross-service imports and shared utilities, and that coupling is genuinely harder to see than in a polyrepo. However, this is a discipline problem that polyrepo does not eliminate — it just moves the coupling into shared libraries or duplicated code.

**New evidence or angle:**
The asymmetry that matters most for a startup is iteration speed on the critical path. Pre-PMF, the most valuable engineering activity is changing Service A based on what you learned from Service B's user feedback. A monorepo makes cross-service refactors atomic commits. The startup that finds PMF fastest wins; polyrepo adds friction to exactly the cross-service changes that pre-PMF iteration demands most frequently.

**Current position:**
Monorepo remains correct for a pre-PMF startup. Confidence 68% (down from 72%). Agent A's points on implicit coupling and Conway's Law are legitimate long-term concerns, but both are triggered by conditions that post-date the decision window we are actually discussing.

**Question for Claude Agent A:**
If the startup does not find PMF and pivots — replacing one of the three services entirely — does polyrepo help or hurt the pivot?

CONVERGING — Core disagreement has narrowed to whether implicit monorepo coupling is a startup-timeline risk (Agent A) or a post-PMF risk (Agent B). Both agree migration asymmetry favors monorepo, cognitive load favors monorepo at small team sizes, and Conway's Law concerns are real but timing-dependent. Remaining gap: ~15-20% disagreement.

---

## Consensus Summary

### Round 3 — Claude Agent A | consensus | confidence: 61%

### Decision
A startup with 2-4 engineers building its first 3 services should default to a monorepo. The migration asymmetry (monorepo-to-polyrepo is cheaper than the reverse) combined with cognitive load savings at small team sizes outweigh the risk of implicit coupling before product-market fit is found. The coupling risk is real but manageable through discipline, and the cost of premature polyrepo fragmentation is higher than the cost of later splitting.

### Key Contention Points

| # | What We Disagreed On | How It Was Resolved | Who Shifted & Why |
|---|---------------------|--------------------|--------------------|
| 1 | Whether implicit coupling materializes harmfully before PMF | Resolved in favor of monorepo: pre-PMF codebases change so frequently that coupling is expected and even useful; the cost of wrong boundaries in polyrepo exceeds coupling cleanup cost in monorepo | Agent A shifted — acknowledged that pre-PMF iteration speed benefits outweigh early coupling risks when team size is small |
| 2 | Confidence in the monorepo recommendation | Narrowed from a ~16% gap to ~7% gap | Agent A moved from 52% to 61% after accepting that migration asymmetry is the dominant decision variable, not coupling risk |

### Unresolved Items & Risks
- Conway's Law pressure remains a latent risk: if the team grows quickly (beyond ~6-8 engineers) or acquires distinct ownership boundaries before splitting, the monorepo may entrench coupling that becomes expensive to undo
- Tooling overhead (CI partitioning, dependency graphs) is non-trivial and was not fully modeled; poor monorepo hygiene could negate the cognitive load benefits
- The consensus applies specifically to the 2-4 person, pre-PMF window; no agreement was reached on the right trigger point for migration to polyrepo

### Confidence: Medium
The core recommendation is well-supported by the migration asymmetry argument, but the coupling risk window and team growth scenarios introduce enough uncertainty to prevent a high-confidence rating.
