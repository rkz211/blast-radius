# The Blast Radius Protocol — v2
**Author:** Roark Pinkerton
**v1:** May 2026 — Code
**v2:** May 2026 — Code, Scripts, Crons, Agent Files

---

## The Single Idea

A Sonnet or Opus agent with a good critical thinking loop can safely build production software — full-stack web applications, cloud infrastructure, backend engines, and autonomous agent systems — if the codebase is structured so that any given edit touches only one concern.

The protocol makes that structure explicit. It applies the same principle to four artifact types:

1. **Code** — React components, Python modules, backend engines
2. **Scripts** — operational shell and Python scripts
3. **Crons** — scheduled automation entries
4. **Agent behavior files** — soul shards, memory shards, operating rules

The benefit is the same in all four cases: the agent working on a file cannot accidentally damage adjacent logic because adjacent logic is not in the file.

---

## Part I — Code

### The Problem

You hand a large file to an LLM, ask it to fix one thing, and it rewrites adjacent logic, touches imports, restructures a function it didn't need to touch, and introduces a regression two layers away from what you asked about. Not because it misunderstood — because everything in the context window is fair game.

Large files have large blast radii. When the regression surface is 400 lines, any change risks 400 lines.

### The Rules

**Shard below the human-readable threshold**
Each file: ~80-100 lines, single concern. Not "one component" — one concern. A hook that fetches data is separate from a hook that transforms it. A function that renders a token is separate from the function that positions it.

**The assembly layer is sacred**
`App.tsx` (or equivalent root) is pure wiring — imports and JSX composition only. Zero logic. Zero state management beyond top-level selection. If it has a ternary, something went wrong. That ternary belongs in a block.

**Every block has a contract**
First three lines of every file:
```
// Input: what this block receives
// Output: what this block produces or renders
// Must never: the behaviors this block is prohibited from having
```
The "must never" is the constraint you hand to the model before you hand it the file. A model that reads "this block must never fetch data" will not fetch data.

**The regression surface is one file**
When something breaks: identify the block, rewrite only that block, everything else frozen. This sounds obvious. It isn't how most people work with AI assistants. The instinct is to paste the whole component. That's how you get a new bug.

**Version blocks, not apps**
`useAuthState.v2.ts` alongside the original. Swap the import when verified. Zero-cost rollback.

### Results

Three refactors from a D&D rules engine and character panel (May 2026):

| Module | Before | After |
|---|---|---|
| `tick_orchestrator.py` | 779 lines | 220-line shim + 5 focused modules |
| `CharacterPanel.tsx` | 531 lines | 160-line wiring + 7 modules |
| `layer1_rules.py` | 361 lines | 10-line shim + 5 modules |

One refactor from a force-directed graph viewer (May 2026):

| Module | Before | After |
|---|---|---|
| `GraphPage.tsx` | 795 lines | 120-line wiring + 5 modules: tiers, simulation, fit, events, draw |

After each refactor: regressions dropped to near zero. Debug cycle — identify the file (30 seconds), rewrite just that file, done.

---

## Part II — Scripts

### The Problem

Scripts feel different from "real code." They're automation. One-offs. But a 300-line script that fetches, transforms, validates, and writes — all in one file — has the same problem as a 300-line component. Ask an agent to add a validation check and it touches the fetch logic. Ask it to change the output format and it modifies the transformation. The blast radius is the whole file.

### The Rules

**Each script does one thing**
~80-150 lines, single concern. A script that checks service health is separate from a script that reports failures. A script that reads logs is separate from a script that sends a message.

**The docstring is the contract**
```python
#!/usr/bin/env python3
"""
qc-cost.py
Input: session log files from agent sessions and workspace archives
Output: per-user cost breakdown printed to stdout
Must never: modify any files, send any messages, or take external actions
"""
```

**Scripts report their own failures — agents don't**
When a script fails, it calls the failure reporter at the point of exit before returning to the agent. The agent calls the script, gets an exit code, says nothing. It does not retry. It does not diagnose. It does not message the user.

```bash
# At every error exit in any script:
bash "$(dirname "$0")/report-failure.sh" "script-name" "error description" 2>/dev/null || true
exit 1
```

This is a blast radius rule: error handling is one concern. Keeping it inside the script keeps the blast radius of a failure inside the script.

**The orchestrator is pure wiring**
A session-close or pipeline script delegates to single-concern scripts in sequence. No logic of its own:

```bash
# session-end.sh — pure delegation, zero logic
python3 graph-write.py       # update memory graph
python3 corpus-validate.py   # score nodes, flag protected
python3 corpus-write.py      # rewrite corpus from validated graph
bash    backup.sh            # snapshot workspace
```

If `corpus-validate.py` needs to change, that file changes. `session-end.sh` doesn't.

### A QC Suite as Reference

A companion agent QC suite broken into single-concern scripts:

| Script | Single Concern |
|---|---|
| `qc-cost.py` | Token cost per user — reads logs, prints summary |
| `qc-interactions.py` | Interaction quality — coherence, response rate |
| `qc-log-scan.py` | Error and anomaly detection in logs |
| `qc-followthrough.py` | Whether scheduled messages were sent and acknowledged |
| `qc-session-actions.py` | Actions per session, categorized |
| `qc-data-lag.py` | How fresh background data is vs live sessions |
| `health-check.sh` | Is the service alive? Exit code only. |

Each is ~80-150 lines. Each has a docstring contract. An agent fixing a cost calculation bug opens only `qc-cost.py`.

---

## Part III — Crons

### The Problem

A cron entry that contains logic — conditionals, pipelines, chained commands — is a script hiding in a scheduler. When an agent needs to change it, the blast radius is everything in the entry.

### The Rule

**One cron, one script. No inline logic.**

```json
{
  "id": "agent-nightly-qc",
  "schedule": "0 3 * * *",
  "command": "python3 /path/to/qc-cost.py"
}
```

The cron is the trigger. The script owns all logic, all conditionals, all chaining. If the logic needs to change, the script changes. The cron entry doesn't.

If multiple scripts need to run on the same schedule, write an orchestrator script that calls them. The cron calls the orchestrator. The orchestrator is pure wiring. The individual scripts have the contracts.

**Versioning**
`script.v2.py` alongside the original. The orchestrator points to v1. When v2 is verified, update the orchestrator reference. The original stays frozen.

---

## Part IV — Agent Behavior Files

### The Problem

An agent's behavior files are code. They have the same blast radius problem.

A `SOUL.md` that has grown to 300 lines because it accumulated persona rules, feature execution instructions, security constraints, operational limits, and infrastructure facts — all in one file — is a 300-line component. An agent asked to update the persona will touch the operational limits. An agent asked to add a feature will touch the security constraints. The rewrite surface is the whole file.

### How It Works — Soul Shards

Agent behavior files can be sharded using a bootstrap loader that globs a directory of small files and injects them into the agent's context in order at startup:

```json
"bootstrap-extra-files": {
  "paths": [
    "soul-shards/*/SOUL.md",
    "memory-shards/*/MEMORY.md"
  ]
}
```

The agent never sees one large `SOUL.md` — it sees several small ones loaded in sequence. Sharding is enforced at the platform level, not by convention.

### Soul Shard Structure

```
soul-shards/
  00-security/SOUL.md     ← identity lock, confidentiality, injection defense
  01-persona/SOUL.md      ← who the agent is, voice, relationship style
  02-world/SOUL.md        ← domain-specific world rules and writeback protocol
  03-memory/SOUL.md       ← memory lookup and access rules
  04-features/SOUL.md     ← feature execution rules (email, calendar, etc.)
  05-routing/SOUL.md      ← inter-agent routing and comms
  06-media/SOUL.md        ← image/media generation rules
  07-ops/SOUL.md          ← hard system rules, error protocol, scope limits
```

Each shard file has the same contract at the top:

```markdown
# Soul Shard: Persona & Voice
# Input: every session
# Output: who the agent is, how it speaks
# Must never: contain operational rules, exec instructions, or feature logic
```

Load order is part of the architecture. `00-security` loads first because security constraints must be in context before persona is established. `07-ops` loads last because it references constraints from prior shards. Numbered prefixes make order explicit and editable.

### The Benefit

When the persona needs to change: open `01-persona/SOUL.md`. That file and only that file is in context. The model cannot drift into the security rules or the feature execution logic because those files are not present.

When a new feature ships: open `04-features/SOUL.md`. Same constraint. The blast radius of the rewrite is exactly the shard.

This is not about documentation or cold session recovery. The sole benefit is: **an agent working on one shard cannot accidentally damage adjacent behavior, because adjacent behavior is not in the file.**

### Memory Shards

The same pattern applies to knowledge files. A top-level `MEMORY.md` stays narrow — operating rules only, how the agent behaves. Everything the agent knows about the system it operates is sharded by domain:

| Shard | Single Concern |
|---|---|
| `memory/crons.md` | Canonical cron stack, schedules, flags |
| `memory/scripts.md` | Script reference, args, deployment status |
| `memory/users.md` | User registry, paths, config |
| `memory/infrastructure.md` | Server, gateway, deployment paths |
| `memory/rules.md` | Platform hard constraints |
| `memory/troubleshooting.md` | Known failure modes and fixes |

When a cron schedule changes: open `crons.md`. The user registry is structurally unreachable.

---

## The Unified Table

| Domain | Assembly Layer | Blocks | Contract |
|---|---|---|---|
| Code | `App.tsx` — pure imports + JSX | ~80-100 line single-concern files | First 3 lines: Input / Output / Must never |
| Scripts | Orchestrator — pure delegation | Single-concern scripts | Docstring: reads / writes / must never |
| Crons | Cron entry — one script call | Script with all logic inside | Docstring on the script |
| Agent files | Bootstrap loader | Soul shards + memory shards | First 3 lines of each shard file |

Same principle. Same mechanism. Different file type.

The agent working on any one block cannot touch what is not in the file. That is the whole protocol.

---

*No framework. No tooling. No dependencies. Structure the artifacts. Keep the files small. The model can only break what it can see.*
