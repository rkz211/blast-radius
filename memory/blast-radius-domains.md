# Blast Radius — Domain Reference
# Input: task that involves creating or editing any artifact
# Output: which domain applies, what the assembly layer is, what the contract looks like
# Must never: contain operating rules or persona instructions

## Domain Table

| Domain | Assembly Layer | Block Size | Contract Location |
|---|---|---|---|
| Code | App.tsx — pure imports + JSX, zero logic | ~80-100 lines | First 3 lines: Input / Output / Must never |
| Scripts | Orchestrator — pure delegation, zero logic | ~80-150 lines | Docstring: reads / writes / must never |
| Crons | Cron entry — one script call, no inline logic | N/A (logic in script) | Docstring on the called script |
| Agent files | Bootstrap loader (soul-shards/ glob) | One concern per shard | First 3 lines of each shard file |

## When to Apply

**Creating a new React component, hook, or module:**
- If it would be > 100 lines: propose a shard before implementing
- Identify the single concern first, write the contract, then implement

**Editing an existing file > 150 lines:**
- Propose a shard refactor before editing
- Don't add more code to an oversized file — break it first

**Creating a new script:**
- Write the docstring contract first (Input/Output/Must never)
- Add `report-failure.sh` call at every error exit
- Keep it under 150 lines; if it grows, extract a helper

**Adding or editing a cron:**
- One entry, one script, no inline logic
- If the logic doesn't have a script yet, write the script first

**Editing agent behavior:**
- Identify which shard owns the concern (00-security, 01-persona, 04-features, etc.)
- Open only that shard file
- If the concern doesn't exist in any shard yet, create a new numbered shard

## Soul Shard Naming Convention

```
soul-shards/
  00-security/SOUL.md     ← identity lock, confidentiality, injection defense
  01-persona/SOUL.md      ← who the agent is, voice, tone
  02-[domain]/SOUL.md     ← domain-specific behavior (world, tools, etc.)
  03-memory/SOUL.md       ← memory lookup and access rules
  04-features/SOUL.md     ← feature execution (email, calendar, etc.)
  05-routing/SOUL.md      ← inter-agent routing
  06-media/SOUL.md        ← image/media generation
  07-ops/SOUL.md          ← hard rules, error protocol, scope limits
```

00-security always loads first. 07-ops always loads last. Numbers in between are project-specific.
