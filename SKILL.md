---
name: iterative-web-dev
description: Manage long-running AI agent development projects with incremental progress, scoped features, and E2E verification. Use this skill when working on multi-session projects, implementing features incrementally, running E2E tests with screenshots, initializing project scopes, or continuing work from previous sessions. Triggers on phrases like "continue working", "pick up where I left off", "next feature", "run E2E tests", "verify with screenshots", "initialize scope", "switch scope", "feature list", "incremental progress", or any multi-session development workflow.
---

# Iterative Web Development Workflow

This skill provides a complete workflow for AI agents working on long-running development projects across multiple sessions. It ensures **incremental, reliable progress** with proper handoffs between sessions.

## Core Principles

1. **Incremental progress** — Work on ONE feature at a time. Finish, test, and commit before moving on.
2. **Feature list is sacred** — `feature_list.json` is the single source of truth. NEVER remove or edit feature descriptions.
3. **Git discipline** — Commit after every completed feature. Never leave uncommitted work.
4. **Clean handoffs** — Every session ends with committed work and updated progress notes.
5. **Test before build** — Verify existing features work before implementing new ones.
6. **Autonomous execution** — Make all decisions yourself. Never stop to ask the human. The human may be asleep.
7. **Subagent per feature** — Each feature is implemented in its own subagent for isolation and parallelism safety.
8. **Refactor and unit test** — Actively extract logic into testable modules and write unit tests. Keep code reusable and maintainable.

## When to Use Each Workflow

| Workflow | Use When |
|----------|----------|
| **init-scope** | Starting a new scope, switching scopes, or setting up project structure |
| **continue** | Every session after init — picking up work, implementing ALL remaining features, and verifying each with E2E screenshot review |

---

## Workflow: Initialize Scope

Use this to create a new development scope or switch between existing scopes.

### Concepts

- **Scope**: A focused set of features (e.g., "auth", "video-editor", "phase-2")
- **Active Scope**: Currently active scope stored in `.active-scope`
- **Scope Files**: `specs/{scope}/spec.md` and `specs/{scope}/feature_list.json`

### Directory Structure

```
project-root/
├── specs/
│   ├── auth/
│   │   ├── spec.md
│   │   └── feature_list.json
│   └── video-editor/
│       ├── spec.md
│       └── feature_list.json
├── .active-scope
├── spec.md              # Symlink to active scope
├── feature_list.json        # Symlink to active scope
├── progress.txt
└── init.sh
```

### Steps

1. **Check current state**
   ```bash
   ls -la specs/ 2>/dev/null || echo "No scopes yet"
   cat .active-scope 2>/dev/null || echo "No active scope"
   ```

2. **Create new scope** (if needed)
   ```bash
   mkdir -p specs/auth
   # Create specs/auth/spec.md with specification
   ```

3. **Switch to scope**
   ```bash
   echo "auth" > .active-scope
   ln -sf specs/auth/spec.md spec.md
   ln -sf specs/auth/feature_list.json feature_list.json
   ```

4. **Create feature list** from spec — see `references/feature-list-format.md`

5. **Create/update init.sh** — see `references/init-script-template.md`

6. **Commit and update progress log**

---

## Workflow: Continue Session (Autonomous Feature Loop)

This is the main workflow. It runs ALL remaining features to completion without stopping.

**IMPORTANT: You MUST keep looping until every feature in `feature_list.json` has `"passes": true`. Do NOT stop after one feature. Do NOT ask the human what to do next. The human may be asleep.**

### Session Startup Sequence

1. **Get bearings**
   ```bash
   pwd
   cat progress.txt
   cat feature_list.json
   git log --oneline -20
   ```

2. **Start environment**
   ```bash
   bash init.sh
   ```

3. **Verify existing features** — Run all unit tests (fast) and only the E2E tests for features completed in previous sessions (not this session's new work). Skip E2E tests for features not yet implemented.

### Autonomous Feature Loop

After startup, enter the **feature loop**. This loop runs until ALL features pass:

```
WHILE there are features with "passes": false in feature_list.json:
    1. Read feature_list.json to find next incomplete feature (highest priority first)
    2. Launch a SUBAGENT to implement, test, verify screenshots, and commit
    3. After subagent completes, confirm feature_list.json was updated
    4. CONTINUE to next feature — do NOT stop
END WHILE
```

### Launching Feature Subagents (Claude Code)

For each feature, use the **Agent tool** to launch a subagent. This keeps each feature's work isolated and prevents context window overflow.

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
2. Implement the feature following existing code patterns
3. Make sure the implementation is complete and production-quality

### Phase 2: Refactor & Unit Test
4. Review the code you just wrote and any code you touched. Actively refactor for:
   - **Testability** — Extract pure functions and logic out of UI components and handlers.
     Move business logic, validation, data transformation, and state calculations into
     separate utility/service modules that can be unit tested without DOM or network.
   - **Reusability** — If you see duplicated logic (in your code or existing code you touched),
     extract shared helpers. Don't duplicate what already exists elsewhere in the codebase.
   - **Maintainability** — Keep functions small and single-purpose. Name things clearly.
     Split large files. Prefer composition over deep nesting.
5. Write unit tests for all extracted logic — pure functions, validators, transformers,
   state calculations, business rules. Use the project's existing test framework.
   Run them: npm test (or the project's unit test command) — fix until green.
6. Do NOT unit test UI rendering or things that are better covered by E2E tests.
   Unit tests are for logic; E2E tests are for behavior.

### Phase 3: E2E Test & Verify
7. Write or update E2E tests in the Playwright test files
8. Follow the screenshot naming convention: {scope}-feature-{id}-step{N}-{description}.png
9. Snapshot existing screenshots before running tests:
   find e2e/screenshots -name "*.png" -type f 2>/dev/null | sort > /tmp/screenshots-before.txt
10. Run the feature's E2E tests (not the full suite):
    npx playwright test --grep "feature-{id}"  # or the relevant test file
11. If tests fail: read the errors, fix, and re-run until they pass
12. Find new/changed screenshots:
    find e2e/screenshots -name "*.png" -type f 2>/dev/null | sort > /tmp/screenshots-after.txt
    comm -13 /tmp/screenshots-before.txt /tmp/screenshots-after.txt
    find e2e/screenshots -name "*.png" -newer /tmp/screenshots-before.txt -type f 2>/dev/null | sort
13. Visually review ONLY the new/changed screenshots using the Read tool. Evaluate for:
    - Layout — Content fits? No overflow or clipping?
    - Spacing — Appropriate padding/margins?
    - Touch targets — Buttons/inputs at least 44px?
    - Visual hierarchy — Important actions obvious?
    - Error states — Messages visible and red?
    - Typography — Text readable, proper sizes?
    - Consistency — Similar screens use same patterns?
14. If screenshot issues found: fix (minimal CSS changes, keep data-testid), re-run tests, review again

### Phase 4: Commit
15. Update feature_list.json — change "passes": false to "passes": true for this feature
16. Update progress.txt with what was done and current feature pass count
17. Commit all changes:
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

**How to launch the subagent:**

Use the Agent tool with `subagent_type: "general-purpose"`. Example:

```
Agent tool call:
  description: "Implement feature #3"
  prompt: [filled template above]
```

### After Each Subagent Completes

The subagent handles implementation, testing, screenshot verification, and committing. The parent agent only needs to:

1. **Confirm** the feature was committed (`git log --oneline -1`)
2. **Confirm** `feature_list.json` was updated (`"passes": true`)
3. If the subagent failed to complete, launch another subagent to fix and finish
4. **Loop back** — pick the next incomplete feature and repeat

### Decision Making Guidelines

Since the human may be asleep, follow these rules for autonomous decisions:

| Situation | Decision |
|-----------|----------|
| Ambiguous spec | Choose the simplest reasonable interpretation |
| Multiple implementation approaches | Pick the one matching existing patterns |
| Test is flaky | Add proper waits/retries, don't skip the test |
| Feature seems too large | Break into sub-tasks within the subagent |
| Dependency conflict | Use the version compatible with existing packages |
| Build error | Read the error, fix it, rebuild |
| Port conflict | Kill the conflicting process and restart |
| Database issue | Reset/reseed the database |
| Feature blocked by another | Skip to next feature, come back later |
| Unclear UI design | Follow existing UI patterns in the app |

### Session End

Only end the session when:
- **ALL features have `"passes": true`**, OR
- A truly unrecoverable error occurs (hardware failure, missing credentials, etc.)

Before ending:
1. Run all unit tests
2. Run E2E tests only for features that were completed in previous sessions (regression check)
3. Ensure clean git status (`git status` shows clean working tree)
4. Update `progress.txt` with final summary
5. Commit any remaining changes

---

## Playwright Configuration

Optimize for AI agent consumption:

```typescript
export default defineConfig({
  timeout: 10000,           // 10s max per test
  expect: { timeout: 3000 },
  reporter: [
    ['list'],
    ['json', { outputFile: 'e2e/test-results/results.json' }],
  ],
  use: {
    actionTimeout: 5000,
    navigationTimeout: 10000,
    screenshot: 'on',       // Keep ALL screenshots
    trace: 'retain-on-failure',
  },
});
```

### Screenshot Naming Convention

Format: `{scope}-feature-{id}-step{N}-{description}.png`

Examples:
- `auth-feature-17-step3-modal-open.png`
- `core-feature-7-step6-project-in-list.png`
- `video-editor-feature-15-complete-flow.png`

---

## Critical Rules

### Feature List Rules
- NEVER remove or edit feature descriptions or test steps
- NEVER weaken or delete tests
- ONLY change `"passes": false` to `"passes": true` after verification
- Work on features in priority order (high -> medium -> low)

### Code Quality
- Write production-ready code, not prototypes
- Follow existing code patterns
- Keep functions focused and files well-organized
- Refactor files exceeding 300 lines
- Extract business logic, validation, and data transformations into pure, testable modules
- Write unit tests for logic; use E2E tests for UI behavior
- Do not duplicate logic — reuse or extract shared helpers

### Session Handoff
- All work committed before ending session
- `progress.txt` updated with session summary
- No debug code or console.logs left in
- Codebase in clean, working state

### Autonomous Operation
- NEVER stop to ask the human a question
- NEVER wait for human approval
- Make reasonable decisions based on existing patterns
- If blocked, try alternative approaches before giving up
- Keep working until ALL features are complete

---

## Troubleshooting

### Tests Timeout
- Check backend is responding
- Look for infinite loading states
- Increase timeout if genuinely needed

### Flaky Tests
- Use `await expect()` instead of raw assertions
- Wait for network idle: `await page.waitForLoadState('networkidle')`

### Screenshots Blank
- Ensure page fully loaded before screenshot
- Check viewport size
- Verify correct URL navigation

---

## Reference Files

For detailed templates and examples, see:
- `references/feature-list-format.md` — Feature list JSON structure
- `references/init-script-template.md` — init.sh template
- `references/continue-workflow.md` — Full continue workflow details
- `references/e2e-verification.md` — E2E screenshot evaluation criteria and Playwright setup
