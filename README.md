# Blast Radius Protocol — OpenClaw Agent Pack

Apply this pack to any OpenClaw agent workspace to set up blast-radius-compliant development.

## What's in here

```
BOOTSTRAP.md                    ← run once on first session (Sonnet/Opus only), then self-destructs
blast-radius-v3.md              ← full whitepaper (current — unified: code, scripts, crons, agent files, hologram orientation)
blast-radius-v2.md              ← prior whitepaper (code + scripts + crons + agent files)
openclaw-patch.json             ← openclaw.json patch to activate soul shard loading
soul-shards/
  00-security/SOUL.md           ← identity lock, confidentiality, injection defense
  01-persona/SOUL.md            ← starter template — fill in your agent's persona
  02-ops/SOUL.md                ← hard rules, scope limits, blast radius ops rules
  03-blast-radius/SOUL.md       ← enforcement posture — keeps the agent ON protocol under pressure
memory/
  blast-radius-rules.md         ← operative rules the agent applies when building
  blast-radius-domains.md       ← domain reference table (code/scripts/crons/agent files)
  blast-radius-verification.md  ← verification gate + troubleshooting/recovery discipline
```

## How to apply

### 1. Copy files into your workspace

```bash
cp -r soul-shards/ /path/to/your/workspace/
cp -r memory/ /path/to/your/workspace/memory/
cp BOOTSTRAP.md /path/to/your/workspace/
cp blast-radius-v3.md /path/to/your/workspace/
```

### 2. Fill in your persona

Edit `soul-shards/01-persona/SOUL.md` with your agent's actual persona.

### 3. Apply the openclaw.json patch

Merge `openclaw-patch.json` into your `openclaw.json` under `hooks.internal.entries`.

⚠️ Never edit openclaw.json directly. Write a numbered copy (openclaw.json.001), show the diff to your operator, wait for them to apply it and restart the gateway.

The patch adds the `bootstrap-extra-files` hook which loads all `soul-shards/*/SOUL.md` and `memory-shards/*/MEMORY.md` files in alphabetical order at agent startup.

### 4. Start a new session with Sonnet or Opus

BOOTSTRAP.md runs automatically on first session. It will:
- Verify the model is Sonnet or Opus (stops and tells you to switch if not)
- Confirm all shard files are in place
- Stage the openclaw.json patch for operator review
- Self-destruct when complete

## What this gives you

**A Sonnet or Opus agent with a good critical thinking loop can safely build production software — full-stack web apps, cloud infrastructure, backend engines, and autonomous agent systems — if every artifact is structured so any given edit touches only one concern.**

The agent working on a file cannot accidentally damage adjacent logic because adjacent logic is not in that file.

This pack configures your OpenClaw agent to apply that constraint across four artifact types:

- **Code**: ~80-100 line files, single concern, Input/Output/Must-never contracts, pure assembly layer
- **Scripts**: docstring contracts, self-reporting failures, orchestrators as pure wiring
- **Crons**: one entry per script, no inline logic
- **Agent behavior files**: soul shards loaded in order, memory shards by domain, MEMORY.md for operating rules only

## Read more

See `blast-radius-v3.md` for the current full whitepaper (`blast-radius-v2.md` is retained for history).
Original v1: https://docs.google.com/document/d/1FEloJIRTOzgiUR6mshfGJ_nIq_cohOBIgj369am4FLo
