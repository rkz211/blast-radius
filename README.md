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

## Who This Is For

You need this if:
- Your AI agent keeps creating regressions when fixing bugs in a codebase that's grown past trivial
- You're stuck in loops where every fix breaks something else
- You've started to believe autonomous AI development doesn't work for real, maintained systems
- You want surgical bug fixing to completion on a live, in-use system — without a human reviewing every line

You don't need this if:
- You're building a weekend prototype you'll throw away
- Your codebase is small enough that the agent can hold it all in context without drifting
- You're happy with human-in-the-middle review on every change

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
blast-radius-v4.3.md            ← full whitepaper (current v4.3)
blast-radius-v4.md              ← prior version (retained for history)
blast-radius-v3.md              ← prior version (retained for history)
blast-radius-v2.md              ← prior version (retained for history)
case-studies/                   ← before/after evidence from real refactors
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
  cursor/.cursor/               ← drop-in rules + /verify command for Cursor
```

### How to apply (OpenClaw)

1. Copy `soul-shards/`, `memory/`, `BOOTSTRAP.md`, and `blast-radius-v4.3.md` into your workspace
2. Fill in `soul-shards/01-persona/SOUL.md` with your agent's persona
3. Merge `openclaw-patch.json` into your `openclaw.json` (see patch file for instructions)
4. Start a new session with Sonnet or Opus — BOOTSTRAP.md runs automatically

---

## Read More

See `blast-radius-v4.3.md` for the current full whitepaper (v4.3). Prior versions are retained for history.

Original v1: https://docs.google.com/document/d/1FEloJIRTOzgiUR6mshfGJ_nIq_cohOBIgj369am4FLo
