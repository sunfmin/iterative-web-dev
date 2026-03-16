# Continue Workflow — Full Details

This is the primary workflow for every session after initialization.

## Session Startup Sequence

Every coding session should start by:

1. `pwd` — Confirm working directory
2. Read `progress.txt` — Understand what previous sessions did
3. Read `feature_list.json` — See current feature status
4. `git log --oneline -20` — See recent commits
5. Run `bash init.sh` — Start the dev environment
6. Quick verification — Make sure existing features work

## Step-by-Step Process

### Step 1: Get Your Bearings

```bash
pwd
cat progress.txt
cat feature_list.json
git log --oneline -20
```

### Step 2: Start the Development Environment

```bash
bash init.sh
```

If `init.sh` doesn't exist or fails, check the project README or package.json for how to start the dev server. Fix `init.sh` if needed.

**Ensure all required services are running:**
- Database (PostgreSQL, CosmosDB, etc.)
- Backend server on correct port
- Frontend dev server
- Any emulators (blob storage, etc.)

```bash
# Check what services are needed
grep -A 10 "webServer" playwright*.config.ts 2>/dev/null || true
lsof -i :3000  # Check frontend port
lsof -i :8080  # Check backend port
```

### Step 3: Verify Existing Features (Regression Check)

Before implementing anything new, **verify that existing passing features still work**.

Use the E2E Screenshot Verification workflow for comprehensive verification.

If Playwright is not set up yet:
1. Check that the app loads in browser
2. Test basic functionality (login, navigation)
3. If something is broken, **fix it first**

### Step 4: Pick the Next Feature

From `feature_list.json`, find the **highest-priority feature** that has `"passes": false`.

- Work on features in order of priority (high → medium → low)
- Within the same priority, work in the order they appear in the file
- Pick **exactly ONE** feature to work on

Announce which feature you're working on:
> "Working on feature #[id]: [description]"

### Step 5: Implement the Feature

Follow these principles:

- **Write clean, production-quality code** — No shortcuts, no TODOs left behind
- **Follow existing code patterns** — Match the style and architecture already in place
- **Keep changes focused** — Only modify what's needed for this feature
- **Don't break other features** — If your change affects other code, verify it still works

#### Code Modularity and Refactoring

When working with source files, **actively refactor large files**:

**File Size Guidelines:**
- If a file exceeds **300 lines**, consider splitting it
- If a component has **multiple responsibilities**, extract them
- If logic is **reused or could be reused**, extract it

**Refactoring Actions:**
1. **Extract reusable hooks** — `useXxx.ts`
2. **Extract API functions** — `api/xxxService.ts`
3. **Extract types** — `types/xxx.ts`
4. **Extract utilities** — `utils/xxx.ts`
5. **Extract sub-components** — Smaller, focused components

```bash
# Check file sizes before implementing
wc -l src/**/*.tsx src/**/*.ts | sort -n | tail -20
```

### Step 6: Test the Feature with Screenshots

Test using **real verification with screenshots**, not just code review.

```bash
# Run Playwright tests for the feature
npx playwright test --grep "feature-name" 

# Or run all tests
npx playwright test
```

#### List and Review Screenshots

```bash
# Find screenshots from the current scope and feature
# Replace {scope} with active scope name (e.g., "auth", "core")
find e2e/screenshots -name "{scope}-feature-{id}-*.png" -type f | sort

# Check test-results for failure screenshots
find test-results -name "*.png" -type f | sort
```

**IMPORTANT**: Focus on screenshots matching the current scope and feature ID pattern: `{scope}-feature-{id}-*.png`

#### Evaluate Each Screenshot

For each screenshot, check:
1. **Layout** — Does content fit? Any overflow or clipping?
2. **Spacing** — Appropriate padding/margin?
3. **Touch targets** — Buttons/inputs large enough (min 44px)?
4. **Visual hierarchy** — Most important action obvious?
5. **Error states** — Error messages visible, red, associated with input?
6. **Data loading** — Shows real data, not empty/loading states?
7. **Typography** — Text readable? Labels and values distinguishable?
8. **Consistency** — Similar screens use same patterns?

#### Fix Issues Found

If screenshots reveal UX issues:
1. Locate the relevant component
2. Make minimal CSS/layout changes
3. Prefer Tailwind utilities over custom CSS
4. Keep all `data-testid` attributes intact
5. Re-run tests to capture updated screenshots
6. Review again until all issues resolved

### Step 7: Update `feature_list.json`

If the feature passes all tests:
```json
"passes": true
```

**CRITICAL RULES:**
- NEVER remove or edit feature descriptions or test steps
- NEVER change a passing feature back to failing (unless genuine regression)
- ONLY change the `"passes"` field from `false` to `true`

### Step 8: Update `progress.txt`

Append a new session entry:

```
## Session N — [DATE]
### What was done:
- Implemented feature #X: [description]
- Fixed [any bugs]

### Current state:
- Features passing: X / N
- App status: [running/stable/has issues]
- Next priority: Feature #Y — [description]

### Known issues:
- [any issues or blockers]
```

### Step 9: Commit All Progress (Single Commit)

Commit everything together:

```bash
git add -A
git commit -m "feat: [brief description]

- Implemented feature #[id]: [description]
- [any notable implementation details]
- Tests: [X] passing / [N] total"
```

**Important:** All changes go in ONE commit.

### Step 10: Final Verification (MANDATORY)

Before finishing, verify:

1. **All tests still pass** — Run the full test suite one more time
2. **No uncommitted changes** — `git status` should show clean working tree
3. **Codebase is clean** — No half-implemented features, no debug code, no broken imports

## What NOT To Do

- Don't try to one-shot the entire app
- Don't declare the project "done" prematurely — check feature_list.json
- Don't leave the codebase in a broken state
- Don't skip testing — verify features end-to-end as a user would
- Don't modify feature descriptions or test steps in feature_list.json
- Don't implement features out of priority order without good reason
