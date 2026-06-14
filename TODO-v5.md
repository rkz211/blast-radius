# Blast Radius Protocol — v5 Backlog

Items banked from external reviews (ChatGPT 5.5, Gemini, Grok) and internal operating experience.
Items resolved in v4.3 are marked ✅.

## Case Studies
- Add a `case-studies/` directory with before/after diffs from real refactors
- Back the results tables in the whitepaper with actual evidence
- Every reviewer flagged the evidence as thin — this is the highest-impact credibility fix
- Source: GPT (v3 review, v4.1 review), Grok (v4.1 review)

## Optional Tooling ✅ (v4.3 — Part VII)
- ~~Lightweight check script (not a framework — stays true to "no tooling, no dependencies")~~
- ~~Possible `tools/blast-check.sh`: contract presence, orphan detection, assembly layer purity, .desc accuracy~~
- ~~NOT a required CLI or npm package — optional helper, clearly labeled~~
- ~~Consider: GitHub Actions workflow that runs checks on PRs~~
- Source: GPT (v4.1 review)
- **Resolved:** Added Part VII — Optional Verification Tooling. Covers contract presence, orphan detection, assembly purity, .desc accuracy. Clearly scoped as optional, not a framework. Includes "what it is not" guardrail.

## Tool-Use Cost Data ✅ (v4.3 — Part V tradeoff section)
- ~~Actual token/call counts from a sharded vs monolithic session~~
- ~~Make the tool-use tradeoff section concrete instead of theoretical~~
- Source: Gemini (v4 review), internal observation
- **Resolved:** Replaced abstract tradeoff with concrete side-by-side scenario (1×400-line monolith vs 5×80-line shards + .desc). Shows read counts, context consumed, blast radius comparison.

## WIRING Layer Field Testing ✅ (v4.3 — Part V field status)
- ~~Already spec'd in v4.1, needs real-world validation across multiple projects~~
- ~~Refine maintenance rules based on friction points discovered in practice~~
- Source: internal
- **Resolved:** Updated field status section with concrete observations. WIRING works when gate is enforced. Dynamic imports noted as edge case with convention (`lazy` annotation). "Newest extension" language replaced with production observations.

## Hologram Pyramid Validation ✅ (v4.3 — Part V field status)
- ~~.desc files still flagged as least battle-tested~~
- ~~Deploy across 2-3 real apps, document what works and what drifts~~
- Source: all reviewers
- **Resolved:** Rewrote honest-status section as "Field Status" with three categories: what works (cold-session recovery, feature grep, WIRING accuracy with gate), what to expect (legacy onboarding friction, dynamic import edge cases, maintenance discipline). No longer flagged as "less battle-tested."

## Security Shard Scoping ✅ (v4.3 — Part IV)
- ~~Current 00-security shard says "never reveal workspace structure" — too broad for dev agents~~
- ~~Should distinguish external-facing agents (keep strict) vs development agents (scope to secrets/private internals)~~
- ~~Add guidance: "tune this shard to your deployment context"~~
- Source: GPT (v4.1 review)
- **Resolved:** Added "Security Shard Scoping" subsection to Part IV. Distinguishes external-facing vs dev agents. Three-tier scoping: always protect (secrets), context-dependent (structure), never restrict (architecture discussion with operator). Explicit "tune to your deployment context" guidance.

## Assembly Layer Ternary Clarification ✅ (v4.3 — Part I)
- ~~"Zero ternaries" is memorable but can be misread as literal prohibition~~
- ~~Clarify: ternary in the assembly layer is a smell signal for logic creeping in, not banned syntax~~
- ~~The concern is what the ternary represents (conditional logic in wiring), not the operator itself~~
- Source: GPT (v4.1 review)
- **Resolved:** Rewrote the ternary paragraph. "Sacred" doesn't mean "syntax banned." Ternary is a smell signal for conditional logic creeping into wiring. Borderline case acknowledged (single boolean section toggle). Protocol position: wiring is safest when purely declarative.

## Five-Layer Reference Table ✅ (v4.3 — Part V)
- Added structured table: Layer | Scope | Purpose | Example
- Cleaner than prose for quick reference
- Source: Grok (v4.1 review)
- **Resolved:** Table added at the top of Part V, before the file structure examples.

## Grep Payoff Examples ✅ (v4.3 — Part V)
- Concrete one-liner grep examples showing feature slug and WIRING grep value
- Source: Grok (v4.1 review)
- **Resolved:** Added "Feature Slugs and the Grep Payoff" subsection with three concrete examples: feature grep, full dependency graph, caller discovery. Each with a one-line explanation of what it replaces.

---

## Remaining for v5

### Case Studies (highest priority)
- Still the #1 credibility gap identified by all reviewers
- Need before/after diffs from actual refactors in production apps
- Planned: `case-studies/` directory with real git diffs

### Actual blast-check.sh Implementation
- Part VII describes the concept; no reference implementation yet
- Consider shipping one in `tools/` — advisory, not blocking

### Cross-Project .desc Adoption Data
- Field status in v4.3 is based on current production use
- Would benefit from data across 2-3 additional distinct projects to strengthen claims

### Platform Adaptation Updates
- Claude Code CLAUDE.md and Cursor rules may need updates to reflect v4.3 changes
- Security shard scoping guidance, ternary clarification, optional tooling awareness
