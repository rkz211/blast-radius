# Case Studies

Before/after evidence from real refactors that applied the Blast Radius Protocol in production. These are the actual commits, contracts, and post-refactor patterns from live systems — not constructed examples.

Each case study includes:
- **The monolith** — what the file looked like before, and what was breaking
- **The refactor** — what sharded, why, and the contracts on each new block
- **The evidence** — the commit, the diff stats, and the post-refactor commit pattern
- **The outcome** — how bug fixes behaved after the shard

## The Case Studies

| # | Module | System | Before | After | Commit |
|---|---|---|---|---|---|
| 1 | [`tick_orchestrator.py`](01-tick-orchestrator.md) | D&D rules engine (Python) | 779 lines | 220-line wiring + 5 modules | [`2fdc7b0`](https://github.com/rkz211/thornwood/commit/2fdc7b0) |
| 2 | [`layer1_rules.py`](02-layer1-rules.md) | D&D rules engine (Python) | 361 lines | 10-line shim + 5 modules | [`338a8c0`](https://github.com/rkz211/thornwood/commit/338a8c0) |
| 3 | [`CharacterPanel.tsx`](03-character-panel.md) | D&D frontend (React/TS) | 531 lines | 160-line wiring + 7 modules | [`c975869`](https://github.com/rkz211/rkz-thornwood/commit/c975869) |
| 4 | [`GraphPage.tsx`](04-graph-page.md) | Force-directed graph viewer (React/TS) | 795 lines | 120-line wiring + 5 modules | [`654044b`](https://github.com/rkz211/rkz-app12/commit/654044b) |

## What to Look For

The line counts are not the point. The point is what happens *after* the refactor:
- Bug fixes touch exactly one file
- New features land in one module without disturbing adjacent logic
- The commit messages name the single block in play — no "fix various issues" scatter
- Zero regression loops (fix → break → fix → break)
