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
11. **Standards are auditable** — Quality standards live in reference docs and are systematically verified, not just aspirational checklists.

## Standards Documents

All verifiable quality standards are extracted into reference docs. These are used both as guidance during implementation and as audit targets for systematic verification.

| Document | What it covers |
|----------|---------------|
| `references/ux-standards.md` | Loading/empty/error states, responsive design, accessibility, forms, tables, navigation |
| `references/frontend-design.md` | Typography, color, spatial composition, micro-interactions, anti-patterns |
| `references/code-quality.md` | File organization, testable architecture, unit testing, no duplication |
| `references/gitignore-standards.md` | Files that must never be committed |
| `references/e2e-verification.md` | Screenshot rules, visual review criteria, Playwright setup |

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
features_completed_this_session = 0

WHILE there are features with "passes": false in feature_list.json:
    1. Read feature_list.json to find next incomplete feature (highest priority first)
    2. Launch a SUBAGENT to implement, test, verify screenshots, and commit
    3. After subagent completes, VERIFY screenshots and quality (see below)
    4. features_completed_this_session++
    5. If features_completed_this_session % 5 == 0: run STANDARDS AUDIT (see below)
    6. CONTINUE to next feature — do NOT stop
END WHILE

Run FINAL STANDARDS AUDIT before ending session
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

## Standards Documents
Read these reference docs and follow them during implementation:
- references/code-quality.md — Code organization, testability, unit testing rules
- references/ux-standards.md — UX quality requirements (loading/empty/error states, responsive, accessibility)
- references/frontend-design.md — Visual design principles (typography, color, composition)
- references/gitignore-standards.md — Files that must never be committed
- references/e2e-verification.md — Screenshot and E2E testing rules

## Instructions

### Phase 1: Implement
1. Read the relevant source files to understand the current codebase
2. Read the spec.md file for full project context
3. Read the standards documents listed above
4. Implement the feature following existing code patterns and the standards
5. Make sure the implementation is complete and production-quality

### Phase 2: Refactor & Unit Test
6. Review the code you wrote. Refactor following references/code-quality.md:
   - Extract pure functions out of UI components and handlers
   - Move business logic into testable utility/service modules
   - Eliminate duplication — reuse existing helpers or extract new shared ones
   - Keep files under 300 lines
7. Write unit tests for all extracted logic. Run them until green.
8. Do NOT unit test UI rendering — that's what E2E tests are for.

### Phase 3: E2E Test & Visual Verification
Follow the full process in references/e2e-verification.md:
9. Write E2E tests with screenshots at key user journey points
10. Run the feature's E2E tests — fix until green
11. MANDATORY: Use the Read tool to visually review EVERY screenshot
    Evaluate against the criteria in references/e2e-verification.md (layout, spacing,
    hierarchy, states, aesthetics, consistency). Fix and re-run until all pass.

### Phase 4: Gitignore Review
Follow references/gitignore-standards.md:
12. Run `git status --short` and check every file against gitignore patterns
13. Add any missing patterns to `.gitignore`, remove from tracking if needed

### Phase 5: Commit
14. Update feature_list.json — change "passes": false to "passes": true
15. Update progress.txt with what was done and current feature pass count
16. Commit all changes:
    git add -A && git commit -m "feat: [description] — Implemented feature #[id]: [description]"

## Key Rules
- Follow existing code patterns and the standards documents
- Keep changes focused on this feature only
- Do not break other features
- Make all decisions yourself, never ask for human input
- EVERY test must take screenshots — no exceptions
- EVERY screenshot must be visually reviewed — no exceptions
- BEFORE committing, review ALL files for .gitignore candidates
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

### Periodic Standards Audit

**When to run:** Every 5 completed features AND at session end (before final commit).

This uses the same audit pattern as `references/constitution-audit.md`, but applied to the project's own standards documents. The audit catches issues that individual subagents missed — self-review has blind spots.

**Audit process:**

1. For EACH standards document (`ux-standards.md`, `frontend-design.md`, `code-quality.md`, `gitignore-standards.md`), launch a **verification subagent** that:
   - Reads the standards document
   - Reads the code/files changed since the last audit (use `git diff --name-only HEAD~5` or similar)
   - Checks each standard against the actual code
   - Reports: COMPLIANT or VIOLATION with specific file and line

2. Collect all violations across subagents

3. If violations found:
   - Group related violations into fix batches
   - Launch a **fix subagent** for each batch
   - Each fix subagent commits its changes
   - Re-verify the fixed code

4. Log audit results in `progress.txt`

**Subagent prompt template for standards audit:**

```
You are auditing recently changed code against a project standards document.

## Standards Document
{paste the full content of the standards doc}

## Files to Audit
{list of files changed since last audit}

## Instructions
1. Read each file listed above
2. For EACH standard in the document, check if the code complies
3. Report findings as:
   - COMPLIANT: {standard} — {brief evidence}
   - VIOLATION: {standard} — {file}:{line} — {what's wrong} — {fix needed}
4. Be thorough — check every standard, don't skip "obvious" ones
```

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
1. Run **final standards audit** (see Periodic Standards Audit above)
2. Run all unit tests
3. Run E2E tests only for features that were completed in previous sessions (regression check)
4. Ensure clean git status (`git status` shows clean working tree)
5. Update `progress.txt` with final summary
6. Commit any remaining changes

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

### Standards Enforcement
- All quality standards live in `references/` docs — subagents MUST read them
- Standards are verified both during implementation (by subagent) AND periodically (by audit)
- Audit violations MUST be fixed before session ends
- See: `references/code-quality.md`, `references/ux-standards.md`, `references/frontend-design.md`, `references/gitignore-standards.md`, `references/e2e-verification.md`

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
- Review references/frontend-design.md and references/ux-standards.md
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
- `references/code-quality.md` — Code organization, testability, and unit testing standards
- `references/gitignore-standards.md` — Gitignore patterns and review process
- `references/constitution-audit.md` — Systematic audit workflow for compliance/alignment scopes
