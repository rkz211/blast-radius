# Case Study 2: layer1_rules.py — 361 → 10-line shim + 5 modules

**System:** Thornwood — D&D 5e physics engine. Layer 1 is the rules engine that translates player actions into mechanical deltas (HP changes, position updates, combat state transitions) by calling an LLM with structured prompts and parsing the response against a strict schema.
**Repo:** Private production codebase (same system as Case Study 1)
**Commit:** `338a8c0` — May 29, 2026

---

## The Monolith

`layer1_rules.py` was 361 lines containing five distinct concerns:

1. **The LLM caller** — builds the user prompt from world state, calls xAI, parses the response
2. **The delta schema** — defines the JSON structure for HP changes, movement, loot, spell effects, with sign conventions
3. **Core rules prompt** — rules 1-10 covering spell slots, death saves, action economy
4. **Physics rules prompt** — rules 11-18 covering movement, loot, dialogue, physics, secrets, lore
5. **Class features prompt** — all 12 D&D classes with their unique mechanics

When the class features needed updating (e.g., adding Paladin's Lay on Hands), the agent would edit inside a file that also contained the schema definition and the LLM calling logic. A formatting error in the class block could break the prompt assembly. A schema change could break the class features. All five concerns shared one blast radius.

## The Refactor

The monolith became a 10-line re-export shim and five focused modules. Existing importers saw no change — `from layer1_rules import run_layer1` still worked.

### l1_caller.py (232 lines — orchestrator)
```python
"""
l1_caller.py — Layer 1 orchestrator
Input:  world dict, character dict, player_action string
Output: delta dict parsed from LLM response
Must never: define rules, schema, or class features — only assemble prompt and call LLM
"""
```

### l1_schema.py (57 lines)
```python
"""
l1_schema.py — Delta schema and sign conventions
Input:  none (pure definitions)
Output: JSON schema string for LLM structured output
Must never: call LLMs, build prompts, or reference specific rules
"""
```

### l1_prompt_base.py (38 lines)
```python
"""
l1_prompt_base.py — Core rules 1-10
Input:  none (pure prompt text)
Output: rules text covering spell slots, death saves, action economy
Must never: contain class-specific features, physics rules, or schema
"""
```

### l1_prompt_physics.py (59 lines)
```python
"""
l1_prompt_physics.py — Rules 11-18: movement, loot, dialogue, physics, secrets, lore
Input:  none (pure prompt text)
Output: rules text for physical world interactions
Must never: contain class features, core action rules, or schema
"""
```

### l1_prompt_class.py (44 lines)
```python
"""
l1_prompt_class.py — Class features block for all 12 D&D classes
Input:  none (pure prompt text)
Output: class-specific rules text
Must never: contain core rules, physics rules, schema, or LLM calling logic
"""
```

### layer1_rules.py (10 lines — frozen shim)
```python
"""Re-export shim — all importers unchanged."""
from engine.l1_caller import run_layer1
```

## The Diff

```
 engine/l1_caller.py         | 215 ++++++++++++++++++++++++++
 engine/l1_prompt_base.py    |  35 +++++
 engine/l1_prompt_class.py   |  44 ++++++
 engine/l1_prompt_physics.py |  59 +++++++
 engine/l1_schema.py         |  57 +++++++
 engine/layer1_rules.py      | 363 +-------------------------------------------
 6 files changed, 416 insertions(+), 357 deletions(-)
```

## Post-Refactor: What Bug Fixes Look Like

| Commit | Files touched | What changed |
|---|---|---|
| `617decb` | l1_caller | Suppress NPC damage hallucination on non-attack actions |
| `054a4d2` | l1_caller | L1 combat initiation veto — prevent L1 from starting combat on its own |
| `11ca65e` | l1_caller | Merge NPC turn rolls into player response + transparency rule |

All three fixes targeted `l1_caller.py` — the orchestrator that builds prompts and calls the LLM. None of them needed to touch the schema, the core rules, the physics rules, or the class features. The concerns were isolated. The fixes were surgical.

## Why This One Matters

This refactor demonstrates the protocol on **prompt engineering files** — not just traditional code. The LLM prompt text is code. It has the same blast radius problem. When a 44-line class features block lives in the same file as the 57-line schema definition, an edit to one risks the other. Sharding prompt files by concern works exactly like sharding components by concern.
