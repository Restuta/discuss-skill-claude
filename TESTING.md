# Testing discuss-skill

## What to test

The skill is a markdown command file that instructs an AI agent. There's no runtime code to unit test. Testing means running `/discuss` in different modes and verifying the output matches the protocol.

## Manual test matrix

### 1. Council mode (fastest — self-contained)

```
/discuss "Should a small CLI tool use env vars or a config file?" test-council.md --mode council
```

**Verify:**
- [ ] File created with correct frontmatter (all fields present, correct defaults)
- [ ] Key Questions section generated with 2-3 concrete sub-questions
- [ ] Blind research phase: two agents, different lenses (risk vs. value)
- [ ] Research entries have: analysis, key uncertainty, confidence %
- [ ] Discussion rounds: steel-manning, new evidence, position, question
- [ ] Round 3+: convergence assessment (CONVERGING/PARALLEL/DIVERGING/DEADLOCKED)
- [ ] Consensus summary has: decision, contention table, unresolved items, confidence
- [ ] Frontmatter updated: status → consensus/deadlock, round reflects final state
- [ ] Git commit created (if git_commit: final_only)

### 2. External mode — start

```
/discuss "Should we use a monorepo?" test-external.md
```

**Verify:**
- [ ] File created with correct frontmatter (agent_b: "unassigned")
- [ ] Key Questions generated
- [ ] Agent A blind research written
- [ ] Copy-paste snippet printed with absolute file path
- [ ] turn: B in frontmatter after research
- [ ] Agent starts polling for changes

### 3. External mode — join

```
/discuss test-external.md
```

(Run in a second Claude Code window after step 2)

**Verify:**
- [ ] Reads file, claims Agent B slot
- [ ] Re-reads to confirm no collision
- [ ] Writes Agent B blind research with value/opportunity lens
- [ ] Updates status: discussing, round: 1, turn: A
- [ ] Begins polling for Agent A's response

### 4. Edge cases

**blind_briefs: false**
- Manually create a file with `blind_briefs: false` and verify no research phase, jumps straight to discussion.

**Human interjection**
- During a discussion, manually add a `### Human Interjection | human-note` entry and verify the next agent acknowledges it.

**Max rounds forced synthesis**
- Set `max_rounds: 2` and verify consensus is forced after round 2.

**Join when both slots taken**
- Both agent_a and agent_b are filled, join should say "Both participant slots are taken."

## Automated testing

True automated integration tests are hard because the "runtime" is an AI following instructions. Possible approaches:

1. **Snapshot testing**: Run council mode, save the output, verify frontmatter and section structure with a simple script that checks for required headings and YAML fields.
2. **Protocol compliance checker**: A script that reads a discussion file and validates it against the protocol (correct headings, required sections in responses, valid frontmatter state transitions). This would be useful for catching adapter drift.

Neither exists yet. For now, manual testing with the matrix above is the way.

## Quick smoke test

The fastest way to verify everything works:

```
/discuss "Tabs vs spaces for a new Python project?" smoke-test.md --mode council
```

Then check:
1. Does the file have valid frontmatter?
2. Does it have Key Questions?
3. Does it have research from both agents?
4. Does it have at least 2 discussion rounds?
5. Does it end with a consensus summary?
6. Is the consensus scannable and actionable?
