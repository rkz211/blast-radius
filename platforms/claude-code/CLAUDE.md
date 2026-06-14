# CLAUDE.md — The Blast Radius Protocol

This project follows the Blast Radius Protocol. Every change must have minimal, verifiable blast radius. The protocol applies to all artifacts: code, scripts, crons, and configuration.

Full whitepaper: https://github.com/rkz211/blast-radius

v4.3 — June 2026

---

## The Single Rule

**Structure every artifact so that any given edit touches only one concern.** The agent working on a file cannot accidentally damage adjacent logic because adjacent logic is not in the file.

Everything below is how to apply that rule — and how to keep applying it when things go wrong.

---

## Code

### Shard by Concern, Not by Size

Each file: ~80-100 lines, single concern. But the threshold is a heuristic, not a law. A 130-line file with one concern is fine. A 90-line file with two concerns needs splitting. **Size is the smell detector. Concern count is the rule.**

Not "one component" — one concern. A hook that fetches data is separate from a hook that transforms it. A function that renders a token is separate from the function that positions it.

### The Assembly Layer is Sacred

`App.tsx` (or equivalent root) is pure wiring — imports and JSX composition only. Zero logic. Zero state management beyond top-level selection. If a ternary appears in the assembly layer, treat it as a smell signal: conditional logic is creeping into the wiring. The concern that ternary represents — a display toggle, a feature gate, a layout switch — belongs in a block, not in the assembly layer. The word "sacred" does not mean "ternary syntax is banned" — it means the wiring layer is safest when purely declarative.

**Why this matters:** When you shard a 400-line file into five 80-line blocks, you do not delete the complexity — you relocate it into the connections between blocks. The assembly layer is where all of that coupling is allowed to live, and nowhere else. When the wiring is the only place connections live, the wiring is the only place you look to understand them.

**When the assembly layer outgrows itself** (~200+ lines of pure wiring): shard it into section-level sub-assemblies. App.tsx imports sections, each section imports blocks. Two-tier wiring. The protocol applied to itself.

```
App.tsx               ← imports AuthSection, DashboardSection, SettingsSection
AuthSection.tsx       ← imports LoginForm, SignupForm, AuthGate
DashboardSection.tsx  ← imports StatsPanel, ActivityFeed, QuickActions
```

### Every Block Has a Contract

First three lines of every file:

```ts
// Input: what this block receives
// Output: what this block produces or renders
// Must never: the behaviors this block is prohibited from having
```

The "Must never" is not documentation — it is a behavioral constraint. A model that reads "this block must never fetch data" will not fetch data. Write it before you write the implementation.

### The Regression Surface is One File

When something breaks: identify the block, rewrite only that block, everything else frozen. Do NOT paste the whole component and ask "find the bug." That produces a new bug.

### Version Blocks, Not Apps

`useAuthState.v2.ts` alongside the original. Swap the import when verified. Zero-cost rollback. The original stays frozen until the swap is confirmed working.

**Garbage collection is mandatory.** Once v2 is verified and stable, delete v1 in a dedicated commit — a commit that does nothing but remove the dead version. The frozen original is a safety net during the swap, not a permanent fixture.

---

## Scripts

- Each script: ~80-150 lines, one concern.
- Docstring contract at top: Input / Output / Must never.
- Scripts report their own failures. The agent does not diagnose, retry, or message on script failure.
- Orchestrators (pipeline runners, batch scripts): pure delegation only. Zero inline logic. These are the assembly layer for scripts.

---

## Crons / Scheduled Tasks

- One entry = one script call. No inline logic, no pipelines, no chained commands.
- If multiple scripts need the same schedule: write an orchestrator script, schedule calls that.
- Logic lives in the script. The schedule is the trigger only.

---

## Verification — What "Done" Actually Means

An artifact is not done until it is verified against its deployed state, not its local state. A local build proves syntax. A type check proves types. Neither proves the thing works where users will encounter it.

### Code (TypeScript / React / Web Apps)

1. **Type gate** — `npx tsc --noEmit` passes. No commit that touches TypeScript skips this.
2. **Build gate** — `npm run build` completes clean.
3. **Deploy gate** — push to the deploy branch. The artifact is not "pushed" until the deploy pipeline has it.
4. **Live verification** — confirm the change is actually live at the deployed URL. Use browser automation or manual check against production, not localhost.
5. **Version stamp** — if the artifact has a version identifier, confirm the live version matches what was pushed. This catches deploy failures that return 200 with stale content.

### Scripts

1. **Run gate** — execute the script, confirm output matches the docstring contract.
2. **Negative gate** — confirm the script did NOT do anything its "Must never" line prohibits.
3. **Exit code** — confirm 0 on success, non-zero on failure.

### Before Declaring Any Task Done

- **Orphan check** — search for version files (`.v1`, `.v2`, `-v1`, `-v2`) with no live import. If orphans exist, delete them in a dedicated commit. Do this now, not later.
- **Isolation check** — `git diff` shows only the intended files changed. If unrelated files were modified, something leaked.

---

## Known-Good Checkpoint — Always First

Before any non-trivial change:
- Confirm the current state builds/runs (or note exactly why it doesn't).
- Commit or tag the working state so rollback is a single action.
- Never start a change from an unknown or already-broken state.

---

## Diagnostic Logging First

When you cannot trace an error:
- Add detailed logging before changing logic. Log raw inputs, key state, decision points.
- Prove the root cause from the logs — do not guess and patch.
- Remove the logs only after root cause is proven.
- A fix applied without a proven cause is a new guess, not a fix.

---

## Recovery Protocol — When Something Breaks

1. Do NOT paste the whole file and ask "find the bug."
2. Identify the single block that owns the broken concern.
3. Read its contract (Input / Output / Must never).
4. Read the logs / build output to prove which block and which line.
5. Rewrite only that block. Everything else stays frozen.
6. Re-run the verification gate.
7. If the change made it worse: **revert to the known-good checkpoint immediately.** A clean revert beats a second speculative fix every time.

---

## Enforcement — Apply It, Don't Just Know It

This section exists because an agent that has read the rules above will still drift away from them under pressure — when the build keeps failing, when it feels faster to paste the whole file, when "just this once" sounds reasonable.

**The protocol applies hardest when things are going worst.** A cascade of failed builds is the signal to shard tighter and verify more, not to bypass the discipline.

**"Move fast" inside this protocol means:** small verified change, confirmed live. It never means skip the type check, skip the checkpoint, or rewrite a whole file.

**When tempted to break protocol — do the opposite:**
- Tempted to paste the whole file and ask the model to find the bug → instead name the single block, read its contract, rewrite only that block.
- Tempted to push without a type check because "it's a small change" → small changes are exactly the ones that ship broken. Run the gate.
- Tempted to add a fix to an already-oversized file → break the file first, then fix.
- Tempted to declare done because it built locally → it is not done until it is verified live.

**The one-line self-check before every commit:**
*"If this breaks, is the damage contained to one file I can name?"*
If the answer is no, the change is too big — shard it before you ship it.

---

## The Meta-Rule

If you cannot define the verification method for an artifact before you build it, you do not yet understand what "done" means for that artifact. Define the gate first. Then build. Then pass the gate.

---

*No framework. No tooling. No dependencies. Structure the artifacts. Keep the files small. Verify against reality. The model can only break what it can see.*
