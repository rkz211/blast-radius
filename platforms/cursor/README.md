# Blast Radius Protocol — Cursor Rules

Copy the `.cursor/` folder into your project root to apply the Blast Radius Protocol in Cursor.

## What's included

```
.cursor/
  rules/
    blast-radius.mdc    ← core protocol: structure, contracts, versioning, enforcement
    contracts.mdc       ← TS/JS-specific contract format with examples
    recovery.mdc        ← debugging and recovery discipline
  commands/
    verify.mdc          ← /verify slash command: run the full verification gate
```

## How to use

1. Copy `.cursor/` into your project root
2. The rules load automatically for matching files
3. Use `/verify` before declaring any task done

## What this gives you

- **Single-concern file discipline** — smell detector at ~80-100 lines, real rule is concern count
- **Input / Output / Must never contracts** — on every file, enforced by glob
- **Assembly layer discipline** — App.tsx as pure wiring, coupling lives there only
- **Recovery protocol** — diagnostic logging first, single-block rewrites, checkpoint rollback
- **Enforcement under pressure** — the protocol applies hardest when things are going worst
- **Verification gate** — type check, build, isolation, contracts, orphan detection

Full whitepaper: https://github.com/rkz211/blast-radius
