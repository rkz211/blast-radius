# Case Study 1: tick_orchestrator.py — 779 → 220 + 5 modules

**System:** Thornwood — a procedural AI Dungeon Master with a true D&D 5e physics engine, persistent world state, and authored narration.
**Repo:** Private production codebase
**Commit:** `2fdc7b0` — May 29, 2026

---

## The Monolith

`tick_orchestrator.py` was the heart of the game engine — the function called on every player action. It handled:

- Rest detection and HP restoration
- Death save rolls (d20, natural 20/1, threshold checks)
- Combat state machine (start/end/flag toggling)
- Post-tick persistence (disk writes, AppSync sync)
- Lore extraction from narrative text
- NPC turn coordination
- Result assembly

All 779 lines. One file. Every concern interleaved.

When a bug appeared in death saves, fixing it meant editing inside a file that also contained combat logic, persistence, and rest handling. The agent would fix the death save and introduce a regression in combat state. Fix the combat regression and break persistence. The classic tailspin.

## The Refactor

Six files. Each with a docstring contract. The orchestrator became pure wiring.

### tick_rest.py (82 lines)
```python
"""
tick_rest.py — Rest action handler
Input:  player_action string, campaign_dir string
Output: tick result dict (if rest detected), or None (if not a rest action)
Must never: run combat, call Layer 1-5, or modify narrative outside rest events
"""
```

### tick_death.py (111 lines)
```python
"""
tick_death.py — Death save gate and death check
Input:  character dict, world dict, campaign_dir string
Output: tick result dict (if dead/dying), or None (if alive — caller continues)
Must never: call LLMs, apply combat deltas, or handle non-death state
"""
```

### tick_combat.py (88 lines)
```python
"""
tick_combat.py — Server-side combat state machine
Input:  world dict, character dict, delta dict, player_action string
Output: mutates world in place (combat active/inactive flags)
Must never: call LLMs, apply HP deltas, or narrate — only set combat flags
"""
```

### tick_persistence.py (125 lines)
```python
"""
tick_persistence.py — Save, log, and sync after a completed tick
Input:  campaign_dir, character, world, delta, narrative, npc_narrative, and voice fields
Output: none (side effects only — disk write + AppSync sync)
Must never: modify character/world state, call LLMs, or return narrative
"""
```

### tick_lore.py (69 lines)
```python
"""
tick_lore.py — Discovered lore persistence after Layer 3 narration
Input:  world dict, delta dict, player_action string, narrative string
Output: mutates world["areas"][area_id]["objects"][obj_id]["discovered_detail"] in place
Must never: call Layer 1-3, modify HP/combat state, or write to disk
"""
```

### tick_orchestrator.py (220 lines — wiring layer)
```python
"""
tick_orchestrator.py — Tick wiring layer
Input:  player_action string, campaign_dir string, prev_narrative string
Output: result dict
Must never: contain game logic, prompt engineering, delta math, or narrative text
"""
```

## The Refactor Process

The refactor was performed by an AI agent in a single session with zero errors. Reading the 779-line monolith gave the agent full context of all five concerns; writing each module was straightforward because the output was small and single-purpose. Reading big and writing small is the easy direction.

## The Diff

```
 engine/tick_combat.py       |  93 +++++
 engine/tick_death.py        | 111 ++++++
 engine/tick_lore.py         |  69 ++++
 engine/tick_orchestrator.py | 848 +++++++++++---------------------------------
 engine/tick_persistence.py  |  99 ++++++
 engine/tick_rest.py         |  82 +++++
 6 files changed, 661 insertions(+), 641 deletions(-)
```

## Post-Refactor: What Bug Fixes Look Like

After the shard, subsequent commits that touched tick modules:

| Commit | Files touched | What changed |
|---|---|---|
| `617decb` | tick_combat, tick_persistence | Suppress NPC damage hallucination on non-attack actions; async AppSync sync |
| `054a4d2` | l1_caller (separate module) | L1 combat initiation veto |
| `095e199` | tick_combat | Look/examine no longer trigger NPC provocation |
| `c5d4292` | tick_persistence | Expose actionValid in persisted turns |
| `11ca65e` | l1_caller | NPC turn rolls merged into player response |

Every fix names its target in the commit message. Every fix touches the module that owns the concern. No "fixed tick_death and also had to adjust tick_combat because the combat flags were tangled with the death check" — because they're not tangled anymore.

## Verification

From the commit message:

> Verified: playtest 20/20 checks pass, 0 failures
