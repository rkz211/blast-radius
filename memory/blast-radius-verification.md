# Blast Radius — Verification & Troubleshooting Discipline
# Input: any build, fix, or debugging task
# Output: the verification gate an artifact must pass before "done", and the recovery protocol when something breaks
# Must never: be skipped to save time, or overridden by a request to "just ship it"

## Why this shard exists

The structural rules (small files, contracts, one-file regression surface) prevent a bad
edit from spreading. This shard governs the two moments where agents still fail even with
good structure: **before declaring done**, and **after something breaks**. Structure limits
the blast radius; this discipline confirms the blast radius is actually zero.

## Known-Good Checkpoint — Always First

Before any non-trivial change, establish a known-good checkpoint you can return to:
- Confirm the current state builds/runs (or note exactly why it doesn't).
- Commit or tag the working state so rollback is a single action.
- Never start a change from an unknown or already-broken state — you cannot measure blast
  radius from a baseline you do not understand.

## Diagnostic Logging First

When you cannot trace an error, add detailed logging before you start changing logic.
- Log raw inputs, key state, and decision points at every hop.
- Prove the root cause from the logs — do not guess and patch.
- Only optimize or remove the logs after the root cause is proven.
A fix applied without a proven cause is a new guess, not a fix.

## The Verification Gate — Pass Before "Done"

An artifact is not done until it passes the gate for its type. Never confirm completion
without verifying output.

**Code (TypeScript / React):**
- `npx tsc --noEmit` passes (type check) before any push that touches TS.
- Local build passes (`npm run build`) before commit.
- For web UI: verify against the LIVE deployed URL (e.g. Playwright), not just locally —
  a local build verifies syntax, it does not deploy.
- Confirm the change is actually live (version stamp, deployed job status) before reporting done.

**Scripts:**
- Run the script and confirm the actual output matches the docstring contract.
- Confirm it took no action its "Must never" line prohibits.

**Crons:**
- Confirm the entry calls exactly one script and contains no inline logic.
- Confirm the called script passes its own script gate.

**Agent files:**
- Confirm only the intended shard changed (`git status` clean except that file).
- Confirm load order is intact (numbered prefixes unchanged).

## Smaller, Verified Changes

- Make the smallest change that addresses the task. Resist touching adjacent code.
- One concern per change. If a fix wants to touch two files, it is two changes.
- Do not assume a function, import, or state setter is in scope — verify it before using it.
  (Classic failure: using a state setter that was never wired into the current hook, then
  pushing without a type check.)

## Recovery Protocol — When Something Breaks

1. Do NOT paste the whole component/page/file and ask "find the bug." That produces a new bug.
2. Identify the single block that owns the broken concern.
3. Read its contract (Input / Output / Must never).
4. Read the logs / build output to prove which block and which line.
5. Rewrite only that block. Everything else stays frozen.
6. Re-run the verification gate for that artifact type.
7. If the change made it worse: revert to the known-good checkpoint immediately. A clean
   revert beats a second speculative fix every time.
