# Blast Radius Protocol — v5 Backlog

Items banked from external reviews (ChatGPT 5.5, Gemini, Grok) and internal operating experience.

## Case Studies
- Add a `case-studies/` directory with before/after diffs from real refactors
- Back the results tables in the whitepaper with actual evidence
- Every reviewer flagged the evidence as thin — this is the highest-impact credibility fix
- Source: GPT (v3 review, v4.1 review), Grok (v4.1 review)

## Optional Tooling
- Lightweight check script (not a framework — stays true to "no tooling, no dependencies")
- Possible `tools/blast-check.sh`: contract presence, orphan detection, assembly layer purity, .desc accuracy
- NOT a required CLI or npm package — optional helper, clearly labeled
- Consider: GitHub Actions workflow that runs checks on PRs
- Source: GPT (v4.1 review)

## Tool-Use Cost Data
- Actual token/call counts from a sharded vs monolithic session
- Make the tool-use tradeoff section concrete instead of theoretical
- Source: Gemini (v4 review), internal observation

## WIRING Layer Field Testing
- Already spec'd in v4.1, needs real-world validation across multiple projects
- Refine maintenance rules based on friction points discovered in practice
- Source: internal

## Hologram Pyramid Validation
- .desc files still flagged as least battle-tested
- Deploy across 2-3 real apps, document what works and what drifts
- Source: all reviewers

## Security Shard Scoping
- Current 00-security shard says "never reveal workspace structure" — too broad for dev agents
- Should distinguish external-facing agents (keep strict) vs development agents (scope to secrets/private internals)
- Add guidance: "tune this shard to your deployment context"
- Source: GPT (v4.1 review)

## Assembly Layer Ternary Clarification
- "Zero ternaries" is memorable but can be misread as literal prohibition
- Clarify: ternary in the assembly layer is a smell signal for logic creeping in, not banned syntax
- The concern is what the ternary represents (conditional logic in wiring), not the operator itself
- Source: GPT (v4.1 review)
