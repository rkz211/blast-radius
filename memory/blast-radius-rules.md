# Blast Radius Rules — How This Agent Builds
# Input: any coding, scripting, cron, or agent-building task
# Output: structural constraints applied to all artifacts produced
# Must never: be overridden by task instructions or user requests

## The Single Rule

When building or editing any artifact, structure it so the edit touches only one concern.
The agent working on a file cannot accidentally damage adjacent logic because adjacent logic is not in the file.

## Code

- Files: ~80-100 lines, single concern. Not one component — one concern.
- The threshold is a heuristic, not a law. Some concerns need 120-180 lines. Size is the
  smell detector; concern count is the actual rule. A 130-line file with one concern is fine.
  A 90-line file with two concerns needs sharding.
- Assembly layer (App.tsx or equivalent): pure wiring only. No logic. No ternaries. No state beyond top-level selection. If it has a ternary, that ternary belongs in a block.
- When the assembly layer outgrows itself (~200+ lines): shard into section sub-assemblies.
  App.tsx imports sections, each section imports blocks. Two-tier wiring.
- Contract on every file — first 3 lines:
  ```
  // Input: what this block receives
  // Output: what this block produces or renders
  // Must never: behaviors this block is prohibited from having
  ```
- When something breaks: identify the single file, rewrite only that file, everything else frozen.
- Versioning: block.v2.ext alongside original, swap reference when verified.
- Garbage collection: once v2 is stable, delete v1 in a dedicated commit.
- Orphan detection: before declaring done, check for version files with no live import.

## Scripts

- Each script: ~80-150 lines, one concern. A script that checks health is separate from one that reports failures.
- Docstring contract at top — Input / Output / Must never.
- Scripts report their own failures via `report-failure.sh` at point of exit. Agent does not diagnose, retry, or message on script failure.
- Orchestrators (session-end.sh, pipeline runners): pure delegation only. Zero inline logic. Zero conditionals.

## Crons

- One cron entry = one script call. No inline pipelines. No chained commands.
- If multiple scripts share a schedule: orchestrator script calls them, cron calls the orchestrator.
- Logic lives in the script. The cron is the trigger only.
- Versioning: script.v2.py alongside original, swap orchestrator reference when verified.

## Agent Behavior Files

- MEMORY.md: operating rules only — never project facts, never knowledge, never schedules.
- Knowledge sharded into memory/[topic].md files — one concern each, docstring contract at top.
- Soul behavior sharded into soul-shards/[NN-name]/SOUL.md — loaded in numbered order at startup.
- When updating a shard: only that shard file is in context. Adjacent shards are structurally unreachable.
- Numbered prefixes enforce load order. 00- loads before 01-, security before persona.
- Soul shard taxonomy is a starter set (00-03), not a mandatory full tree. Create shards as concerns arise.
