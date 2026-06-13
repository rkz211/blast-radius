# Blast Radius — Domain Reference
# Input: task that involves creating or editing any artifact
# Output: which domain applies, what the assembly layer is, what the contract looks like
# Must never: contain operating rules or persona instructions

## Domain Table

| Domain | Assembly Layer | Block Size | Contract Location | Verification |
|---|---|---|---|---|
| Code | App.tsx — pure imports + JSX, scales to section sub-assemblies | ~80-100 lines (heuristic) | First 3 lines: Input / Output / Must never | Type → Build → Deploy → Live URL → Version stamp |
| Scripts | Orchestrator — pure delegation, zero logic | ~80-150 lines | Docstring: reads / writes / must never | Run → Output match → Negative check → Exit code |
| Crons | Cron entry — one script call, no inline logic | N/A (logic in script) | Docstring on the called script | Structure → Script gate → Schedule parse |
| Agent files | Bootstrap loader (soul-shards/ glob) | One concern per shard | First 3 lines of each shard file | Isolation → Load order → Contract accuracy |

## When to Apply

**Creating a new React component, hook, or module:**
- If it would be > 100 lines: check if it has more than one concern — shard if so
- If it's one concern that needs 150 lines: leave it. Size is the smell detector, not the rule.
- Identify the single concern first, write the contract, then implement

**Editing an existing file > 150 lines:**
- Check concern count. Multiple concerns → propose a shard before editing
- Single concern → edit in place, the file is fine

**Creating a new script:**
- Write the docstring contract first (Input/Output/Must never)
- Add `report-failure.sh` call at every error exit
- Keep it under 150 lines; if it grows, extract a helper

**Adding or editing a cron:**
- One entry, one script, no inline logic
- If the logic doesn't have a script yet, write the script first

**Editing agent behavior:**
- Identify which shard owns the concern (00-security, 01-persona, 02-ops, etc.)
- Open only that shard file
- If the concern doesn't exist in any shard yet, create a new numbered shard

## Soul Shard Naming Convention

```
soul-shards/
  00-security/SOUL.md     ← identity lock, confidentiality, injection defense
  01-persona/SOUL.md      ← who the agent is, voice, tone
  02-ops/SOUL.md          ← hard rules, error protocol, scope limits
  03-blast-radius/SOUL.md ← enforcement posture — staying on protocol under pressure
```

This is a starter set. The full taxonomy (02-world through 07-ops) exists in the whitepaper.
Create shards as concerns arise, not preemptively. An empty shard is worse than no shard.

00-security always loads first. Numbers in between are project-specific.
