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

## When to Use Each Workflow

| Workflow | Use When |
|----------|----------|
| **init-scope** | Starting a new scope, switching scopes, or setting up project structure |
| **continue** | Every session after init — picking up work and implementing features |
| **e2e-screenshot-verification** | Verifying features visually, reviewing UI, catching UX issues |

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

## Workflow: Continue Session

Use this for every session after initialization. Picks up where the last session left off.

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

3. **Verify existing features** — Run E2E tests to check for regressions

4. **Pick next feature** — Highest priority with `"passes": false`

5. **Implement feature** — Clean, production-quality code

6. **Test with screenshots** — Use E2E verification workflow

7. **Update feature_list.json** — Only change `"passes": false` to `"passes": true`

8. **Update progress.txt**
   ```
   ## Session N — [DATE]
   ### What was done:
   - Implemented feature #X: [description]
   
   ### Current state:
   - Features passing: X / N
   - Next priority: Feature #Y
   ```

9. **Commit all progress**
   ```bash
   git add -A
   git commit -m "feat: [description]"
   ```

10. **Final verification** — All tests pass, clean working tree

---

## Workflow: E2E Screenshot Verification

Use this to verify features work correctly with visual inspection.

### Steps

1. **Ensure environment running**
   ```bash
   lsof -i :3000 | head -2  # Frontend
   lsof -i :8082 | head -2  # Backend
   ```

2. **Clear old screenshots**
   ```bash
   rm -rf e2e/screenshots/*.png 2>/dev/null || true
   rm -rf test-results/**/*.png 2>/dev/null || true
   ```

3. **Run E2E tests**
   ```bash
   npx playwright test
   ```

4. **List screenshots**
   ```bash
   find e2e/screenshots -name "*.png" -type f | sort
   ```

5. **Review each screenshot** — Use Read tool to visually inspect

6. **Evaluate for issues**:
   - **Layout** — Content fits? No overflow?
   - **Spacing** — Appropriate padding/margins?
   - **Touch targets** — Buttons at least 44px?
   - **Visual hierarchy** — Important actions obvious?
   - **Error states** — Messages visible and red?
   - **Typography** — Text readable?

7. **Fix issues found** — Minimal CSS changes, keep data-testid intact

8. **Re-run and verify** — Repeat until all issues resolved

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
- Work on features in priority order (high → medium → low)

### Code Quality
- Write production-ready code, not prototypes
- Follow existing code patterns
- Keep functions focused and files well-organized
- Refactor files exceeding 300 lines

### Session Handoff
- All work committed before ending session
- `progress.txt` updated with session summary
- No debug code or console.logs left in
- Codebase in clean, working state

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
- `references/e2e-verification.md` — Full E2E verification details
