# The Blast Radius Protocol — v4
**Author:** Roark Pinkerton
**v1:** May 2026 — Code
**v2:** May 2026 — Code, Scripts, Crons, Agent Files
**v3:** June 2026 — Unified: Hologram Pyramid, garbage collection, sharding/coupling tradeoff
**v4:** June 2026 — Verification methods, assembly scaling, threshold honesty, orphan detection
**v4.1:** June 2026 — WIRING layer in .desc files, tool-use tradeoff acknowledgment, .desc maintenance gate
**v4.2:** June 2026 — "When not to shard" exceptions, risk-tier versioning
**v4.3:** June 2026 — Security shard scoping, ternary clarification, tool-use cost scenario, five-layer reference table, grep payoff examples, Hologram Pyramid + WIRING field status updates, optional verification tooling

---

## The Single Idea

In our experience: a Sonnet or Opus agent with a good critical thinking loop can safely build production software — full-stack web applications, cloud infrastructure, backend engines, and autonomous agent systems — if every artifact is structured so that any given edit touches only one concern. This is the result we have observed consistently across our own systems. It has not yet been independently validated beyond our work, and the case studies in this repository are from private projects by the same authors. We present this as documented field experience, not as a proven universal claim.

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
- **v4.1** adds the **WIRING layer** to `.desc` files — dependency edges (who calls this block, what it calls) tracked independently from code, so adding a new caller never touches the called file. Acknowledges the **tool-use tradeoff** of heavy sharding (more files = more tool calls = more tokens) and positions the Hologram Pyramid as the mitigation. Adds a **`.desc` maintenance gate** to the verification checklist so orientation files don't drift from reality.
- **v4.2** adds two refinements from external review. **"When not to shard"** — explicit exceptions where sharding is counterproductive (parsers, generated files, state machines, dense algorithms). **Risk-tier versioning** — distinguishes routine edits (commit and move on) from risky refactors (version alongside, verify, swap, clean up). Not every change needs a `.v2` file; Git handles rollback for normal work.
- **v4.3** addresses seven items surfaced by external reviewers (GPT 5.5, Gemini, Grok) and internal production experience. **Security shard scoping** — the 00-security shard now distinguishes external-facing agents from development agents. **Ternary clarification** — "zero ternaries" was memorable but misleading; the concern is conditional logic in wiring, not the syntax itself. **Tool-use cost scenario** — replaces the abstract tradeoff acknowledgment with a concrete side-by-side comparison. **Five-layer reference table** — a structured quick-reference for the Hologram Pyramid layers. **Grep payoff examples** — concrete one-liners that show the feature slug and WIRING grep value immediately. **Field status updates** — the Hologram Pyramid and WIRING honest-status sections now reflect actual production usage. **Optional verification tooling** — a new Part VII introducing a lightweight, optional check script.

---

## Part I — Code

### The Problem

You hand a large file to an LLM, ask it to fix one thing, and it rewrites adjacent logic, touches imports, restructures a function it didn't need to touch, and introduces a regression two layers away from what you asked about. Not because it misunderstood — because everything in the context window is fair game.

Large files have large blast radii. When the regression surface is 400 lines, any change risks 400 lines.

### Where the Wall Is

In production use, the breakdown point is around **~700 lines**. Below that, agents cope — they make mistakes, but the mistakes are containable and the fixes land. Above ~700 lines, the tailspin starts: fixes create regressions, regressions create fixes, and the agent that built the system begins dismantling it.

The protocol eliminates this wall entirely. After sharding a codebase to single-concern files, we have not hit a new complexity ceiling. As systems grow, you add more files and more layers of assembly (section sub-assemblies, orchestrator hierarchies), and the agent's effective blast radius stays constant — one file, one concern, regardless of total system size.

### Why the Refactor Itself Is Easy

A common concern: "If the agent can't handle large files, how does it handle the refactor?" The answer is that reading big and writing small is the easy direction. The agent reads the full monolith — 500, 700, 800 lines — and has complete context of every concern. Then it writes each module individually: 80-100 lines, one concern, with a contract. The input is large; the output is small. That's the direction LLMs are good at.

The hard direction is the one that created the problem: writing *into* a large file. Adding a fix to a 700-line file means the agent must hold all 700 lines in its regression surface while producing output that doesn't damage any of them. That's where it breaks down.

In practice, all four case studies in this repository were refactored by an AI agent in a single session each, with zero errors. The refactor is not the hard part. Maintaining the monolith is the hard part. The protocol replaces the hard part with structure.

### The Rules

**Shard below the human-readable threshold**
Each file: ~80-100 lines, single concern. Not "one component" — one concern. A hook that fetches data is separate from a hook that transforms it. A function that renders a token is separate from the function that positions it.

**The threshold is a heuristic, not a law.** Some concerns genuinely need 120-180 lines. A complex conditional render, a data transformation with many cases, a state machine with several transitions — these may be one concern expressed in more code. The signal is not "the file is 130 lines, panic." The signal is "the file has more than one concern." Size is the smell detector that surfaces that question. Concern count is the actual rule.

When a file is long but single-concern, leave it alone. When a file is 90 lines but does two things, shard it. The protocol targets concern isolation, and line count is the early warning system, not the constraint.

**When not to shard.** Some artifacts are single-concern but inherently long, and splitting them would create coupling problems worse than the large file:

- **Parsers and grammars** — a parser's rules need to be visible together. Splitting a 300-line grammar across files forces the agent to hold multiple files in context to understand one concern.
- **Generated files** — code generated by tooling (schema outputs, API clients, type definitions from a spec) should not be manually split. The generator owns the structure.
- **State machines** — a state machine's transitions are one concern even if they span 200 lines. The transitions only make sense as a complete set.
- **Dense algorithm modules** — a sorting algorithm, a physics solver, a compression routine. These are single-concern by nature. Sharding a tight algorithm across files makes it harder to reason about, not easier.
- **Configuration and schema files** — a JSON schema, a database migration, a Terraform resource block. These are declarations, not logic, and their structure is dictated by the tool that reads them.

The rule is not "break up every long file." The rule is "every file should have one concern." When a file is long because its concern is inherently complex, the correct response is to leave it alone and put a contract at the top.

**The assembly layer is sacred**
`App.tsx` (or equivalent root) is pure wiring — imports and JSX composition only. Zero logic. Zero state management beyond top-level selection. If a ternary appears in the assembly layer, treat it as a smell signal: conditional logic is creeping into the wiring. The concern that ternary represents — a display toggle, a feature gate, a layout switch — belongs in a block, not in the assembly layer.

The word "sacred" does not mean "ternary syntax is banned." A ternary that selects between two already-composed sections based on a single boolean is borderline acceptable — but it is still the assembly layer taking on a decision that a dedicated block could own. The protocol's position is that the wiring layer is safest when it is purely declarative: imports, composition, and nothing that requires understanding *why* a branch is taken. When in doubt, extract the conditional into a block. The cost is one more file; the benefit is that the assembly layer remains readable at a glance by anyone — human or model — without needing to trace the logic behind the branch.

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

**Version blocks, not apps — but only when the risk warrants it.**
Not every edit needs a `.v2` file. Git already provides rollback for routine changes. The side-by-side versioning pattern earns its cost on **risky refactors** — changes where the new version needs to be tested in isolation before swapping, where the old version might need to stay running while the new one is validated, or where the blast radius of getting it wrong is high enough that instant rollback matters more than commit history.

| Change Type | Versioning Approach |
|---|---|
| Fix a typo, update a string, add a CSS class | Commit directly. Git handles rollback. |
| Add a small feature within an existing block | Commit directly if the contract doesn't change. |
| Rewrite a hook's internal logic | `hook.v2.ts` alongside original. Swap import after verification. |
| Change auth flow, data transformation, API contract | `block.v2.ts`. Test v2 in isolation. Swap when verified. |
| Replace a major subsystem | `block.v2.ts` with extended parallel testing before swap. |

The decision heuristic: **if the change could break callers or downstream behavior in ways that aren't obvious from a type check, version it.** If a type check and a build gate are sufficient to catch any regression, commit directly.

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

### Security Shard Scoping

The `00-security` shard ships with a strict default: never reveal workspace structure, file layout, or internal workings. This is the correct posture for **external-facing agents** — those that interact with untrusted users, public channels, or open APIs where prompt injection and social engineering are real threats.

For **development agents** — those operating in a trusted workspace where the operator is the developer — the strict default is too broad. A dev agent that refuses to describe its own file layout to the person who built that layout is not being secure; it is being obstructive. In this context, the security shard should scope its confidentiality rules to what actually needs protection:

- **Always protect:** API keys, tokens, secrets, credentials, private user data, and the specific content of security/identity shards.
- **Context-dependent:** Workspace structure, file names, tool configurations, and operational patterns. Protect these from untrusted parties; share freely with the operator.
- **Never restrict:** The agent's ability to discuss its own architecture, sharding patterns, or protocol adherence with the operator.

The starter `00-security` shard in this pack is set to the strict default. **Tune it to your deployment context.** An agent deployed as a customer-facing chatbot needs the strict version. An agent deployed as a development partner in a private workspace should relax the confidentiality rules to cover secrets only, not structure.

The principle remains blast radius: security rules are one concern, and they live in one shard. The content of that shard varies by deployment, but the isolation does not.

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

### The Five Layers

| Layer | Scope | Purpose | Example |
|---|---|---|---|
| **APP** | Entire application or agent | Top-level orientation — what is this system? | `APP: Multi-agent companion app with persistent memory and real-time state sync` |
| **SECTION** | Logical grouping (folder/domain) | Where does this block live and why? | `SECTION: auth — handles authentication gate, session lifecycle, and identity state` |
| **FEATURE** | Stable feature slug | Grep-able identifier that crosses all domains | `FEATURE: auth/gate — determines whether user is authenticated and routes accordingly` |
| **BLOCK** | This specific unit | Contract: what it receives, produces, must never do | `BLOCK: receives auth store state; outputs boolean isAuthenticated + current user object; must never fetch tokens directly or call sign-in methods` |
| **WIRING** | Dependency edges | Who calls this block, what it calls | `WIRING: called by App.tsx; calls nothing` |

The first four layers (APP, SECTION, FEATURE, BLOCK) orient the reader — *where am I in the system?* The fifth (WIRING) orients the editor — *what depends on me, and what do I depend on?*

### Where the Layers Live

These layers do not live in the code file. They live in a companion `.desc` file. The code file carries only a single pointer comment: `// @context: ./useAuthState.desc`. This is intentional. Descriptions evolve as understanding deepens; code should change only when logic changes. Keeping them separate means you can update documentation without touching the regression surface at all. This is zero blast radius documentation.

File structure:

```
src/auth/
  auth.section.desc   ← APP + SECTION layers (shared by all blocks in this section)
  useAuthState.ts     ← pure code, single concern, frozen until logic changes
  useAuthState.desc   ← FEATURE + BLOCK + WIRING layers unique to this file
```

A `.desc` file looks like this:

```
APP: Multi-agent companion app with persistent memory and real-time state sync
SECTION: auth — handles authentication gate, session lifecycle, and identity state
FEATURE: auth/gate — determines whether user is authenticated and routes accordingly
BLOCK: receives auth store state; outputs boolean isAuthenticated + current user object; must never fetch tokens directly or call sign-in methods
WIRING: called by App.tsx; calls nothing
```

### The WIRING Layer

The WIRING layer tracks dependency edges: who calls this block and what it calls. This information changes when callers are added or removed — which is a different lifecycle than when the block's logic changes. Keeping WIRING in the `.desc` file means adding a new caller to a script never touches the script file. It also means you can reconstruct the full dependency graph of the system from `.desc` files alone, with zero tooling.

For scripts and crons, WIRING is especially valuable:

```
APP: Agent QC suite — operational quality checks
SECTION: qc — quality checks and cost audits
FEATURE: qc/cost — per-user token cost tracking
BLOCK: reads session logs; outputs cost breakdown to stdout; must never modify files or send messages
WIRING: called by session-end.sh, cron agent-nightly-qc; calls nothing
```

When an orchestrator is retired or a cron schedule changes, you update the `.desc` files of the affected blocks. The blocks themselves stay frozen.

WIRING format: `called by <comma-separated callers>; calls <comma-separated callees>` — use `calls nothing` or `called by nothing` when one side is empty.

### Feature Slugs and the Grep Payoff

The feature identifier (`auth/gate`) is a stable slug. Grep it across the system and you immediately see every block that participates in that feature — across code, scripts, or shards. That makes feature-scoped context retrieval fast: paste only the blocks relevant to the current task.

This is where the sharding tax pays for itself. Three concrete examples:

```bash
# Find every block that participates in the auth gate feature
grep -r "auth/gate" --include="*.desc"

# Reconstruct the full dependency graph of the system
grep -r "^WIRING:" --include="*.desc"

# Find everything that calls a specific block
grep -r "called by.*session-end" --include="*.desc"
```

The first grep replaces "read every file in src/auth/ and trace the imports." The second replaces building a dependency graph tool. The third answers "what breaks if I change session-end.sh?" in one command. Feature slugs are stable — once assigned, do not rename them. The grep payoff depends on consistency.

### Rules for Maintaining `.desc` Files

- Update descriptions by editing the `.desc` file only — never edit the code file for a description change
- Feature identifiers are stable slugs — do not rename them once assigned
- Section `.desc` files hold APP + SECTION only; block `.desc` files hold FEATURE + BLOCK + WIRING only
- Every new block gets a `.desc` file before its first commit
- When a block is versioned (`v2`), create a matching `v2.desc` alongside it
- When a caller is added or removed, update the WIRING line in the callee's `.desc` file

### The Tool-Use Tradeoff

Heavy sharding produces many small files. An agent working in a heavily-sharded system makes more tool calls (`read_file`, `grep`, `list_directory`) to understand the landscape than one working in a few large files. This is a real cost in tokens and latency. The Hologram Pyramid is the mitigation.

Consider a concrete scenario. An auth module exists as either one 400-line file or five 80-line blocks with `.desc` files:

**Monolith approach (1 file, no `.desc`):**
- Cold start: 1 `read_file` call, 400 lines of context consumed. The agent has everything but must parse the entire file to find the relevant concern.
- Editing: the agent rewrites within a 400-line context window. Any line is fair game. The blast radius is 400 lines.

**Sharded approach (5 blocks + `.desc` files):**
- Cold start: 1 `read_file` on the section `.desc` (5 lines — instant orientation), then 1 `read_file` on the specific block's `.desc` (4 lines — exact contract and wiring), then 1 `read_file` on the block itself (80 lines). Total: 3 reads, ~90 lines of context consumed. The agent knows exactly where it is and what it can touch.
- Editing: the agent rewrites within an 80-line context window. Adjacent logic is structurally unreachable. The blast radius is 80 lines.

The sharded approach uses more tool calls but consumes less total context and produces a smaller blast radius. The `.desc` files make the extra reads cheap — a few lines each that compress the entire system map. The tradeoff is worth it. It is not free, but contained blast radius and fast orientation outweigh the extra calls.

### Field Status

The Hologram Pyramid has been in production use since v4.1. What works:

- **Cold-session recovery is measurably faster.** Pasting the assembly layer and relevant `.desc` files gives a new session full orientation in one message. No re-explanation, no "let me read through the codebase" warm-up.
- **Feature-scoped grep is reliable.** Stable slugs make cross-domain feature discovery a one-liner.
- **WIRING lines accurately track caller/callee relationships** when the `.desc` maintenance gate (Part VI) is enforced. Without the gate, agents skip `.desc` updates reliably — the gate is mandatory, not optional.

What to expect when adopting:

- **Initial onboarding of legacy codebases** is the biggest friction point. Generating `.desc` files for an existing system requires understanding the system first. Start with the section `.desc` files (APP + SECTION), then add block `.desc` files as you touch each block. Do not try to `.desc` the entire codebase at once.
- **Dynamic imports and conditional wiring** are edge cases for the WIRING layer. If a block is conditionally imported (e.g., lazy-loaded routes), note the condition in the WIRING line: `called by App.tsx (lazy); calls nothing`. This is a convention, not enforcement — refine as your system's patterns stabilize.
- **Maintenance discipline is real.** `.desc` files drift from reality when agents are not held to the Orientation Gate in Part VI. The gate exists specifically because this will happen.

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

### Orientation Gate (`.desc` files)

If the project uses the Hologram Pyramid:
1. Every block that was created or modified has a corresponding `.desc` file.
2. The BLOCK line in each `.desc` accurately reflects the current contract (Input/Output/Must never).
3. The WIRING line reflects the current callers and callees — if a caller was added or removed during this task, the callee's `.desc` is updated.
4. If a new section was created, a `section.desc` file exists with APP + SECTION layers.

Agents will skip `.desc` maintenance unless it is part of the gate. This is that gate.

### Orphan Detection Gate

Before declaring any task done — regardless of domain — check for orphaned version files:

1. Search for files with version markers in the name (`.v1`, `.v2`, `.v3` or `v1.`, `v2.`, `v3.` in filenames).
2. For each, check whether any live file imports, references, or calls it.
3. If an orphan exists with no live reference: delete it in a dedicated commit. Do this in the current session, not "next time."

This gate exists because agents reliably forget garbage collection when it is framed as a separate discipline. Making it part of the verification checklist means it runs every time, not when someone remembers.

### The Meta-Rule

If you cannot define the verification method for an artifact before you build it, you do not yet understand what "done" means for that artifact. Define the gate first. Then build. Then pass the gate.

---

## Part VII — Optional Verification Tooling

Parts I-VI are the protocol. They require no tooling, no framework, and no dependencies — just structure and discipline. This part describes an **optional** check script that automates the mechanical parts of the verification gate. It is not part of the protocol. It is a convenience for teams that want automated enforcement alongside the discipline.

### Why This Is Optional

The protocol's tagline is "no framework, no tooling, no dependencies." That is intentional. The value is in the structure, not in a CLI tool. A team that adopts the structural rules without the script gets the full benefit. A team that runs the script without understanding the structural rules gets a linter, not a protocol.

That said, some verification steps are mechanical and tedious — checking for contract presence, scanning for orphaned version files, confirming the assembly layer has no logic. Automating these reduces the chance that an agent or developer skips a gate out of fatigue, not intention.

### What a Check Script Would Cover

A lightweight `blast-check.sh` (or equivalent) that runs against a directory or a set of changed files:

**Contract presence:**
- Every `.ts`, `.tsx`, `.js`, `.jsx` file has `// Input:`, `// Output:`, and `// Must never:` in its first five lines.
- Every `.py` file has a docstring with `Input:`, `Output:`, and `Must never:`.
- Every `SOUL.md` or `MEMORY.md` has `# Input:`, `# Output:`, and `# Must never:` in its header.
- Report: list of files missing contracts.

**Orphan detection:**
- Find files with version markers (`.v1`, `.v2`, `-v1`, `-v2`, `v1.`, `v2.`) in their names.
- For each, check whether any other file in the project imports, requires, or references it.
- Report: list of orphaned version files with no live reference.

**Assembly layer purity:**
- Identify assembly files (`App.tsx`, files matching `*Section.tsx`, orchestrator scripts).
- Check for logic indicators: ternary operators (`?`), `if` statements, `switch`, `for`/`while` loops, state declarations beyond top-level.
- Report: list of assembly files with suspected logic. (This is a heuristic — flags for review, not hard failures.)

**`.desc` accuracy (if Hologram Pyramid is adopted):**
- Every `.ts`/`.tsx`/`.py` file that has a `@context:` pointer has a corresponding `.desc` file that exists.
- Every `.desc` file has all required layers (FEATURE, BLOCK, WIRING for block-level; APP, SECTION for section-level).
- Report: missing `.desc` files, `.desc` files with missing layers.

### What It Would NOT Cover

- Whether contracts are *accurate* — only a human or model can judge that. The script checks presence, not correctness.
- Whether the assembly layer's composition is *right* — only structural understanding determines that.
- Whether `.desc` WIRING lines are *current* — the script can check for presence but not for accuracy against the actual call graph.
- Build, type-check, or deployment — those are existing tool gates (`tsc`, `npm run build`, CI pipelines). The check script is not a build system.

### Implementation Guidance

If you build this:
- Keep it under 150 lines. It is a single-concern script — apply the protocol to the protocol's own tooling.
- Do not make it a required CI gate on day one. Run it manually or in advisory mode. Let teams build trust in the checks before making them blocking.
- Do not package it as an npm module or CLI framework. A shell script in a `tools/` directory is the right weight.
- Consider a GitHub Actions workflow that runs the checks on PRs and posts results as a comment — advisory, not blocking.

### What This Is Not

This is not "Blast Radius: The Framework." The protocol is structural discipline. The script catches mechanical oversights. If you find yourself extending the script to 500 lines with plugins and configuration files, you have built a linter and lost the plot. Delete it and go back to the rules.

---

## The Unified Table

| Domain | Assembly Layer | Blocks | Contract | Orientation | Verification |
|---|---|---|---|---|---|
| Code | `App.tsx` — pure imports + JSX (scales to section sub-assemblies) | ~80-100 line single-concern files (heuristic, not law) | First 3 lines: Input / Output / Must never | `.desc` file: APP/SECTION/FEATURE/BLOCK/WIRING | Type → Build → Deploy → Live URL → Version stamp |
| Scripts | Orchestrator — pure delegation | ~80-150 line single-concern scripts | Docstring: reads / writes / must never | `.desc` file: BLOCK/WIRING (who calls, what called) | Run → Output match → Negative check → Exit code |
| Crons | Cron entry — one script call | Script with all logic inside | Docstring on the script | `.desc` on the script: WIRING to cron entry | Structure → Script gate → Schedule parse |
| Agent files | Bootstrap loader (soul-shards glob) | Soul shards + memory shards | First 3 lines of each shard | Shard header carries APP/SECTION | Isolation → Load order → Contract accuracy |

Same principle. Same mechanism. Different file type. Every domain now has a concrete definition of "done."

The agent working on any one block cannot touch what is not in the file. Every block carries a map of the whole — including who calls it and what it calls. Dead versions are swept before the task closes. Completion is verified against the live environment, not the local one. That is the entire protocol.

---

*No framework. No tooling. No dependencies. Structure the artifacts. Keep the files small. Give every fragment a map. Verify against reality. The model can only break what it can see.*
