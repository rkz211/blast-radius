# Case Study 4: GraphPage.tsx — 795 → 120-line wiring + 5 modules

**System:** A force-directed graph viewer rendering an Obsidian-style knowledge graph with tier-based node sizing, physics simulation, zoom/pan interaction, and canvas drawing. Built as part of a personal knowledge OS dashboard.
**Repo:** Private production codebase
**Commit:** `654044b` — May 31, 2026

---

## The Monolith

`GraphPage.tsx` was 795 lines containing five distinct concerns:

1. **Tier classification** — mapping connection counts to visual tiers (radius, color, gravity)
2. **Physics simulation** — force-directed layout with repulsion, attraction, center gravity, cooling
3. **Viewport fitting** — auto-fit transform to center the graph in the canvas
4. **Event handling** — mouse/wheel events for pan, zoom, node selection, hit testing
5. **Canvas drawing** — rendering nodes, edges, labels, selection rings, hover tooltips

All five concerns shared state, shared the canvas context, and shared the animation loop. When the physics simulation needed tuning (repulsion force, cooling rate), the edit landed inside a file that also contained the drawing logic, the event handlers, and the tier calculations. A change to node radius in the tier system would require re-reading the drawing code to check for hardcoded radius assumptions. A change to zoom behavior would risk breaking the physics simulation because both touched the transform state.

## The Refactor

Six files. Each with a three-line contract. The page became wiring + UI chrome.

### graphTiers.ts (35 lines)
```typescript
// Input: connections:number
// Output: tier (0-3), screenRadius, gravityBoost, colorOpacityHex
// Must never: touch canvas, React state, or physics simulation
```

### graphSim.ts (89 lines)
```typescript
// Input: nodes[], edges[], params, alpha
// Output: mutates node x/y/vx/vy positions in place
// Must never: render anything, read canvas dimensions, manage React state
```

### graphFit.ts (33 lines)
```typescript
// Input: nodes[], canvasWidth, canvasHeight, padding?
// Output: {scale, x, y} transform object
// Must never: mutate nodes, touch React, render
```

### graphEvents.ts (76 lines)
```typescript
// Input: canvas mouse/wheel events, transform, nodes
// Output: updated transform (pan/zoom), hit-tested node
// Must never: render, run physics, manage application state beyond transform
```

### graphDraw.ts (147 lines)
```typescript
// Input: ctx, canvas, nodes, edges, transform, params, drawState
// Output: pixels on canvas
// Must never: run physics, manage React state, fetch data
```

### GraphPage.tsx (120 lines — wiring layer)
```typescript
// Input: onNavigate prop
// Output: renders canvas + UI chrome (legend, controls, tooltip)
// Must never: contain physics logic, drawing logic, or tier calculations
```

## The Diff

```
 src/graph/graphDraw.ts   | 111 ++++++
 src/graph/graphEvents.ts |  66 ++++
 src/graph/graphFit.ts    |  33 ++
 src/graph/graphSim.ts    |  89 +++++
 src/graph/graphTiers.ts  |  34 ++
 src/pages/GraphPage.tsx  | 869 +++++++++++------------------------------------
 6 files changed, 534 insertions(+), 668 deletions(-)
```

## Post-Refactor: What Bug Fixes Look Like

14 commits touched the graph modules after the refactor. Here's the single-file discipline in action:

| Commit | Graph files changed | What changed |
|---|---|---|
| `726cf52` | 1 (graphDraw) | Labels only for selected neighborhood + tier-3 hubs, no smear |
| `6c72f55` | 1 (graphDraw) | Thin selection ring instead of bloom, no glow artifact |
| `557a5fa` | 1 (graphEvents) | Cap zoom velocity to prevent blast-past |
| `e23c17a` | 1 (graphEvents) | Hard cap scale at 8x max zoom — prevents viewport escape |
| `55cb722` | 1 (graphEvents) | Tighter velocity cap (0.15), faster friction, stable zoom target |
| `ea01c74` | 2 (graphDraw, graphTiers) | Remove neighbor rings (bloom), reduce hub opacity |
| `79e643c` | 1 (graphTiers) | Log-scale node radius — prevents mega-hub bloom |
| `889f125` | 1 (graphDraw) | Zoom-scaled opacity — low zoom = transparent, high zoom = full color |
| `6881e08` | 1 (graphPlace*) | Incremental placement layout — galaxy grows from center |
| `b5586a2` | 1 (graphPlace*) | Sunflower spiral seeding — replaces center-jitter |
| `f093be7` | 1 (graphPlace*) | Tuning: SCALE_FACTOR 28, r*2.0 bias |
| `b222dfe` | 3 (graphDraw + 2 others) | Cap screen-space node radius, restore edge visibility |
| `7688201` | 1 (graphSim) | Sim settles cleanly at 2/3 hops; stop loop when cooled |

**11 of 14 commits touched exactly 1 graph file.** The remaining 3 touched 2-3 files — and those were coordinated feature changes, not regressions leaking across boundaries.

Note the pattern: three consecutive graphEvents fixes (zoom velocity, max zoom, friction). All three were isolated to the event handler. None of them touched the physics simulation, the drawing logic, or the tier calculations — even though zoom behavior is visually entangled with all of those concerns. The file boundary enforced the isolation that discipline alone would not have maintained.

\* `graphPlace.ts` was added post-refactor as a sixth module (placement/seeding layout). This is the protocol working as designed — new concerns get new files, not appended to existing ones.

## Why This One Matters

This is the **hardest** domain for blast radius discipline. A force-directed graph viewer has deeply coupled concerns: the physics simulation determines node positions, the drawing logic reads those positions, the event handler modifies the transform that the drawing logic uses, and the tier system determines the visual properties that both drawing and simulation reference.

The monolith existed because these concerns feel inseparable. The refactor proved they are not. The coupling between them is real — but the protocol concentrates that coupling in the wiring layer (GraphPage.tsx) and the function signatures (what each module receives and returns). The coupling lives in the interfaces, not in shared file scope.

The result: 14 post-refactor fixes, each naming its target module in the commit message, 11 of them touching exactly one file. No "fixed the zoom and it broke the simulation" regression loops. The concerns are separated. The agent can only break what it can see.
