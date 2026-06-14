# The Blast Radius Protocol

## The Problem This Solves

There's a moment every developer using AI agents hits. The agent built the thing — beautifully, autonomously, fast. Features shipped. It worked. Then something breaks, and you realize the agent that designed and built it can't actually debug and fix it.

You watch it thrash. It rewrites adjacent logic. It touches imports it didn't need to touch. It fixes one bug and introduces two more. You fix those and break something that was working an hour ago. You're three hours into what should have been a ten-minute fix, and you're further from working than when you started.

**That's the panic.** Not that the agent made a mistake — mistakes are fine. The panic is that it *can't stop making them*. Every fix creates a new regression. Every regression creates a new fix. It circles and circles. The agent that built your system is now dismantling it in front of you, and you can't intervene fast enough to stop it.

The natural conclusion is: "AI can build but it can't maintain. It's a prototype tool, not a production tool. Autonomous development doesn't work for real systems."

We were ready to reach that conclusion. Then we tried something different: instead of fixing the agent, we fixed the structure. We broke every file down until each one had exactly one concern, put a contract at the top saying what it does and what it must never do, and made a rule that when something breaks, you rewrite only that file — everything else stays frozen.

**The tailspin stopped.** Not gradually — it just stopped, like it had never been there. The agent couldn't create regressions in adjacent logic because adjacent logic wasn't in the file. Fixes either worked or they didn't, but they never broke something two components away.

The agent didn't get smarter. The structure got smaller. That's the entire protocol.

---

## The Operating Model This Is Built For

This protocol is designed for a specific way of working: **the human is the product owner, the agent is the developer and maintainer.** The human sets direction, makes decisions, and holds the vision. The agent builds, debugs, ships, and verifies — autonomously, without someone reviewing every line.

That operating model breaks down at the 700-line wall. Once a file gets large enough, the agent can't fix one thing without breaking another. The regression tailspin starts. You're suddenly paying human-developer rates to babysit an agent that's dismantling what it built.

**The onboarding cost question misses the point.** Critics ask: "Is the protocol overhead worth it?" That's the wrong comparison. The right comparison is:

| Approach | Cost to onboard | Cost per regression | Who pays the maintenance bill? |
|---|---|---|---|
| No structure | Zero | Human debugging session | You, every time |
| This protocol | One refactor session | Near zero | The agent, autonomously |
| Human developer | Hiring + ramp-up | Human debugging session | Payroll |

A human developer can maintain a 700-line monolith. Slowly. Expensively. That's not the goal here. The goal is an agent that dramatically reduces the regression cost of maintenance — and the protocol is what removes the wall that makes that possible. In our experience, regression loops drop close to zero on sharded codebases. We don't claim that generalizes universally; we claim it has held consistently across our own systems.

If you want AI as a "fast autocomplete for humans," you don't need this. If you want AI doing the actual development work while you stay at the product level, this is what keeps that working past the point where it would otherwise break.

**One warning before you adopt:** the verification discipline is the core of this protocol, not the small files. Small files with stale contracts and no verification gate is file sprawl — it looks like the protocol and does not work like it. The structure limits the blast radius. The verification gate confirms it's actually zero. You need both. If you are only going to adopt part of this, adopt the verification gates and known-good checkpoints first — those pay off immediately regardless of file size.

## Who This Is For

You need this if:
- Your AI agent keeps creating regressions when fixing bugs in a codebase that's grown past trivial
- You're stuck in loops where every fix breaks something else
- You've started to believe autonomous AI development doesn't work for real, maintained systems
- You want surgical bug fixing to completion on a live, in-use system — without a human reviewing every line

You don't need this if:
- You're building a weekend prototype you'll throw away
- Your codebase is small enough that the agent can hold it all in context without drifting
- You want human-in-the-middle review on every change — this protocol is optimized to minimize that, not support it

**Already have a codebase?** See [`MIGRATION.md`](MIGRATION.md) for how to apply the protocol incrementally without a full rewrite.

---

## What It Does

The protocol applies one rule across everything the agent touches — code, scripts, crons, and the agent's own behavior files:

**Every artifact is structured so that any given edit touches only one concern.**

That means:
- **Code**: ~80-100 line files (heuristic, not law — concern count is the real rule), each with a contract declaring what it does and what it must never do. A sacred assembly layer (App.tsx) that is pure wiring — imports and composition only, zero logic. When something breaks, you rewrite one file. Everything else is frozen.
- **Scripts**: Same discipline. One script, one job. Orchestrators are pure delegation.
- **Crons**: One entry, one script call. No inline logic.
- **Agent behavior files**: Sharded into small, numbered modules loaded in order. Updating the agent's persona can't accidentally damage its security rules because they're in different files.
- **Verification**: Concrete gates that define what "done" means — type check, build, deploy, live URL confirmation. Not vibes. Not "it compiled locally."

The protocol also includes an enforcement mechanism — a dedicated behavioral shard that keeps the agent on protocol under pressure. Because the moment things start breaking is exactly when agents abandon discipline, and that's exactly when discipline matters most.

---

## Platform Support

The full protocol is designed for OpenClaw agent workspaces, but the core ideas are portable. First-party adaptations:

| Platform | Location | Setup |
|---|---|---|
| **Claude Code** | `platforms/claude-code/CLAUDE.md` | Copy into your project root |
| **Cursor** | `platforms/cursor/.cursor/` | Copy `.cursor/` folder into your project root |
| **OpenClaw** | Root of this repo | Full pack with soul shards, memory shards, bootstrap |

The Claude Code and Cursor versions carry the protocol's actual contributions — enforcement under pressure, assembly layer discipline, the coupling tradeoff, verification against deployed state — not just "write clean code" advice.

---

## The Full Protocol (OpenClaw)

### What's in here

```
MIGRATION.md                    ← how to apply the protocol to an existing codebase
blast-radius-v4.3.md            ← full whitepaper (current v4.3)
blast-radius-v4.md              ← prior version (retained for history)
blast-radius-v3.md              ← prior version (retained for history)
blast-radius-v2.md              ← prior version (retained for history)
case-studies/                   ← before/after evidence from real refactors (5 studies)
openclaw-patch.json             ← openclaw.json patch to activate soul shard loading
BOOTSTRAP.md                    ← run once on first session, then self-destructs
soul-shards/
  00-security/SOUL.md           ← identity lock, confidentiality, injection defense
  01-persona/SOUL.md            ← starter template — fill in your agent's persona
  02-ops/SOUL.md                ← hard rules, scope limits, blast radius ops rules
  03-blast-radius/SOUL.md       ← enforcement posture — keeps the agent ON protocol under pressure
memory/
  blast-radius-rules.md         ← operative rules the agent applies when building
  blast-radius-domains.md       ← domain reference table
  blast-radius-verification.md  ← verification gates + troubleshooting/recovery discipline
platforms/
  claude-code/CLAUDE.md         ← drop-in for Claude Code
  cursor/.cursor/
    rules/blast-radius.mdc      ← core protocol rules (auto-loaded)
    commands/verify.mdc         ← /verify slash command: full gate including type check, build, accuracy
tools/
  blast-check.sh                ← advisory script: mechanical checks only (contract presence,
                                   orphan detection, assembly purity, .desc coverage)
                                   Run: bash tools/blast-check.sh
                                   /verify and blast-check.sh are complementary:
                                   the script catches what a script can catch;
                                   /verify covers what only a human or agent can judge
```

### How to apply (OpenClaw)

1. Copy `soul-shards/`, `memory/`, `BOOTSTRAP.md`, and `blast-radius-v4.3.md` into your workspace
2. Fill in `soul-shards/01-persona/SOUL.md` with your agent's persona
3. Merge `openclaw-patch.json` into your `openclaw.json` (see patch file for instructions)
4. Start a new session with Sonnet or Opus — BOOTSTRAP.md runs automatically

---

## Why This Is Public

This protocol has not been validated by anyone but its authors. We built it because our agents were falling apart at scale, and this fixed it. We could have kept using it quietly.

But if you're the developer who just watched your agent dismantle something it built yesterday — if you're searching for a solution to a problem you're not even sure has a name — we want to know if this helps. And if it does, we hope you'll share what you learn back.

The case studies are real. The commit hashes are real. The results are from production systems we run. But this is one team's experience on one team's projects. The protocol needs other people building with it, breaking it, and telling us where it doesn't hold. That's why it's here.

If you try it: [open an issue](https://github.com/rkz211/blast-radius/issues), start a discussion, or just tell us what happened. The protocol gets better when it fails in public, not when it succeeds in private.

---

## Read More

See `blast-radius-v4.3.md` for the current full whitepaper (v4.3). Prior versions are retained for history.

Original v1: https://docs.google.com/document/d/1FEloJIRTOzgiUR6mshfGJ_nIq_cohOBIgj369am4FLo
