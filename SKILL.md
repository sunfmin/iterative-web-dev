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
9. **Visual quality is non-negotiable** — Every feature MUST be verified via screenshots. A feature that works but looks bad is NOT complete. Screenshots are the primary evidence of correct implementation.
10. **Design with intention** — Follow the `/frontend-design` skill principles: commit to a bold aesthetic direction, avoid generic AI aesthetics, and create interfaces that are distinctive and production-grade. Every UI decision should be intentional, not default.

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

4. **Create feature list** — choose the right method:

   **If scope references a constitution / standards document** (e.g., "align with AGENTS.md", "refactor to follow standards"):
   Use the **Constitution Audit Workflow** from `references/constitution-audit.md`. This is a multi-subagent process:
   - Split the reference document into sections (~200 lines each)
   - Launch parallel subagents to extract EVERY requirement from each section (read actual text, not summaries)
   - Launch parallel subagents to verify each requirement against the actual codebase
   - Generate features ONLY from verified violations
   - This is NON-NEGOTIABLE for compliance scopes — ad-hoc auditing misses requirements

   **If scope is new feature development** (e.g., "build a PIM system", "add auth"):
   Use the standard process from `references/feature-list-format.md`

5. **Create/update init.sh** — see `references/init-script-template.md`

6. **Commit and update progress log**

---

## Workflow: Continue Session (Autonomous Feature Loop)

This is the main workflow. It runs ALL remaining features to completion without stopping.

**⚠️ CRITICAL NON-STOP RULE (NON-NEGOTIABLE) ⚠️**

**You MUST keep looping until EVERY feature in `feature_list.json` has `"passes": true`. Do NOT stop after one feature. Do NOT stop after two features. Do NOT stop to report progress to the user. Do NOT ask the human what to do next. The human may be asleep.**

**After EACH subagent completes, you MUST immediately launch the NEXT subagent for the next incomplete feature. The ONLY acceptable reasons to stop are:**
1. **ALL features have `"passes": true`**
2. **A truly unrecoverable error** (hardware failure, missing credentials that cannot be worked around)

**Stopping to "report back" or "check in" with the user is a VIOLATION of this workflow. The user explicitly chose autonomous execution. KEEP GOING.**

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
    3. After subagent completes, VERIFY screenshots and quality (see below)
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
2. Read the spec.md file for full project context
3. Implement the feature following existing code patterns
4. Make sure the implementation is complete and production-quality

### Phase 2: UX Quality Checklist

Before moving on, verify your implementation meets these UX standards.
See references/ux-standards.md for full details.

**Required for ALL features:**
- [ ] Loading states — Show skeleton/spinner while data loads (server components can use Suspense)
- [ ] Empty states — Show icon + helpful message + CTA when no data exists
- [ ] Error states — Show clear error messages with recovery actions
- [ ] Responsive — Verify layout works at 375px, 768px, 1280px widths
- [ ] Accessibility — aria-labels on interactive elements, keyboard navigable, focus rings visible

**Required for form features:**
- [ ] Field grouping — Related fields grouped with section headers/dividers
- [ ] Help text — Placeholder text or descriptions for non-obvious fields
- [ ] Validation feedback — Inline errors below fields with red borders, clearing on fix
- [ ] Submit feedback — Button shows loading state ("Saving..."), disables during submit

**Required for list/table features:**
- [ ] Column alignment — Numbers right-aligned, text left-aligned
- [ ] Hover states — Row highlighting on hover
- [ ] Empty search results — "No results for 'X'" with clear filters CTA
- [ ] Mobile layout — Table scrolls horizontally or collapses to cards on small screens

**Required for all UI work (follow /frontend-design principles):**
- [ ] Typography — Use distinctive, characterful fonts. Avoid generic defaults (Inter, Arial, system fonts)
- [ ] Color & theme — Commit to a cohesive aesthetic. Dominant colors with sharp accents
- [ ] Spatial composition — Intentional layout. Generous negative space OR controlled density
- [ ] Visual details — Atmosphere and depth. Subtle shadows, borders, or textures where appropriate
- [ ] Micro-interactions — Hover transitions (150-200ms), focus effects, button feedback

If any checkbox fails, fix it before proceeding.

### Phase 3: Refactor & Unit Test
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

### Phase 4: E2E Test & Visual Verification

**MANDATORY SCREENSHOT RULE: Every E2E test MUST take at least one fullPage screenshot.
Every screenshot MUST be visually reviewed using the Read tool.
If you skip screenshots or visual review, the feature is NOT complete.**

8. Write or update E2E tests in the Playwright test files. EVERY test must include at least one:
   ```typescript
   await page.screenshot({
     path: `e2e/screenshots/{scope}-feature-{id}-step{N}-{description}.png`,
     fullPage: true
   });
   ```
9. Snapshot existing screenshots before running tests:
   find e2e/screenshots -name "*.png" -type f 2>/dev/null | sort > /tmp/screenshots-before.txt
10. Run the feature's E2E tests (not the full suite):
    npx playwright test --grep "feature-{id}"  # or the relevant test file
11. If tests fail: read the errors, fix, and re-run until they pass
12. Find new/changed screenshots:
    find e2e/screenshots -name "*.png" -type f 2>/dev/null | sort > /tmp/screenshots-after.txt
    comm -13 /tmp/screenshots-before.txt /tmp/screenshots-after.txt
13. **MANDATORY VISUAL REVIEW** — Use the Read tool to open and inspect EVERY new screenshot.
    For each screenshot, evaluate and explicitly note in your work:

    ✓ or ✗ Layout — No overflow, clipping, or misalignment
    ✓ or ✗ Spacing — Consistent padding/margins, not cramped
    ✓ or ✗ Hierarchy — Primary actions obvious, text readable, proper font sizes
    ✓ or ✗ States — Loading/empty/error states present and styled
    ✓ or ✗ Aesthetics — Looks polished and intentional, not generic/prototype-level
    ✓ or ✗ Consistency — Matches existing UI patterns, colors, spacing scale

    If ANY item is ✗: fix the issue, re-run tests, review screenshots again.
    Do NOT proceed to Phase 6 until all items pass.

### Phase 5: Gitignore Review (before committing)
14. Review ALL files that would be staged by running:
    ```bash
    git status --short
    ```
    For every untracked or modified file, check if it should be in `.gitignore`. Common files that MUST be gitignored:
    - Build artifacts: `dist/`, `build/`, `.next/`, `out/`, `.output/`
    - Dependencies: `node_modules/`, `vendor/`, `.pnp.*`
    - Environment/secrets: `.env`, `.env.local`, `.env.*.local`, `*.pem`, `*.key`
    - IDE/editor: `.idea/`, `.vscode/`, `*.swp`, `*.swo`
    - OS files: `.DS_Store`, `Thumbs.db`, `desktop.ini`
    - Test artifacts: `test-results/`, `playwright-report/`, `coverage/`, `.nyc_output/`
    - Logs: `*.log`, `npm-debug.log*`, `yarn-debug.log*`
    - Cache: `.cache/`, `.parcel-cache/`, `.turbo/`, `.eslintcache`
    - Database files: `*.sqlite`, `*.db`
    - Generated files: `*.map` (source maps in production), `*.tsbuildinfo`

    If ANY file should be gitignored:
    a. Add the pattern to `.gitignore`
    b. If already tracked, remove from tracking: `git rm --cached <file>`
    c. Verify with `git status` that the file is now ignored

### Phase 6: Commit
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
- EVERY test must take screenshots — no exceptions
- EVERY screenshot must be visually reviewed with the Read tool — no exceptions
- BEFORE committing, review ALL files for .gitignore candidates — never commit build artifacts, secrets, or generated files
```

**How to launch the subagent:**

Use the Agent tool with `subagent_type: "general-purpose"`. Example:

```
Agent tool call:
  description: "Implement feature #3"
  prompt: [filled template above]
```

### After Each Subagent Completes

The subagent handles implementation, testing, screenshot verification, and committing. The parent agent MUST verify:

1. **Confirm commit** — `git log --oneline -1`
2. **Confirm feature_list.json** — feature has `"passes": true`
3. **VERIFY SCREENSHOTS EXIST** for this feature:
   ```bash
   ls e2e/screenshots/{scope}-feature-{id}-*.png 2>/dev/null | wc -l
   ```
   If count is 0, the subagent skipped screenshots. Launch a follow-up subagent to add screenshots and visual review.
4. **SPOT-CHECK one screenshot** — Use the Read tool to open one screenshot from this feature. Verify the UI looks polished and production-quality, not prototype-level. Check for:
   - Professional visual design (not bare HTML or unstyled elements)
   - Proper spacing and alignment
   - Loading/empty states present
   - Consistent with other pages in the app
5. If quality is poor, launch a **polish subagent** to fix visual issues before moving on.
6. If the subagent failed to complete, launch another subagent to fix and finish.
7. **Loop back IMMEDIATELY** — pick the next incomplete feature and launch a new subagent RIGHT NOW. Do NOT stop, do NOT report to the user, do NOT wait for instructions. KEEP GOING until ALL features pass.

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
| Unclear UI design | Follow /frontend-design principles: bold aesthetic, intentional choices |
| UI looks generic/plain | Add visual polish: shadows, transitions, better typography, spacing |

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

### Visual Quality (enforced by /frontend-design principles)
- Every page must have loading states, empty states, and error states
- Every interactive element must have hover/focus transitions
- Typography must be intentional — distinctive fonts, clear hierarchy
- Color palette must be cohesive — dominant colors with sharp accents
- Spacing must be consistent — follow a scale (4/8/12/16/24/32/48px)
- No generic AI aesthetics — no bare unstyled HTML, no default system fonts

### Screenshot Verification (MANDATORY)
- Every E2E test MUST take at least one fullPage screenshot
- Every screenshot MUST be visually reviewed with the Read tool
- Parent agent MUST verify screenshot count > 0 after each subagent
- Parent agent MUST spot-check at least one screenshot per feature
- If screenshots are missing or UI is poor quality, a follow-up subagent is required

### Session Handoff
- All work committed before ending session
- `progress.txt` updated with session summary
- No debug code or console.logs left in
- Codebase in clean, working state

### Autonomous Operation (NON-NEGOTIABLE)
- NEVER stop to ask the human a question
- NEVER wait for human approval
- NEVER stop to "report progress" or "check in" — the user can see commits in git log
- NEVER output a summary and wait — immediately launch the next subagent
- After each subagent completes: verify → launch next subagent. That's it. No pausing.
- Make reasonable decisions based on existing patterns
- If blocked, try alternative approaches before giving up
- Keep working until ALL features are complete
- The continue workflow is a LOOP, not a single step. You are the loop controller.

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

### UI Looks Generic/Plain
- Review /frontend-design principles and references/ux-standards.md
- Add distinctive typography (not Inter/Arial/system fonts)
- Add visual depth: shadows, borders, gradients
- Add micro-interactions: hover transitions, focus effects
- Ensure loading/empty/error states are polished, not bare text

---

## Reference Files

For detailed templates and examples, see:
- `references/feature-list-format.md` — Feature list JSON structure
- `references/init-script-template.md` — init.sh template
- `references/continue-workflow.md` — Full continue workflow details
- `references/e2e-verification.md` — E2E screenshot evaluation criteria and Playwright setup
- `references/ux-standards.md` — UX quality standards and checklist
- `references/frontend-design.md` — Design principles from /frontend-design skill
- `references/constitution-audit.md` — Systematic audit workflow for compliance/alignment scopes
