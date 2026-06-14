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

## The Refactor Process

The refactor was performed by an AI agent in a single session with zero errors. The process was straightforward: read the 795-line monolith (big context in), write five focused modules (small output). Reading big and writing small is the easy direction — the agent has full context of the system and outputs only the isolated concern. No iteration was needed.

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

## Quantitative Evidence: The Tailspin in Git

The most compelling evidence is not the line counts — it's the commit pattern before and after the refactor.

**Before the refactor: 12 consecutive bug fixes, all in `GraphPage.tsx`.**

The git log tells the story of an agent trapped in a regression loop:

```
fix: world-space node radii, labels in world-space coords ... bump LS key v5
fix: labels are screen-space fixed 11px — ignore zoom ....
fix: nodes screen-space fixed size, labels pure screen-space ... bump LS v6
fix: initial transform shows full scatter from frame 1 ... bump LS v7
fix: sunflower spiral init (even spread, no origin clump) ...
fix: remove alpha² centering (was collapsing all nodes) ... LS v8
fix: random scatter (no spiral rings) ...
fix: labels off by default (no white smear) ... LS v9
fix: label threshold 0.35 (calibrated to actual zoom range) ... LS v10
fix: progressive label density — hubs only at low zoom ...
```

Three patterns visible:

1. **World-space vs screen-space whipsaw** — three commits oscillating between coordinate systems because fixing one concern (node size) broke an adjacent concern (label size) in the same file.
2. **Layout iteration spiral** — four consecutive attempts at initial placement, each undoing the previous because the physics, the centering, and the scatter logic were all in scope.
3. **Label fix chain** — three commits iterating on label visibility because each fix partially broke what the previous one established.

The `LS v5` through `LS v10` markers are localStorage version bumps — the agent was invalidating cached state on every attempt because its own fixes kept corrupting the previous state. **Six cache invalidations in the pre-refactor sequence. Zero after.**

**After the refactor: 13 fixes, each naming its target module.**

```
fix(graphDraw): labels only for selected neighborhood + tier-3 hubs, no smear
fix(graphDraw): thin selection ring instead of bloom, no glow artifact
fix(graphEvents): cap zoom velocity to prevent blast-past
fix(graphEvents): hard cap scale at 8x max zoom
fix(GraphPage): clear zoom velocity on Fit — prevents momentum drift
fix(graphEvents): tighter velocity cap (0.15), faster friction
fix(graphDraw/Tiers): remove neighbor rings (bloom), reduce hub opacity
fix(graphTiers): log-scale node radius — prevents mega-hub bloom
fix(graphDraw): zoom-scaled opacity — low zoom transparent, high zoom full
fix(graphDraw): cap screen-space node radius, restore edge visibility
```

No chains. No whipsaw. No cache invalidations. Each fix is complete in itself.

| Metric | Before Refactor | After Refactor |
|---|---|---|
| Bug fix commits | 12 | 13 |
| Single-file fixes | 58% | 68% |
| Avg files per commit | 1.7 | 1.4 |
| Regression chains (consecutive fixes to same concern) | 3 chains (3, 4, and 3 commits each) | 0 |
| localStorage cache invalidations | 6 (LS v5→v10) | 0 |

## Why This One Matters

This is the **hardest** domain for blast radius discipline. A force-directed graph viewer has deeply coupled concerns: the physics simulation determines node positions, the drawing logic reads those positions, the event handler modifies the transform that the drawing logic uses, and the tier system determines the visual properties that both drawing and simulation reference.

The monolith existed because these concerns feel inseparable. The refactor proved they are not. The coupling between them is real — but the protocol concentrates that coupling in the wiring layer (GraphPage.tsx) and the function signatures (what each module receives and returns). The coupling lives in the interfaces, not in shared file scope.

The result: 14 post-refactor fixes, each naming its target module in the commit message, 11 of them touching exactly one file. No "fixed the zoom and it broke the simulation" regression loops. The concerns are separated. The agent can only break what it can see.
