# Continue Workflow — Full Details

This is the primary workflow for every session after initialization. It runs **autonomously until ALL features are complete**.

**CRITICAL: Do NOT stop after implementing one feature. Keep looping until every feature in `feature_list.json` has `"passes": true`. The human may be asleep — make all decisions yourself.**

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

### Step 4: Enter the Autonomous Feature Loop

**This is the core loop. Do NOT exit until all features pass.**

```
WHILE there are features with "passes": false in feature_list.json:
    1. Read feature_list.json
    2. Find the highest-priority feature with "passes": false
    3. Launch a SUBAGENT to implement that feature
    4. Verify with E2E tests
    5. If pass: mark as passing, commit, log progress
    6. If fail: fix and re-test (do NOT skip)
    7. LOOP BACK to step 1
END WHILE
```

#### 4a: Pick the Next Feature

From `feature_list.json`, find the **highest-priority feature** that has `"passes": false`.

- Work on features in order of priority (high -> medium -> low)
- Within the same priority, work in the order they appear in the file
- If a feature is blocked, skip it and come back later

#### 4b: Launch a Subagent for the Feature

Use the **Agent tool** (Claude Code) to launch a subagent for each feature. This isolates each feature's implementation and prevents context window overflow.

**Subagent prompt template:**

```
You are implementing a feature for a web application. Work autonomously — do NOT ask questions, make your best judgment on all decisions.

## Project Context
- Working directory: {pwd}
- Active scope: {scope from .active-scope}

## Feature to Implement
- ID: {id}
- Description: {description}
- Category: {category}
- Priority: {priority}
- Test Steps:
{steps as bullet list}

## Instructions
1. Read the relevant source files to understand the current codebase
2. Read the spec.md file for full project context
3. Implement the feature following existing code patterns
4. Write or update E2E tests in the Playwright test files
5. Follow the screenshot naming convention: {scope}-feature-{id}-step{N}-{description}.png
6. Make sure the implementation is complete and production-quality
7. Do NOT commit — the parent agent will handle commits after verification

## Key Rules
- Follow existing code patterns and architecture
- Keep changes focused on this feature only
- Do not break other features
- Write clean, production-ready code
- Keep files under 300 lines — refactor if needed
- Use data-testid attributes for test selectors
- Make all decisions yourself, never ask for human input
```

#### 4c: Verify the Feature

After the subagent completes, run E2E tests:

```bash
npx playwright test
```

**If tests pass:**
1. Update `feature_list.json` — change `"passes": false` to `"passes": true`
2. Commit (see Step 5)
3. Update `progress.txt` (see Step 6)
4. Continue to next feature

**If tests fail:**
1. Read the error output carefully
2. Fix the issue yourself or launch another subagent to fix it
3. Re-run tests
4. Repeat until tests pass
5. Do NOT skip a failing feature — fix it

### Step 5: Commit After Each Feature

Commit everything together after each successful feature:

```bash
git add -A
git commit -m "feat: [brief description]

- Implemented feature #[id]: [description]
- [any notable implementation details]
- Tests: [X] passing / [N] total"
```

**Important:** All changes for one feature go in ONE commit.

### Step 6: Update progress.txt After Each Feature

Append progress after each feature:

```
## Feature #[id] completed — [DATE]
### What was done:
- Implemented feature #[id]: [description]
- [any fixes or adjustments]

### Current state:
- Features passing: X / N
- Next priority: Feature #Y — [description]
```

### Step 7: Loop Back

**Go back to Step 4 and pick the next feature. Do NOT stop.**

### Step 8: Final Verification (When ALL Features Pass)

Only when every feature has `"passes": true`:

1. **Run full test suite** one final time
   ```bash
   npx playwright test
   ```
2. **Verify clean git status**
   ```bash
   git status
   ```
3. **Update progress.txt** with final session summary:
   ```
   ## Session Complete — [DATE]
   ### Summary:
   - All [N] features implemented and passing
   - Full test suite green
   - Codebase clean and production-ready
   ```
4. **Final commit** if needed

## Decision Making (Autonomous Mode)

Since the human may be asleep, follow these rules:

| Situation | Decision |
|-----------|----------|
| Ambiguous spec | Choose the simplest reasonable interpretation |
| Multiple approaches | Pick the one matching existing patterns |
| Flaky test | Add proper waits/retries, don't skip |
| Feature too large | Break into sub-tasks within the subagent |
| Dependency conflict | Use version compatible with existing packages |
| Build error | Read error, fix it, rebuild |
| Port conflict | Kill conflicting process, restart |
| Database issue | Reset/reseed the database |
| Feature blocked | Skip to next, come back later |
| Unclear UI design | Follow existing UI patterns in the app |
| Missing dependency | Install it |
| Unclear file structure | Follow existing project conventions |

## What NOT To Do

- Don't stop after one feature — keep going until ALL pass
- Don't ask the human what to do — decide yourself
- Don't try to one-shot the entire app
- Don't declare the project "done" prematurely — check feature_list.json
- Don't leave the codebase in a broken state
- Don't skip testing — verify features end-to-end
- Don't modify feature descriptions or test steps in feature_list.json
- Don't implement features out of priority order without good reason
- Don't wait for human approval between features
