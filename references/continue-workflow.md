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

Before implementing anything new, **verify that existing passing features still work**. To save time, only run what's needed:

1. **Run all unit tests** (fast):
   ```bash
   npm test  # or the project's unit test command
   ```

2. **Run E2E tests only for features already passing** from previous sessions:
   ```bash
   # Read feature_list.json to find features with "passes": true
   # Run only those feature's E2E tests, e.g.:
   npx playwright test --grep "feature-1|feature-2|feature-3"
   ```
   Do NOT run E2E tests for features that haven't been implemented yet.

3. If anything is broken, **fix it first**

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
    3. Launch a SUBAGENT to implement, test, verify screenshots, and commit
    4. Confirm subagent completed (check git log and feature_list.json)
    5. If subagent failed: launch another to fix and finish
    6. LOOP BACK to step 1
END WHILE
```

#### 4a: Pick the Next Feature

From `feature_list.json`, find the **highest-priority feature** that has `"passes": false`.

- Work on features in order of priority (high -> medium -> low)
- Within the same priority, work in the order they appear in the file
- If a feature is blocked, skip it and come back later

#### 4b: Launch a Subagent for the Feature

Use the **Agent tool** (Claude Code) to launch a subagent for each feature. The subagent handles the **full lifecycle**: implement, test, verify screenshots, and commit. This isolates each feature's work and prevents context window overflow.

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

### Phase 1: Implement
1. Read the relevant source files to understand the current codebase
2. Read the spec.md file for full project context
3. Implement the feature following existing code patterns
4. Make sure the implementation is complete and production-quality

### Phase 2: Refactor & Unit Test
5. Review the code you just wrote and any code you touched. Actively refactor for:
   - **Testability** — Extract pure functions and logic out of UI components and handlers.
     Move business logic, validation, data transformation, and state calculations into
     separate utility/service modules that can be unit tested without DOM or network.
   - **Reusability** — If you see duplicated logic (in your code or existing code you touched),
     extract shared helpers. Don't duplicate what already exists elsewhere in the codebase.
   - **Maintainability** — Keep functions small and single-purpose. Name things clearly.
     Split large files. Prefer composition over deep nesting.
6. Write unit tests for all extracted logic — pure functions, validators, transformers,
   state calculations, business rules. Use the project's existing test framework.
   Run them: npm test (or the project's unit test command) — fix until green.
7. Do NOT unit test UI rendering or things that are better covered by E2E tests.
   Unit tests are for logic; E2E tests are for behavior.

### Phase 3: E2E Test & Verify
8. Write or update E2E tests in the Playwright test files
9. Follow the screenshot naming convention: {scope}-feature-{id}-step{N}-{description}.png
10. Snapshot existing screenshots before running tests:
    find e2e/screenshots -name "*.png" -type f 2>/dev/null | sort > /tmp/screenshots-before.txt
11. Run the feature's E2E tests (not the full suite):
    npx playwright test --grep "feature-{id}"  # or the relevant test file
12. If tests fail: read the errors, fix, and re-run until they pass
13. Find new/changed screenshots:
    find e2e/screenshots -name "*.png" -type f 2>/dev/null | sort > /tmp/screenshots-after.txt
    comm -13 /tmp/screenshots-before.txt /tmp/screenshots-after.txt
    find e2e/screenshots -name "*.png" -newer /tmp/screenshots-before.txt -type f 2>/dev/null | sort
14. Visually review ONLY the new/changed screenshots using the Read tool. Evaluate for:
    - Layout — Content fits? No overflow or clipping?
    - Spacing — Appropriate padding/margins? Not cramped or sparse?
    - Touch targets — Buttons/inputs at least 44px?
    - Visual hierarchy — Most important action obvious? Disabled states clear?
    - Error states — Messages visible and red? Associated with correct input?
    - Data display — Real data, not placeholders? Loading/empty states handled?
    - Typography — Text readable? Labels distinguishable from values?
    - Consistency — Similar screens use same patterns? Colors match theme?
15. If screenshot issues found: fix (minimal CSS changes, keep data-testid), re-run tests, review again

### Phase 4: Commit
16. Update feature_list.json — change "passes": false to "passes": true for this feature
17. Update progress.txt with what was done and current feature pass count
18. Commit all changes:
    git add -A && git commit -m "feat: [description] — Implemented feature #[id]: [description]"

## Key Rules
- Follow existing code patterns and architecture
- Keep changes focused on this feature only
- Do not break other features
- Write clean, production-ready code
- Keep files under 300 lines — refactor if needed
- Extract logic out of components/handlers into testable modules — unit test the logic, E2E test the behavior
- Do not duplicate logic — reuse existing helpers or extract new shared ones
- Use data-testid attributes for test selectors
- Make all decisions yourself, never ask for human input
```

#### 4c: Confirm Subagent Completion

After the subagent completes, the parent agent only needs to:

1. **Confirm** the feature was committed: `git log --oneline -1`
2. **Confirm** `feature_list.json` was updated: check that the feature has `"passes": true`
3. If the subagent failed to complete, launch another subagent to fix and finish
4. **Loop back** — pick the next incomplete feature and repeat

**Do NOT stop. Keep looping until all features pass.**

### Step 8: Final Verification (When ALL Features Pass)

Only when every feature has `"passes": true`:

1. **Run all unit tests**
   ```bash
   npm test
   ```
2. **Run E2E tests for features completed in previous sessions** (regression check — this session's features were already verified by their subagents):
   ```bash
   # Only features that were already passing at session start
   npx playwright test --grep "feature-1|feature-2|..."
   ```
3. **Verify clean git status**
   ```bash
   git status
   ```
4. **Update progress.txt** with final session summary:
   ```
   ## Session Complete — [DATE]
   ### Summary:
   - All [N] features implemented and passing
   - Unit tests and regression E2E tests green
   - Codebase clean and production-ready
   ```
5. **Final commit** if needed

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
