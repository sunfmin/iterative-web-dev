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
    4. After subagent completes: VERIFY screenshots and quality (Step 4c)
    5. If quality fails: launch polish subagent
    6. LOOP BACK to step 1
END WHILE
```

#### 4a: Pick the Next Feature

From `feature_list.json`, find the **highest-priority feature** that has `"passes": false`.

- Work on features in order of priority (high -> medium -> low)
- Within the same priority, work in the order they appear in the file
- If a feature is blocked, skip it and come back later

#### 4b: Launch a Subagent for the Feature

Use the **Agent tool** (Claude Code) to launch a subagent for each feature. The subagent handles the **full lifecycle**: implement, UX quality check, test, verify screenshots, and commit. This isolates each feature's work and prevents context window overflow.

Use the subagent prompt template from SKILL.md. The template includes:
- Phase 1: Implement
- Phase 1.5: UX Quality Checklist (NEW — ensures loading/empty/error states, responsive, accessible)
- Phase 2: Refactor & Unit Test
- Phase 3: E2E Test & Visual Verification (ENHANCED — mandatory screenshots + visual review)
- Phase 4: Commit

#### 4c: Verify Subagent Output (MANDATORY)

After the subagent completes, the parent agent MUST verify:

1. **Confirm commit** — `git log --oneline -1`
2. **Confirm feature_list.json** — feature has `"passes": true`
3. **VERIFY SCREENSHOTS EXIST** — This is critical:
   ```bash
   ls e2e/screenshots/{scope}-feature-{id}-*.png 2>/dev/null | wc -l
   ```
   If count is 0, the subagent skipped screenshots. Launch a follow-up subagent:
   ```
   "Add screenshots and visual review for feature #{id}. The feature is already
   implemented and committed, but screenshots are missing. Write E2E tests that
   take fullPage screenshots at key states, run them, then visually review each
   screenshot with the Read tool. Fix any visual issues found."
   ```
4. **SPOT-CHECK one screenshot** — Use the Read tool to open one screenshot from this feature. Evaluate:
   - Does it look polished and professional?
   - Are there loading/empty states where needed?
   - Is spacing and typography consistent?
   - Does it match the design language of other pages?
5. If quality is poor, launch a **polish subagent**:
   ```
   "Polish the UI for feature #{id}. Review all screenshots in
   e2e/screenshots/{scope}-feature-{id}-*.png and fix visual issues:
   [list specific issues found]. Follow references/ux-standards.md and
   references/frontend-design.md for quality standards."
   ```
6. If the subagent failed to complete, launch another subagent to fix and finish.
7. **Loop back** — pick the next incomplete feature and repeat.

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
   - All features have screenshots and visual review verified
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
| Unclear UI design | Follow /frontend-design principles: bold aesthetic, intentional choices |
| UI looks generic/plain | Add visual polish: shadows, transitions, better typography, spacing |
| Missing dependency | Install it |
| Unclear file structure | Follow existing project conventions |
| Subagent skipped screenshots | Launch follow-up subagent to add them |
| Spot-check reveals poor UI | Launch polish subagent to fix visual issues |

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
- Don't skip screenshots or visual review — they are MANDATORY
- Don't accept prototype-quality UI — every page must be polished
- Don't skip the parent spot-check after each subagent completes
