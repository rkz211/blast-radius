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
| 1 | [`tick_orchestrator.py`](01-tick-orchestrator.md) | D&D rules engine (Python) | 779 lines | 220-line wiring + 5 modules | `2fdc7b0` (private repo) |
| 2 | [`layer1_rules.py`](02-layer1-rules.md) | D&D rules engine (Python) | 361 lines | 10-line shim + 5 modules | `338a8c0` (private repo) |
| 3 | [`CharacterPanel.tsx`](03-character-panel.md) | D&D frontend (React/TS) | 531 lines | 160-line wiring + 7 modules | `c975869` (private repo) |
| 4 | [`GraphPage.tsx`](04-graph-page.md) | Force-directed graph viewer (React/TS) | 795 lines | 120-line wiring + 5 modules | `654044b` (private repo) |
| 5 | [The QC Loop](05-qc-loop.md) | Mission Control (React/TS + Lambda) | Feature toggle bug | Playwright caught warm-cache regression; agent self-verified before done | June 2026 (private repo) |

## What to Look For

Case studies 1-4: the line counts are not the point. The point is what happens *after* the refactor:
- Bug fixes touch exactly one file
- New features land in one module without disturbing adjacent logic
- The commit messages name the single block in play — no "fix various issues" scatter
- Zero regression loops (fix → break → fix → break)

Case study 5 documents something different: **the verification loop**. How the agent defines its own QC gate before building, runs Playwright autonomously against the production URL, catches a bug that passed every earlier check, and only declares done after the gate passes. This is the evidence for Part VI of the whitepaper.
