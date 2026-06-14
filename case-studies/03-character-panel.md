# Case Study 3: CharacterPanel.tsx — 531 → 160-line wiring + 7 modules

**System:** Thornwood frontend — a React/TypeScript game interface for the D&D engine. The CharacterPanel renders a full character sheet: HP bar, ability scores, conditions, class-specific resources (spell slots, Ki points, rage charges), and a spell book.
**Repo:** Private production codebase (frontend for Case Studies 1-2)
**Commit:** `c975869` — May 29, 2026

---

## The Monolith

`CharacterPanel.tsx` was 531 lines containing every visual concern of the character sheet:

- HP bar rendering with color-coded fill
- Six-ability-score grid with modifiers
- Condition pills and death save tracker
- Class-specific resource pips for all 12 D&D classes
- Spell slot dot indicators
- Collapsible spell book grouped by level, color-coded by school
- JSON parsing of campaign data into typed props

When the spell book needed color-coding by school, the agent would edit inside a file that also rendered the HP bar, the stat block, and the class resources. A CSS change to spell pills could collide with condition pill styles. A state change for the collapsible spell list could break the class resource toggles. The regression surface was the entire character sheet.

## The Refactor

Eight files. Each with a three-line contract. The panel became pure wiring.

### char/types.ts (49 lines)
```typescript
// Shared types only — SpellEntry, ClassResourceMap interfaces
```

### char/HpBar.tsx (20 lines)
```typescript
// Input:  hp number, hpMax number
// Output: HP bar with color-coded fill and numeric display
// Must never: fetch data, manage state, render anything outside the HP bar
```

### char/StatBlock.tsx (30 lines)
```typescript
// Input:  stats Record<string,number>, mods Record<string,number>
// Output: 6-column ability score grid with modifier + raw value
// Must never: fetch data, manage state, render anything outside the stat grid
```

### char/Conditions.tsx (61 lines)
```typescript
// Input:  conditions string array, deathSaves {successes, failures}, isDying boolean
// Output: condition pills + death save tracker when dying
// Must never: fetch data, manage state, render HP or stats
```

### char/ClassResources.tsx (120 lines)
```typescript
// Input:  cls string, level number, resources ClassResourceMap
// Output: class-specific resource pips and status indicators
// Must never: fetch data, manage state, render spells or stats
```

### char/SpellSlots.tsx (39 lines)
```typescript
// Input:  slots Record<string,number>, max Record<string,number>
// Output: spell slot pip dots grouped by level
// Must never: fetch data, manage state, render spell names or class features
```

### char/SpellBook.tsx (107 lines)
```typescript
// Input:  spells SpellEntry[], slots Record<string,number>, onSpellClick callback
// Output: collapsible spell list grouped by cantrips / level, color-coded by school
// Must never: fetch data, manage slot counts, render HP or class resources
```

### CharacterPanel.tsx (160 lines — wiring layer)
```typescript
// Input:  Campaign object + optional onSpellClick callback
// Output: composed character sheet from sub-components
// Must never: contain display logic, inline styles for individual UI elements,
//             or parse JSON — all parsing lives in this file as the data layer
```

## The Diff

```
 src/components/CharacterPanel.tsx      | 533 +++++----------------------------
 src/components/char/ClassResources.tsx | 120 ++++++++
 src/components/char/Conditions.tsx     |  61 ++++
 src/components/char/HpBar.tsx          |  20 ++
 src/components/char/SpellBook.tsx      | 107 +++++++
 src/components/char/SpellSlots.tsx     |  39 +++
 src/components/char/StatBlock.tsx      |  30 ++
 src/components/char/types.ts           |  49 +++
 8 files changed, 507 insertions(+), 452 deletions(-)
```

## Post-Refactor: What Bug Fixes Look Like

After the shard, subsequent commits that touched the character panel:

| Commit | Files touched | What changed |
|---|---|---|
| `6095787` | SpellBook.tsx | Tap spell to prefill action input |
| `6d691fc` | SpellBook.tsx | Spell pills color-coded by school |
| `901228e` | SpellBook.tsx | SpellBook defaults open for casters, larger slot dots |
| `6748321` | ClassResources.tsx | Per-class feature UI for all 12 classes |
| `8659eec` | CharacterPanel.tsx | Bio fields — ideal/backstory/background |

Five features shipped to the character sheet after the refactor. Each touched exactly one file. The spell book got three consecutive feature commits — all isolated to `SpellBook.tsx`, never touching the HP bar, the stat block, or the conditions tracker. The class resources got a major expansion (all 12 classes) contained entirely within `ClassResources.tsx`.

## Why This One Matters

This is the **frontend equivalent** of the tick_orchestrator refactor. A React component that renders multiple visual concerns is functionally identical to a Python function that handles multiple game concerns — and it has the same blast radius problem. The fix is the same: shard by concern, wire in the assembly layer, freeze everything else.

The "Must never" constraints are especially visible here. A `SpellBook` that "must never render HP or class resources" means an agent asked to change spell rendering literally cannot introduce a regression in the HP display — the HP display is in a different file that isn't loaded.
