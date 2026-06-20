# The Blast Radius Replication Challenge

## Who This Is For

You're in the regression tailspin right now. You asked an AI agent to fix a bug. It fixed the bug and broke something else. You fixed that and broke a third thing. You've been going in circles for an hour — maybe three — and you're further from working than when you started.

You're about to close the laptop and conclude that AI agents can't maintain real software.

Before you do: try this. It takes one session. If it doesn't work, you've lost an hour. If it does, you have your own evidence — not ours, not a whitepaper, not someone else's case study. Yours.

---

## The Test

### Step 1 — Freeze the Scene

You are currently in a regression loop on a specific file (or set of files). Before doing anything else:

```bash
git stash                    # or commit your current broken state
git tag tailspin-state       # mark where you are
git log --oneline -5         # note the last 5 commits for your report
```

Write down:
- Which file(s) are causing the loop
- How many fix attempts you've made in the last hour
- What the original bug was

### Step 2 — Clone Your Environment

Don't apply the protocol to your live working copy. Clone it:

```bash
cp -r my-project my-project-blast-radius
cd my-project-blast-radius
git checkout tailspin-state   # start from the same broken point
```

You now have two copies: the original (where you've been looping) and the test copy (where you'll try the protocol).

### Step 3 — Apply the Protocol to the Problem File

Open the file that's causing the regression loop. It's probably 300+ lines with multiple concerns mixed together.

**Identify the concerns.** Read the file and list what it does. A typical problem file might: fetch data, transform data, manage state, render UI, handle errors — all in one file.

**Tell the agent to shard it:**

```
Read [filename]. It currently contains these concerns: [list them].
For each concern, write a separate file with:
- A three-line contract at the top (Input / Output / Must never)
- Single concern only — no logic from adjacent concerns
- ~80-100 lines

Then rewrite [filename] as a pure wiring layer: imports and composition only,
zero logic, zero state beyond top-level selection.

Do not change any behavior. This is a structural refactor only.
```

**Verify the refactor.** The app should behave identically after sharding. Run your type check, build, and smoke test. If the refactor changed behavior, fix that before proceeding.

### Step 4 — Fix the Original Bug (Again)

Now fix the bug that started the tailspin — but on the sharded version. The agent should:

1. Identify which single-concern file owns the bug
2. Read that file's contract (Input / Output / Must never)
3. Fix only that file — everything else is frozen
4. Verify: type check, build, test

### Step 5 — Observe

Did the fix land without breaking adjacent behavior?

If yes: **the code settled.** That's the state change you're looking for. Not "it compiled" — the system stopped oscillating. Fixes land and stay landed. The regression loop is broken because the regression surface is one file instead of the whole component.

If no: report that too. We need the failures as much as the successes.

---

## Report Your Results

This is the part that matters. Your experience — positive or negative — is the evidence the protocol needs. A controlled test by someone who was actively in a tailspin is worth more than any curated case study.

### Reporting Template

Copy this into a [GitHub Issue](https://github.com/rkz211/blast-radius/issues/new) or a [Discussion post](https://github.com/rkz211/blast-radius/discussions):

```markdown
## Replication Challenge Report

**Date:**
**Agent/Model used:** (e.g., Claude Sonnet 4, GPT-4.1, Codex, Cursor, etc.)
**Platform:** (e.g., VS Code, Cursor, Claude Code, OpenClaw, etc.)
**Language/Framework:** (e.g., React/TypeScript, Python/FastAPI, etc.)

### The Tailspin (Before)
- **Problem file:** (filename and approximate line count)
- **Number of concerns in the file:** (what it was doing — e.g., fetch + transform + render + error handling)
- **Original bug:** (one sentence)
- **Fix attempts before protocol:** (how many rounds of fix → break → fix)
- **Time spent in the loop:** (approximate)
- **Files touched during the loop:** (how many files did the agent modify trying to fix one bug?)

### The Refactor
- **Files created:** (how many single-concern files replaced the monolith)
- **Largest file after sharding:** (line count)
- **Refactor errors:** (did the structural refactor change any behavior? did it need fixes?)
- **Time to shard:** (how long the refactor took)

### The Fix (After)
- **File that owned the bug:** (which single-concern file)
- **Did the fix land clean?** (yes/no — did it resolve the bug without breaking other behavior?)
- **Regressions introduced:** (count — 0 is the target)
- **Follow-up fixes needed:** (count — 0 is the target)
- **Time to fix:** (from "start fixing" to "verified working")
- **Files touched:** (should be 1 — was it?)

### The Verdict
- **Did the code settle?** (yes/no — did the regression loop stop?)
- **Would you use this again?** (yes/no/maybe)
- **What worked:**
- **What didn't work:**
- **What was annoying or unclear about the protocol:**

### Optional: Evidence
- Link to the repo (if public)
- Before/after file structure screenshot
- Git log showing fix commits
- Agent transcript excerpt (if available)
```

### What We Do With Your Report

Every report — positive, negative, or mixed — gets read. We are specifically looking for:

- **Where the protocol fails.** File types, languages, frameworks, or project structures where sharding doesn't help or makes things worse.
- **Where the protocol helps but the docs are unclear.** If you had to guess what to do, the instructions need to be better.
- **Patterns across reports.** Do the results hold across different agents, models, languages, and project sizes?

Negative results are as valuable as positive ones. If you tried this and it didn't help, *that is important data.* We will not filter, cherry-pick, or hide unfavorable reports.

---

## What You're Not Testing

This challenge tests one thing: **does sharding the problem file stop the regression loop?**

You are not testing:
- The Hologram Pyramid (.desc files, orientation layers) — that's a scaling tool for larger systems
- The full verification gate (Playwright, live URL checks) — that's deployment discipline
- Monitoring infrastructure (Part VIII) — that's operational health
- Agent behavior sharding (soul shards, memory shards) — that's agent configuration

Those are all part of the full protocol. This challenge isolates the core mechanism: small files with contracts and frozen adjacent logic. If that core doesn't hold, nothing built on top of it matters.

---

## FAQ

**Do I need to use OpenClaw?**
No. The protocol works with any AI coding agent: Cursor, Claude Code, Copilot, Codex, or any LLM you paste code into. The structural rules are agent-agnostic.

**Do I need to shard my whole codebase?**
No. Shard only the file that's causing the regression loop. The rest of your codebase stays exactly as it is. See [MIGRATION.md](MIGRATION.md) for the full incremental approach.

**What if my problem file is only 150 lines?**
Try it anyway. The protocol's line count (~80-100) is a heuristic. The real rule is concern count. A 150-line file with three concerns will still benefit from sharding. A 150-line file with one concern won't — and that's useful data too.

**What if the refactor itself introduces bugs?**
Report that. A structural refactor that changes behavior is a failure of the refactor, and we need to know when that happens and why.

**What if I'm not in a tailspin right now?**
Bookmark this. Come back when you are. The test only works when you have a real regression loop to compare against. Applying the protocol to a codebase that's working fine proves nothing either way.

---

*The protocol gets better when it fails in public, not when it succeeds in private. Your report — especially if it's bad — is the most valuable contribution you can make.*
