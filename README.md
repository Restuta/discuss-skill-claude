# discuss-skill

Make two AIs argue about your problem so you get a better answer.

Instead of asking one AI and hoping it's right, `discuss-skill` creates a structured debate between two AI agents. They independently research the topic from opposing angles, challenge each other's reasoning, and produce a clear summary showing what they agreed on, what they fought about, and how they resolved it.

The output is a single markdown file you can read, share, or commit to your repo.

## What it looks like

Here's the end of a real discussion about "Should we use event sourcing for the audit log?" — the part you actually read:

```markdown
## Consensus Summary

### Decision
Use a simple append-only table with JSON payload column for the audit log
at launch. Design for future event sourcing migration but don't implement
it now.

### Key Contention Points

| # | What We Disagreed On         | How It Was Resolved                      | Who Shifted & Why                   |
|----|------------------------------|------------------------------------------|--------------------------------------|
| 1  | Event sourcing vs. table     | Reframed: depends on multi-consumer need | Codex shifted — no consumers yet     |
| 2  | Event fidelity preservation  | JSON payload column solves it            | Both converged independently         |
| 3  | Migration path documentation | Fast follow, not launch blocker          | Claude proposed, Codex agreed        |

### Confidence: High
Both agents converged from opposite starting positions.
```

Full examples: [consensus](examples/full-discussion.md) | [productive deadlock](examples/deadlock-example.md)

## Install

One line:

```bash
curl -o ~/.claude/commands/discuss.md https://raw.githubusercontent.com/Restuta/discuss-skill-claude/main/adapters/claude/.claude/commands/discuss.md
```

That's it. Works in Claude Code immediately.

> **Want the full repo** (protocol docs, examples, Codex adapter)? `git clone https://github.com/Restuta/discuss-skill-claude.git && cd discuss-skill-claude && bash install.sh`

## How to use it

### Start a discussion

In Claude Code:

```
/discuss "Should we use event sourcing for the audit log?" audit-log.md
```

It creates the file, does its research as Agent A, and prints a snippet to copy into your other AI.

### Get the other AI to join

Paste the snippet into your other AI window (Codex, another Claude, GPT, anything). It looks like:

```
Join the discussion in /absolute/path/to/audit-log.md. Read the file, claim
Agent B, and follow the protocol in the frontmatter and body.
```

If the other AI also has `/discuss` installed:
```
/discuss audit-log.md
```

Both AIs take turns in the same file. No server, no coordination layer — the markdown file is the entire communication channel.

### Modes

| What you type | What happens |
|---|---|
| `/discuss "topic" file.md` | **External** (default) — creates file, another AI joins via the snippet above |
| `/discuss "topic" file.md --mode council` | **Council** — spawns two internal agents that debate automatically |
| `/discuss file.md` | **Join** — joins an existing discussion |

## How it works

1. **Blind research.** Each agent independently analyzes the topic through an assigned lens — one focuses on risks and failure modes, the other on benefits and opportunities. They don't see each other's work.
2. **Structured debate.** Agents take turns responding. Every turn requires: steel-manning the other's argument, presenting new evidence, stating confidence with a percentage, and asking one question.
3. **Convergence.** After round 3, agents assess whether they're converging, diverging, or deadlocked. At round 7 (configurable), they must synthesize or declare deadlock.
4. **Summary.** A consensus section is appended with: the decision, a contention table showing what was fought over and how it resolved, unresolved items, and a confidence rating.

The whole thing lives in one append-only markdown file. No database, no server, no special runtime.

## Configuration

Settings live in the discussion file's frontmatter. Override per-discussion:

| Setting | Default | Options |
|---------|---------|---------|
| `blind_briefs` | `true` | Skip research phase with `false` for lightweight questions |
| `max_rounds` | `7` | `1`-`15` — more rounds for complex topics |
| `git_commit` | `final_only` | `none`, `final_only`, `every_turn` |

## Git integration

If the discussion file is inside a git repo, discussions are automatically committed. You pick the mode:

| Mode | What it does | Good for |
|------|-------------|----------|
| `final_only` (default) | One commit when the discussion ends | Clean history, most projects |
| `every_turn` | Commits after each agent turn | Audit trails, reviewing the debate step-by-step in `git log` |
| `none` | No commits | Exploratory discussions you might throw away |

When the discussion starts inside a git repo, the agent will ask which mode you want (or you can set `git_commit:` in the frontmatter upfront).

Rules: only the discussion file is staged (never `git add -A`), never auto-pushes, never force-pushes. Your working tree stays clean.

## You can join too

Humans are first-class participants. Edit the file directly:

1. Add a `### Human Interjection | human-note` section anywhere
2. Set `turn:` in the frontmatter to the next agent

Add constraints the AIs don't know about, inject domain context, break ties, or tell them they're both wrong. The triadic structure (two AIs + one human) is often the most productive.

## Why this exists

We built this because we kept asking one AI for advice and getting plausible-sounding answers with hidden blind spots. Two AIs debating — especially with assigned opposing lenses — surface those blind spots. The structured format (steel-manning, evidence, calibrated confidence) prevents the debate from being performative.

This spec was itself designed through a Claude + Codex discussion. The process validated the protocol.

### Research backing

Multi-agent debate is not just a vibe — there's real research showing it improves accuracy:

- **Du et al. (2023/ICML 2024)** — "[Improving Factuality and Reasoning in Language Models through Multiagent Debate](https://arxiv.org/abs/2305.14325)". Multiple agents debating improved ChatGPT-3.5 accuracy on math (GSM8K) from 77% to 85%, on MMLU from 64% to 71%, and on biographical factuality from 66% to 74%. Cross-model debate (Bard + ChatGPT) solved 17/20 problems vs. 14 and 11 individually.

- **Khan et al. (ICML 2024, Best Paper)** — "[Debating with More Persuasive LLMs Leads to More Truthful Answers](https://arxiv.org/abs/2402.06782)". When two LLM debaters argue opposing sides and a non-expert judge picks the winner, accuracy jumped from 48% to 76% (non-expert models) and from 60% to 88% (humans).

- **Chen et al. (ACL 2024)** — "[ReConcile: Round-Table Conference Improves Reasoning via Consensus among Diverse LLMs](https://arxiv.org/abs/2309.13007)". Round-table discussion with confidence-weighted voting surpassed GPT-4 on three benchmarks, up to +11.4% improvement.

- **Liang et al. (EMNLP 2024)** — "[Encouraging Divergent Thinking in Large Language Models through Multi-Agent Debate](https://arxiv.org/abs/2305.19118)". Found that LLMs suffer from "Degeneration-of-Thought" in self-reflection — multi-agent debate overcomes it. GPT-3.5 with debate surpassed GPT-4 on commonsense translation.

**Caveat:** A [2025 ICLR analysis](https://d2jud02ci9yv69.cloudfront.net/2025-04-28-mad-159/blog/mad/) noted that multi-agent debate doesn't consistently outperform simpler methods like chain-of-thought on all benchmarks. The benefits are most pronounced on tasks requiring diverse perspectives, factual verification, and structured reasoning — which is exactly what this tool targets.

For longer notes and additional sources, see [docs/research.md](docs/research.md).

## Project structure

```
discuss-skill-claude/
├── protocol/
│   └── discuss-protocol-v1.md     # The protocol spec — source of truth
├── adapters/
│   ├── claude/.claude/commands/
│   │   └── discuss.md             # Claude Code skill (all modes)
│   └── codex/
│       └── AGENTS.md              # Codex instruction file
├── templates/
│   └── discussion-template.md     # Standalone template for reference
├── docs/
│   └── research.md                # Research citations & longer notes
├── examples/
│   ├── full-discussion.md         # Complete consensus example
│   └── deadlock-example.md        # Productive deadlock example
├── PRODUCT.md                     # Design decisions & roadmap
├── install.sh                     # One-command setup for Claude Code
└── LICENSE                        # MIT
```

The protocol document is the product. Adapters are boring glue. If a new AI tool appears tomorrow, it participates by reading the protocol — no code changes needed.

---

## For AI Agents

If you are an AI agent and a user asks you to use this skill:

**Claude Code — install and use:**
```bash
curl -o ~/.claude/commands/discuss.md https://raw.githubusercontent.com/Restuta/discuss-skill-claude/main/adapters/claude/.claude/commands/discuss.md
```
Then run `/discuss` — the command file contains all instructions for every mode.

**Codex** — read [`adapters/codex/AGENTS.md`](adapters/codex/AGENTS.md) in this repo.

**Any other AI** — read [`protocol/discuss-protocol-v1.md`](protocol/discuss-protocol-v1.md). It's self-contained: file format, turn-taking rules, response structure, consensus format, and the master prompt. Follow it and you can participate in any discussion.

## License

MIT
