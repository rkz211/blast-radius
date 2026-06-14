# Migrating an Existing Codebase to the Blast Radius Protocol

You did not write this codebase under the protocol. The files are large. The concerns are mixed. The agent is already starting to tailspin on fixes. This guide is for that situation.

The goal is not to rewrite everything. The goal is to stop the bleeding first, then improve structure incrementally as you work. You do not need a perfect Blast Radius codebase to get the benefits — you need the files the agent is actively touching to have single concerns and contracts.

---

## The Core Migration Principle

**Do not migrate the whole codebase. Migrate the file that's causing the regression.**

A full upfront refactor of a 10,000-line codebase is a large risky project with unclear payoff timing. A targeted refactor of the 600-line file that's been responsible for the last four regression loops pays for itself immediately.

The protocol is designed so you can adopt it one file at a time, starting wherever the pain is.

---

## Step 1 — Establish a Known-Good Checkpoint

Before touching anything:

```bash
git status          # confirm clean working tree
npm run build       # confirm it builds (or note exactly why it doesn't)
git tag known-good  # or just note the current commit hash
```

If the codebase doesn't build right now, fix that first. You cannot measure improvement from an unknown starting state, and you cannot safely revert to a broken baseline.

---

## Step 2 — Find the File That Needs to Change First

You are looking for the file where the agent keeps creating regressions. Indicators:

```bash
# Files with the most recent fix commits
git log --oneline --follow -- src/ | grep "fix\|bug\|regression" | head -20

# Files changed most frequently in the last 30 commits
git log --oneline -30 --name-only | grep "\.tsx\|\.ts\|\.py" | sort | uniq -c | sort -rn | head -10

# Files over the ~700-line warning threshold
find src/ -name "*.tsx" -o -name "*.ts" | xargs wc -l | sort -rn | head -10
```

The file at the top of all three lists is your first target.

---

## Step 3 — Refactor That File (Reading Big, Writing Small)

The refactor itself is the easy part. Reading a large file and decomposing it into small focused modules is the direction LLMs are good at. The agent reads the full monolith with complete context, then writes each module individually — small output, single concern.

**The prompt pattern that works:**

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

**What you get:** The same behavior, split across N small files. Each file has a contract. The original file becomes a thin assembly layer.

**Case Study 4 — GraphPage.tsx (795 lines → 120-line wiring + 5 modules)** was done exactly this way, by an AI agent, in a single session, with zero errors. The agent read 795 lines of mixed concerns and wrote five focused modules. The refactor took one session. The next 14 bug fixes each named exactly one target file.

---

## The Bug-Fix Prompt Pattern

The single most common operation during migration: the agent needs to fix a bug in a file that hasn't been migrated yet. Use this pattern to get the benefit of the protocol without a full refactor first.

```
Before touching anything:
1. Read [filename] and identify which concern owns this bug.
2. Write a three-line contract for that concern:
   // Input: ...
   // Output: ...
   // Must never: ...
3. Add the contract to the top of the file.
4. Now fix only the behavior described by that contract.
   Do not touch adjacent logic. Do not restructure the file.
   The regression surface is the concern you just named — nothing else.
5. After the fix: confirm git diff shows only [filename] changed.
```

This gives you the protocol's core benefit — named blast radius, frozen adjacent logic — without requiring a full shard first. The contract forces the agent to articulate the boundary before it starts editing. That one step eliminates most of the "fixed one thing, broke another" failures.

Once a file has been through this pattern two or three times, it is usually obvious how it should be sharded — the concern boundaries reveal themselves through the contracts you've been writing.

---

## Step 4 — Add Contracts to Files You Touch

You do not need to add contracts to every file in the codebase immediately. Add them to files as you touch them:

- **Refactoring a file?** Add the contract as part of the refactor.
- **Fixing a bug?** Add the contract before you make the fix — it forces you to articulate what the file is supposed to do, which often reveals why the bug exists.
- **Reviewing a file the agent just modified?** Check whether it has a contract. If not, add one now.

The contract format:

```ts
// Input: what this block receives
// Output: what this block produces or renders
// Must never: the behaviors this block is prohibited from having
```

**The "Must never" line is the most important one.** It names the boundary. "Must never fetch data" means the agent reading this file will not add a fetch call. Write it before you write the fix — it sets the constraint for the session.

---

## Step 5 — Add .desc Files Only Where You Need Orientation

The Hologram Pyramid (`.desc` files) is valuable on large sharded systems where cold context reconstruction is expensive. On a partially-migrated codebase, do not try to `.desc` everything at once.

Start with **section `.desc` files** for the areas you're actively working in:

```
src/auth/
  auth.section.desc       ← add this first: APP + SECTION summary
  useAuthState.ts
  useAuthGate.ts
```

Then add **block `.desc` files** as you touch each block. A block without a `.desc` file is fine — it just means the agent has to read the file to understand it. A block with an accurate `.desc` file means the agent can orient from the summary alone.

Do not add `.desc` files to blocks you are not touching. An inaccurate `.desc` is worse than none.

---

## Step 6 — Run blast-check.sh as You Go

```bash
# Check the files you just touched
bash tools/blast-check.sh src/auth/

# Check only git-changed files
bash tools/blast-check.sh --changed

# Run as a pre-commit advisory check
BLAST_STRICT=1 bash tools/blast-check.sh --changed
```

The script catches mechanical issues: missing contracts, orphaned version files, assembly layer logic, incomplete `.desc` files. It will not catch contract inaccuracies — only you or the agent can judge those.

During migration, expect findings. The goal is not to go green immediately — it is to go green on the files you are actively working in, and let the rest of the codebase improve gradually.

---

## Step 7 — Handle the Assembly Layer Last

The assembly layer (`App.tsx` or equivalent) is often the last file to clean up in a migration because it accumulates logic over time. Do not try to clean it first — the logic it contains is coupling that needs somewhere to go, and that somewhere is the blocks you haven't written yet.

Clean the assembly layer after the blocks exist:
1. Identify each piece of logic in the assembly layer
2. Find or create the block it belongs to
3. Move it there
4. The assembly layer gets smaller with each move

When the assembly layer is pure wiring — imports and JSX composition only — the migration is complete for that section.

---

## Migration Priority Order

If you are not sure where to start, use this order:

1. **The file causing the current regression** — highest immediate payoff
2. **Files over 500 lines** — these are the ones that will cause the next regression
3. **The assembly layer** — once the blocks exist
4. **Everything else** — as you touch it, not proactively

Do not migrate files that are working fine and not being touched. The protocol's value comes from the files the agent is actively modifying. A stable 400-line file that nobody is touching is not a problem.

---

## Common Migration Mistakes

**Trying to migrate everything at once.** The result is a partially-migrated codebase in an unstable state, a large undifferentiated diff, and no clear checkpoint to revert to. Migrate one file per session.

**Creating tiny files without contracts.** File count is not the goal. A 40-line file with no contract and mixed concerns is worse than a 150-line file with a clear contract and single concern. Contract first, then size.

**Adding .desc files to files you haven't read.** A `.desc` file you wrote from the filename alone will be wrong. Only write `.desc` files for blocks you have actually read and understood.

**Migrating the assembly layer before the blocks.** The assembly layer is a destination for complexity, not a starting point. Build the blocks first.

**Declaring a file "migrated" after splitting it without verifying behavior.** The structural refactor must produce identical behavior. Run your full verification gate — type check, build, at minimum a smoke test — before closing the refactor commit. The refactor is not done until it is verified.

---

## The Incremental Progress Signal

You will know the migration is working when:

- Bug fix commits start naming a single target file in the message
- The agent stops needing to read five files to fix a bug in one
- Regression chains (fix → break → fix → break) shorten or disappear
- `git log --oneline -- src/[file]` shows fixes landing and staying fixed

You will not see these signals on files that haven't been migrated yet. That is fine. The benefit is proportional to the coverage of the files the agent is actively working in.

---

## Realistic Timeline

| Phase | What happens | When |
|---|---|---|
| Known-good checkpoint | One command | Day 1, first 5 minutes |
| First file refactored | One agent session | Day 1 |
| Contracts on touched files | Ongoing, per session | Weeks 1-4 |
| Section .desc files | As you work in each section | Weeks 2-6 |
| Assembly layer clean | After blocks exist | Weeks 4-8 |
| Full codebase coverage | If you get there | Months — and that's fine |

Full coverage is not required to get full benefit on the files you're actively maintaining. A codebase where 30% of the files are Blast Radius compliant — but that 30% is the 30% the agent is touching — will have dramatically fewer regressions than a 100% non-compliant codebase.

---

*The wall is not the whole codebase. It is the file in front of you right now. Shard that one. The rest follows.*
