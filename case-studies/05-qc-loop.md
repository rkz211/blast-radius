# Case Study 5: The QC Loop — Agent-Run Verification Before "Done"

**System:** Mission Control — an internal app registry and site manager for a fleet of AWS Amplify apps. Deployed at a production Amplify URL. Built with React/TypeScript, a Lambda backend (Node.js), and DynamoDB.
**Repo:** Private production codebase
**Session:** June 2026 — Show on Fiction Factory toggle feature

---

## What This Case Study Is About

Case studies 1-4 document what happens to structure after applying the Blast Radius Protocol. This one documents something different: **what happens at the end of a task**.

The protocol's Part VI says:

> "An artifact is not done until it is verified against its deployed state, not its local state."

And:

> "For web apps: Playwright or equivalent browser automation against the production URL, not localhost."

This case study is the evidence for that claim. It documents a real task — implementing and debugging a feature toggle — showing the gates the agent defined, the Playwright session it ran autonomously, and the specific failure it caught before declaring done.

---

## The Task

**Feature:** A "Show on Fiction Factory" toggle in Mission Control. Each registered app has a `showOnFictionFactory` boolean. When toggled on, the site appears in the Fiction Factory public-facing site. Mission Control provides the toggle UI; Lambda stores and serves the value; Fiction Factory reads it.

**Involved blocks:**
- Lambda `index.mjs` — PATCH handler, DynamoDB merge logic
- `SiteRow.tsx` — toggle UI + PATCH call
- `useDashboardSites.ts` — data fetch, `fictionFactorySites` field consumption

Three separate concerns. Three separate files. That structure is the protocol working.

---

## Defining the Gate First

Before writing any code, the agent defined the verification method:

```
Verification gate for this task:
1. Type gate — npx tsc --noEmit passes
2. Build gate — npm run build completes clean
3. Deploy gate — push to GitHub; Amplify build succeeds (check list-jobs)
4. Live verification — Playwright against production URL:
   a. Load Mission Control (authenticated)
   b. Toggle "Show on FF" on for a site that has it off
   c. Confirm toggle state persists after page reload
   d. Load Fiction Factory — confirm the toggled site appears in nav/footer
   e. Toggle off — confirm it disappears from Fiction Factory
5. Version stamp — confirm Lambda version in response headers matches deployed version
```

This is the meta-rule applied: **define the gate before you build**. The five-step list above was written in the agent's reasoning before the first file was opened.

---

## The Build

Three targeted edits. No file touched more than once. Each edit named its block:

**Edit 1 — Lambda PATCH handler** (`index.mjs`)
```
// Input: PATCH /sites/{appId} body with showOnFictionFactory boolean
// Output: updated DynamoDB record; 200 response
// Must never: overwrite fields not present in the PATCH body
```
The merge bug: Lambda was replacing the full DynamoDB record with the PATCH body instead of merging. One block, one bug, one fix.

**Edit 2 — Lambda GET handler** (same file, different function)
```
// Must never: return sites with showOnFictionFactory:false in fictionFactorySites array
```
Added the filter. `fictionFactorySites` in the API response is now pre-filtered — only `showOnFictionFactory === true` entries.

**Edit 3 — SiteRow.tsx**
```
// Input: site object with showOnFictionFactory boolean
// Output: pill toggle UI; fires PATCH on click
// Must never: manage its own data-fetching or cache invalidation
```
Toggle pill connected to PATCH call.

Total: three file edits. Git diff confirmed no other files touched.

---

## The Gates

### Type gate
```
npx tsc --noEmit
✓ No errors
```

### Build gate
```
npm run build
✓ dist/ built clean, no warnings
```

### Deploy gate
```bash
git add -A && git commit -m "feat(showOnFF): toggle + Lambda merge fix" && git push
# Amplify build log polled:
APP=d3uvrs0q5cub5l
JOB=$(aws amplify list-jobs --app-id $APP --branch-name main \
  --max-results 1 --query 'jobSummaries[0].jobId' --output text --region us-west-2)
# Build status: SUCCEED
```

### Live verification — Playwright run

Playwright was installed in the `tommy_app_missioncontrol/` repo. The agent wrote a timestamped test file (`e2e-showonff-${Date.now().toString().slice(-6)}.spec.ts`) to avoid collision with prior runs, then executed it against the production URL.

**Test: toggle on → reload → persists**
```typescript
test('showOnFF toggle persists after reload', async ({ page }) => {
  await page.goto('https://main.d3uvrs0q5cub5l.amplifyapp.com');
  // ... auth flow ...
  const toggleBtn = page.locator('[data-site-id="djacznn9zvjgi"] .ff-toggle');
  await toggleBtn.click();
  await page.reload();
  await expect(toggleBtn).toHaveClass(/toggle-on/);
});
```

**Result: FAIL** — the toggle showed ON after click but reverted to OFF after reload.

---

## What the Playwright Failure Caught

The bug: Lambda's PATCH handler was writing the new `showOnFictionFactory` value correctly to DynamoDB — but the GET handler was not reading DynamoDB for sites that were already present in the static `SITE_CONFIG`. It was returning the hardcoded `SITE_CONFIG` value (`false`) instead of the DynamoDB-updated value.

Without the Playwright reload test, this would not have been visible. The UI responded correctly to the click (optimistic state update), and the initial page load showed the correct DynamoDB value — but only because it happened to cold-start with no cache. On a warm load with the static config in memory, the toggle state was lost.

**Root cause (one sentence):** The merge logic skipped DynamoDB override if the site was already in `statusResults` from the static config.

**Fix:** Always merge DynamoDB `showOnFictionFactory` + `archived` flags on top of `SITE_CONFIG` base — DynamoDB values win on every load, warm or cold.

**One edit. One file. `index.mjs` merge logic.** The three UI files were not touched.

### Playwright run after fix
```
✓ showOnFF toggle persists after reload
✓ site appears in Fiction Factory nav after toggle on
✓ site disappears from Fiction Factory nav after toggle off
✓ toggle off persists after reload

Tests: 4 passed (4)
Time: 12.3s
```

**Task declared done only after this run.**

---

## Why This Matters for the Protocol

### The structure made the bug findable in seconds

The bug lived in exactly one place: the GET handler's merge logic in `index.mjs`. Because Lambda concerns were already sharded (PATCH handler, GET handler, DynamoDB layer are separate function blocks within the file — each with a contract), the agent didn't need to read 400 lines to find the bug. It read the GET handler's `// Must never` line ("must never return stale SITE_CONFIG values for sites with DynamoDB records"), confirmed the violation, and patched exactly that function.

### The gate caught what structure alone couldn't

The Blast Radius structure prevented the bug from spreading when fixed — the patch touched one function block. But structure alone would not have caught the warm-cache/cold-cache discrepancy. That required running against the real deployed URL, reloading the page (which exercises the warm-cache path), and observing the behavior.

This is why the gate matters. Type checks pass on buggy code. Builds pass on buggy code. The Playwright reload test does not.

### The gate was defined before the build

This is the part that is easy to skip and hardest to enforce. The agent defined its five-step verification list before writing the first edit. This meant the Playwright test existed before the code was deployed, and the test was designed to catch exactly the class of bug that appeared — persistent state across page reload — not just "does the toggle button render."

Gates defined after the fact are optimistic. They test what you expect. Gates defined before the build test what could go wrong.

---

## The Full Evidence Chain

| Gate | Tool | Result | Bug Caught? |
|---|---|---|---|
| Type check | `tsc --noEmit` | Pass | No |
| Build | `npm run build` | Pass | No |
| Deploy | Amplify build log | SUCCEED | No |
| Live verification (click only) | Playwright | Pass | No |
| Live verification (click + reload) | Playwright | **FAIL** | **Yes** |
| Fix: DynamoDB merge wins on warm load | One edit, `index.mjs` | — | — |
| Live verification (full suite) | Playwright | 4/4 Pass | — |

The bug was invisible to every gate above it in the chain. Only the reload test — run autonomously by the agent, against the production URL — caught it.

---

## The Post-Fix Commit

```
fix(lambda): DynamoDB showOnFF wins over static SITE_CONFIG on warm load

Merge logic was skipping DynamoDB override for sites already in statusResults.
Fix: always merge DDB showOnFictionFactory + archived on top of SITE_CONFIG base.

Playwright: 4/4 pass (toggle on/off, reload persistence, FF nav appearance)
```

The commit message includes the Playwright result explicitly. This is the evidence stamp — the equivalent of a "playtest 20/20 checks pass" stamp from Case Study 1, but for a web app.

---

## What This Case Study Is Not

This is not a claim that all bugs will be caught by Playwright. Some won't. It is a claim that:

1. The agent can define a concrete verification gate before building
2. The agent can run that gate autonomously against a production URL
3. The gate can catch bugs that pass every earlier check
4. The task is not declared done until the gate passes

The structure limits the blast radius of the bug and the fix. The gate confirms the blast radius is actually zero before closing.

---

*The gate catches what structure can't. Structure limits what the gate has to cover. Together they define "done."*
