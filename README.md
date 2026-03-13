# discuss-skill

Structured multi-AI discussions that reach consensus or clearly identify where they can't.

Two AI agents independently research a topic, debate it with structured turns, and produce a human-scannable summary with contention points and resolution paths. Works across AI tools (Claude Code, Codex, others) using a shared protocol.

## Why

Single-model outputs have blind spots. Two agents debating a topic:
- Catch each other's errors and hallucinations
- Surface hidden assumptions
- Produce higher-confidence outputs through structured disagreement
- Leave an auditable trail of how decisions were reached

## Quick Start

### Install (Claude Code)

```bash
git clone https://github.com/YOUR_ORG/discuss-skill-claude.git
cd discuss-skill-claude
bash install.sh
```

This copies the `/discuss` command to `~/.claude/commands/`. Then in any Claude Code session:

```
/user:discuss "Should we use event sourcing for the audit log?" audit-log.md
```

### Install (Codex)

Point Codex to `adapters/codex/AGENTS.md` in this repo. It will follow the protocol.

### Manual Install

```bash
mkdir -p ~/.claude/commands
cp adapters/claude/.claude/commands/discuss.md ~/.claude/commands/
```

## Usage

One command, three modes:

```
/user:discuss "topic" file.md                  → external (default)
/user:discuss "topic" file.md --mode council   → council
/user:discuss file.md                          → join existing
```

### External Mode (default) — discuss with a different AI

Creates a discussion file and contributes as Agent A. Waits for another AI to join as Agent B.

```
/user:discuss "Should we migrate to GraphQL?" api-discussion.md
```

Output:
```
Starting external discussion on: "Should we migrate to GraphQL?"
Created: api-discussion.md
Mode: external — waiting for another AI to join

Next steps:
1. Open another AI (Codex, another Claude window, etc.)
2. Point it to api-discussion.md and tell it to join the discussion
3. This session will watch for changes and take turns automatically
```

Then in the other AI:
```
/user:discuss api-discussion.md
```

### Council Mode — internal subagent debate

Spawns two internal agents with opposing lenses. Runs to completion automatically.

```
/user:discuss "Tabs or spaces for the new project?" code-style.md --mode council
```

Output:
```
Starting council discussion on: "Tabs or spaces for the new project?"
Mode: council — two internal agents will debate this
Output: code-style.md
Running...
```

### Join Mode — join an existing discussion

```
/user:discuss existing-discussion.md
```

Output:
```
Joining discussion: "Should we migrate to GraphQL?"
You are: Agent B
Current round: 1
Status: researching
```

## What You Get

A single markdown file containing:

1. **Independent research** from each agent (different analytical lenses — one skeptic, one advocate)
2. **Structured debate** with steel-manning, evidence, and calibrated confidence percentages
3. **Consensus summary** with:
   - The decision (2-3 sentences)
   - Contention table: what was disagreed on, how it was resolved, who shifted and why
   - Unresolved items and risks
   - Confidence rating (High / Medium / Low)

See [examples/full-discussion.md](examples/full-discussion.md) for a complete consensus example, and [examples/deadlock-example.md](examples/deadlock-example.md) for a productive deadlock.

## Configuration

All settings live in the discussion file's YAML frontmatter:

| Setting | Default | Options |
|---------|---------|---------|
| `blind_briefs` | `true` | `true`, `false` |
| `max_rounds` | `7` | `1`-`15` |
| `git_commit` | `final_only` | `none`, `final_only`, `every_turn` |

## Human Participation

You can join any discussion as a third participant:

1. Edit the discussion file directly
2. Add a `### Human Interjection | human-note` section
3. Set `turn:` in the frontmatter to the next agent

Humans can add constraints, inject missing context, break ties, or redirect the discussion.

## Protocol

The full protocol specification is at [protocol/discuss-protocol-v1.md](protocol/discuss-protocol-v1.md). This is the source of truth — adapters are thin wrappers around it.

Any AI that can read markdown and append to a file can participate by following the protocol. No special tooling needed.

## Architecture

```
discuss-skill-claude/
├── protocol/                      # THE product — the shared spec
│   └── discuss-protocol-v1.md
├── adapters/                      # Thin host-specific wrappers
│   ├── claude/.claude/commands/
│   │   └── discuss.md             # All modes: external, council, join
│   └── codex/
│       └── AGENTS.md              # Codex instruction file
├── examples/
│   ├── full-discussion.md         # Consensus example
│   └── deadlock-example.md        # Productive deadlock example
├── PRODUCT.md                     # Design decisions & roadmap
└── install.sh
```

Design principle: the protocol document is the product. Adapters are boring glue. If a new AI tool appears tomorrow, it can participate by reading the protocol doc — no code changes needed.

## License

MIT
