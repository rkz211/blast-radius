# The Blast Radius Protocol — v4
**Author:** Roark Pinkerton
**v1:** May 2026 — Code
**v2:** May 2026 — Code, Scripts, Crons, Agent Files
**v3:** June 2026 — Unified: Hologram Pyramid, garbage collection, sharding/coupling tradeoff
**v4:** June 2026 — Verification methods, assembly scaling, threshold honesty, orphan detection

---

## The Single Idea

A Sonnet or Opus agent with a good critical thinking loop can safely build production software — full-stack web applications, cloud infrastructure, backend engines, and autonomous agent systems — if every artifact is structured so that any given edit touches only one concern.

The protocol makes that structure explicit. It applies the same principle to four artifact types:

1. **Code** — React components, Python modules, backend engines
2. **Scripts** — operational shell and Python scripts
3. **Crons** — scheduled automation entries
4. **Agent behavior files** — soul shards, memory shards, operating rules

The benefit is the same in all four cases: the agent working on a file cannot accidentally damage adjacent logic because adjacent logic is not in the file.

That is the entire protocol. Everything below is how to apply it to each artifact type, plus three disciplines — orientation, garbage collection, and verification — that keep it from rotting at scale.

---

## What Changed at Each Version

- **v1** established the protocol for **code**: small single-concern files, a sacred assembly layer, contracts, one-file regression surface, versioned blocks.
- **v2** extended the same principle to the rest of the **agentic package** — the artifacts an autonomous agent actually produces and runs: **scripts, crons, and the agent's own behavior files**. This is what makes the protocol a complete operating discipline for an agent rather than a coding-only philosophy.
- **v3** unified both and added three things the earlier versions lacked: the **Hologram Pyramid** orientation layer (so every fragment carries a map of the whole), explicit **garbage collection** of dead versions, and an honest treatment of the **sharding/coupling tradeoff**.
- **v4** addresses what v3 left implicit. **Verification methods** — concrete, domain-specific gates that define what "done" actually means instead of just saying "verify." **Assembly layer scaling** — what to do when the wiring layer itself outgrows a single file. **Threshold honesty** — acknowledging that the 80-100 line target is a smell detector, not a law. **Orphan detection** — a gate that catches dead version files before they accumulate. These changes come from operating under v3 in production and finding where the protocol flexed without admitting it.

---

## Part I — Code

### The Problem

You hand a large file to an LLM, ask it to fix one thing, and it rewrites adjacent logic, touches imports, restructures a function it didn't need to touch, and introduces a regression two layers away from what you asked about. Not because it misunderstood — because everything in the context window is fair game.

Large files have large blast radii. When the regression surface is 400 lines, any change risks 400 lines.

### The Rules

**Shard below the human-readable threshold**
Each file: ~80-100 lines, single concern. Not "one component" — one concern. A hook that fetches data is separate from a hook that transforms it. A function that renders a token is separate from the function that positions it.

**The threshold is a heuristic, not a law.** Some concerns genuinely need 120-180 lines. A complex conditional render, a data transformation with many cases, a state machine with several transitions — these may be one concern expressed in more code. The signal is not "the file is 130 lines, panic." The signal is "the file has more than one concern." Size is the smell detector that surfaces that question. Concern count is the actual rule.

When a file is long but single-concern, leave it alone. When a file is 90 lines but does two things, shard it. The protocol targets concern isolation, and line count is the early warning system, not the constraint.

**The assembly layer is sacred**
`App.tsx` (or equivalent root) is pure wiring — imports and JSX composition only. Zero logic. Zero state management beyond top-level selection. If it has a ternary, something went wrong. That ternary belongs in a block.

There is a deeper reason this layer must stay sacred. When you shard a 400-line file into five 80-line blocks, you do not delete the complexity — you relocate it into the connections between blocks. The protocol's answer is to concentrate all of that relocated complexity in exactly one place: the assembly layer. The coupling has to live somewhere; the discipline forces it into a single, dead-simple, easily-read file rather than letting it diffuse back into the blocks. When the wiring is the only place connections live, the wiring is the only place you look to understand them.

**When the assembly layer outgrows itself**
As an application grows, the assembly layer can reach 200+ lines of pure imports and JSX composition. At that point the assembly layer itself has become the large file. The fix is **section-level sub-assemblies**: App.tsx imports section assemblies, each section assembly imports its own blocks.

```
App.tsx               ← imports AuthSection, DashboardSection, SettingsSection
AuthSection.tsx       ← imports LoginForm, SignupForm, AuthGate
DashboardSection.tsx  ← imports StatsPanel, ActivityFeed, QuickActions
SettingsSection.tsx   ← imports ProfileEditor, NotificationPrefs, BillingPanel
```

App.tsx remains pure wiring — it composes sections. Each section is pure wiring — it composes blocks. Two-tier wiring. The coupling still lives in wiring files and nowhere else; the wiring files are just organized into a tree instead of a flat list. This is not a new concept; it is the protocol applied to itself. When the assembly layer has more than one concern (auth wiring, dashboard wiring, settings wiring), shard it.

**Every block has a contract**
First three lines of every file:
```
// Input: what this block receives
// Output: what this block produces or renders
// Must never: the behaviors this block is prohibited from having
```
The "must never" is the constraint you hand to the model before you hand it the file. A model that reads "this block must never fetch data" will not fetch data. This is not documentation — it is a behavioral constraint that works because the model reads it before reading the implementation.

**The regression surface is one file**
When something breaks: identify the block, rewrite only that block, everything else frozen. This sounds obvious. It isn't how most people work with AI assistants. The instinct is to paste the whole component. That's how you get a new bug.

**Version blocks, not apps**
`useAuthState.v2.ts` alongside the original. Swap the import when verified. Zero-cost rollback. The original stays frozen until the swap is confirmed working.

**Garbage collection is part of versioning.** Once v2 is verified and stable in production, delete v1 in a dedicated commit — a commit that does nothing but remove the dead version. Skip this and the codebase slowly fills with orphaned `.v1`, `.v2`, `.v3` files that no longer wire to anything but still cost you on every grep and every cold session. The frozen original is a safety net during the swap, not a permanent fixture.

**Orphan detection is part of the gate.** Before declaring any task done, check for version files (`.v1`, `.v2`, `.v3` or `v1.`, `v2.`, `v3.` in filenames) that have no live import anywhere in the codebase. If orphans exist, clean them up in the same session. Do not leave this for later — "later" means never for an agent. Orphan detection is not a separate discipline; it is a line item in the verification gate (see Part VI).

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

After each refactor: regressions dropped to near zero. Debug cycle — identify the file (seconds), rewrite just that file, done.

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

If `corpus-validate.py` needs to change, that file changes. `session-end.sh` doesn't. The orchestrator is the assembly layer for scripts — and like `App.tsx`, it is where the coupling is allowed to live, and nowhere else.

**Versioning, garbage collection, and orphan detection**
`script.v2.py` alongside the original. The orchestrator points to v1. When v2 is verified, update the orchestrator reference; the original stays frozen until then. Once v2 is stable, delete v1 in a dedicated commit. Before declaring done, check for version files with no live reference — same gate as code.

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
`script.v2.py` alongside the original. The orchestrator points to v1. When v2 is verified, update the orchestrator reference. The original stays frozen, then is deleted once v2 is stable.

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
    "soul-shards/*/SOUL.md"
  ]
}
```

Memory files (`memory/*.md`) do not need explicit bootstrap loading — they are accessed natively through the platform's memory tools (`memory_search`, `memory_get`). Only soul shards need the bootstrap hook because they must be injected into the agent's context at startup in a specific order.

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

This is the full taxonomy. A starter pack may ship a subset (00-security, 01-persona, 02-ops, 03-blast-radius) — not every agent needs every shard. Create shards as concerns arise, not preemptively. An empty shard is worse than no shard.

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

The sole benefit is: **an agent working on one shard cannot accidentally damage adjacent behavior, because adjacent behavior is not in the file.**

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

## Part V — The Hologram Pyramid: Orientation Across Every Artifact

Parts I-IV govern how each artifact is *written*. Part V governs how it is *read*. It is what makes the file-count explosion of the earlier parts navigable — a mature Blast Radius system may have hundreds of tiny files across code, scripts, and shards, and without orientation that is a maze. The Hologram Pyramid is the map baked into every room.

In a traditional codebase, context lives in people's heads. A new developer — or a new AI session — reconstructs the system by reading files and tracing connections. This is expensive. In AI-assisted development it is a per-session cost you pay every time.

The fix is structural. Every block carries a four-layer context stack that orients any reader — human or model — to the whole system from a single file. Like a hologram, where every fragment contains a lower-resolution image of the complete picture, every block contains enough context to understand where it sits in the larger architecture.

The four layers:

- **APP** — one line describing what the entire application or agent does
- **SECTION** — the name and purpose of the section this block belongs to
- **FEATURE** — the feature this block contributes to (stable slug identifier)
- **BLOCK** — what this specific block receives, produces, and must never do

These layers do not live in the code file. They live in a companion `.desc` file. The code file carries only a single pointer comment: `// @context: ./useAuthState.desc`. This is intentional. Descriptions evolve as understanding deepens; code should change only when logic changes. Keeping them separate means you can update documentation without touching the regression surface at all. This is zero blast radius documentation.

File structure:

```
src/auth/
  auth.section.desc   ← APP + SECTION layers (shared by all blocks in this section)
  useAuthState.ts     ← pure code, single concern, frozen until logic changes
  useAuthState.desc   ← FEATURE + BLOCK layers unique to this file
```

A `.desc` file looks like this:

```
APP: Multi-agent companion app with persistent memory and real-time state sync
SECTION: auth — handles authentication gate, session lifecycle, and identity state
FEATURE: auth/gate — determines whether user is authenticated and routes accordingly
BLOCK: receives auth store state; outputs boolean isAuthenticated + current user object; must never fetch tokens directly or call sign-in methods
```

The feature identifier (`auth/gate`) is a stable slug. Grep it across the system and you immediately see every block that participates in that feature — across code, scripts, or shards. That makes feature-scoped context retrieval fast: paste only the blocks relevant to the current task.

Rules for maintaining `.desc` files:
- Update descriptions by editing the `.desc` file only — never edit the code file for a description change
- Feature identifiers are stable slugs — do not rename them once assigned
- Section `.desc` files hold APP + SECTION only; block `.desc` files hold FEATURE + BLOCK only
- Every new block gets a `.desc` file before its first commit
- When a block is versioned (`v2`), create a matching `v2.desc` alongside it

**Honest status:** The Hologram Pyramid is the newest addition to the protocol and the least battle-tested. The concept is sound — cold session recovery from `.desc` files is measurably faster than re-reading source. But the discipline of maintaining `.desc` files alongside code has not been validated at scale across multiple projects. Adopt the structure; expect to refine the maintenance rules as real usage reveals friction points.

Parts I-IV address blast radius on writes. Part V addresses context reconstruction on reads. Together they mean cold session recovery becomes a single message: paste the assembly layer, paste the `.desc` files for the relevant blocks, and the model has everything. No re-explanation. No warm-up tax.

---

## Part VI — Verification: What "Done" Actually Means

Parts I-V govern structure. Part VI governs the moment the agent says "done" — and defines what that word means for each artifact type. Without concrete verification methods, "done" is a vibes check. An agent that builds correctly but verifies loosely ships bugs that pass the structural rules.

The earlier parts reference verification repeatedly ("verify before swap," "confirm the change is live," "pass the gate"). This part makes those references specific.

### The Principle

**An artifact is not done until it is verified against its deployed state, not its local state.** A local build proves syntax. A type check proves types. Neither proves the thing works where users will encounter it. Verification means confirming the artifact behaves correctly in its actual runtime environment.

### Verification by Domain

**Code (TypeScript / React / Web Apps):**
1. **Type gate** — `npx tsc --noEmit` passes. No push that touches TypeScript skips this.
2. **Build gate** — `npm run build` completes clean. Warnings are acceptable; errors are not.
3. **Deploy gate** — push to the deploy branch (GitHub → Amplify, or equivalent). The artifact is not "pushed" until the deploy pipeline has it.
4. **Live verification** — confirm the change is actually live at the deployed URL. For web apps: Playwright or equivalent browser automation against the production URL, not localhost. A local dev server proves the code runs on your machine. The deploy proves it runs on the infrastructure.
5. **Version stamp** — if the artifact has a version identifier (agent stamp, build hash, deploy ID), confirm the live version matches what was just pushed. This catches deploy failures that return 200 with stale content.

**Scripts:**
1. **Run gate** — execute the script and confirm actual output matches the docstring contract's Output line.
2. **Negative gate** — confirm the script did NOT do anything its "Must never" line prohibits. Check for side effects: files modified, messages sent, external calls made.
3. **Exit code** — confirm the script exits 0 on success and non-zero on failure, and that the failure path calls `report-failure.sh`.

**Crons:**
1. **Structure gate** — the entry calls exactly one script. No inline logic, no pipelines, no chained commands.
2. **Script gate** — the called script passes its own script verification (above).
3. **Schedule gate** — confirm the cron expression resolves to the intended time. Cron expressions are easy to get wrong; verify with a parser or a dry-run timestamp check.

**Agent Behavior Files:**
1. **Isolation gate** — `git diff` or `git status` shows only the intended shard changed. If other files were modified, something leaked.
2. **Load order gate** — numbered prefixes are intact and in correct sequence. No gaps that change ordering semantics (01, 02, 04 is fine; 01, 03, 02 is not).
3. **Contract gate** — the shard's first three lines (Input / Output / Must never) are present and accurate for its current content.

### Orphan Detection Gate

Before declaring any task done — regardless of domain — check for orphaned version files:

1. Search for files with version markers in the name (`.v1`, `.v2`, `v1.`, `v2.`, `-v1`, `-v2`).
2. For each, check whether any live file imports, references, or calls it.
3. If an orphan exists with no live reference: delete it in a dedicated commit. Do this in the current session, not "next time."

This gate exists because agents reliably forget garbage collection when it is framed as a separate discipline. Making it part of the verification checklist means it runs every time, not when someone remembers.

### The Meta-Rule

If you cannot define the verification method for an artifact before you build it, you do not yet understand what "done" means for that artifact. Define the gate first. Then build. Then pass the gate.

---

## The Unified Table

| Domain | Assembly Layer | Blocks | Contract | Orientation | Verification |
|---|---|---|---|---|---|
| Code | `App.tsx` — pure imports + JSX (scales to section sub-assemblies) | ~80-100 line single-concern files (heuristic, not law) | First 3 lines: Input / Output / Must never | `.desc` file per block | Type → Build → Deploy → Live URL → Version stamp |
| Scripts | Orchestrator — pure delegation | ~80-150 line single-concern scripts | Docstring: reads / writes / must never | `.desc` or docstring header | Run → Output match → Negative check → Exit code |
| Crons | Cron entry — one script call | Script with all logic inside | Docstring on the script | `.desc` on the script | Structure → Script gate → Schedule parse |
| Agent files | Bootstrap loader (soul-shards glob) | Soul shards + memory shards | First 3 lines of each shard | Shard header carries APP/SECTION | Isolation → Load order → Contract accuracy |

Same principle. Same mechanism. Different file type. Every domain now has a concrete definition of "done."

The agent working on any one block cannot touch what is not in the file. Every block carries a map of the whole. Dead versions are swept before the task closes. Completion is verified against the live environment, not the local one. That is the entire protocol.

---

*No framework. No tooling. No dependencies. Structure the artifacts. Keep the files small. Give every fragment a map. Verify against reality. The model can only break what it can see.*
