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

## Verification by Domain — Concrete Gates

An artifact is not done until it passes the gate for its type. Never confirm completion
without verifying output. **Verify against the deployed state, not the local state.**

### Code (TypeScript / React / Web Apps)
1. **Type gate** — `npx tsc --noEmit` passes. No push that touches TS skips this.
2. **Build gate** — `npm run build` completes clean. Warnings acceptable; errors not.
3. **Deploy gate** — push to deploy branch (GitHub → Amplify, or equivalent). Not "pushed"
   until the deploy pipeline has it.
4. **Live verification** — confirm the change is live at the deployed URL. For web apps:
   Playwright or equivalent browser automation against production, not localhost. A local dev
   server proves the code runs on your machine. The deploy proves it runs on infrastructure.
5. **Version stamp** — if the artifact has a version identifier (agent stamp, build hash,
   deploy ID), confirm the live version matches what was just pushed. Catches deploy failures
   that return 200 with stale content.

### Scripts
1. **Run gate** — execute the script, confirm actual output matches the docstring's Output line.
2. **Negative gate** — confirm the script did NOT do anything its "Must never" line prohibits.
   Check for side effects: files modified, messages sent, external calls made.
3. **Exit code** — confirm 0 on success, non-zero on failure, and the failure path calls
   `report-failure.sh`.

### Crons
1. **Structure gate** — entry calls exactly one script. No inline logic, no pipelines.
2. **Script gate** — the called script passes its own script verification above.
3. **Schedule gate** — confirm the cron expression resolves to the intended time.

### Agent Behavior Files
1. **Isolation gate** — `git diff` or `git status` shows only the intended shard changed.
2. **Load order gate** — numbered prefixes intact and in correct sequence.
3. **Contract gate** — shard's first three lines are present and accurate for current content.

## Orientation Gate (.desc files)

If the project uses the Hologram Pyramid:
1. Every created or modified block has a corresponding `.desc` file.
2. BLOCK line accurately reflects current contract.
3. WIRING line reflects current callers/callees — if a caller was added or removed, the
   callee's `.desc` is updated.
4. New sections have a `section.desc` with APP + SECTION layers.

Agents will skip `.desc` maintenance unless it is part of the gate. This is that gate.

## Orphan Detection Gate — All Domains

Before declaring any task done, regardless of domain:
1. Search for files with version markers (`.v1`, `.v2`, `v1.`, `v2.`, `-v1`, `-v2`).
2. For each, check whether any live file imports, references, or calls it.
3. If an orphan exists with no live reference: delete it in a dedicated commit. Do this in
   the current session. Do not leave it for later — "later" means never for an agent.

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

## The Meta-Rule

If you cannot define the verification method for an artifact before you build it, you do not
yet understand what "done" means for that artifact. Define the gate first. Then build. Then
pass the gate.
