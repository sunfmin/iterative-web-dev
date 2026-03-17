# Feature List Format

The `feature_list.json` file is the single source of truth for project progress.

## Structure

```json
[
  {
    "id": 1,
    "category": "functional",
    "priority": "high",
    "description": "Brief description of the feature",
    "steps": [
      "Step 1: Navigate to relevant page",
      "Step 2: Perform action",
      "Step 3: Verify expected result"
    ],
    "passes": false
  }
]
```

## Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `id` | number | Unique numeric identifier within scope |
| `category` | string | Either "functional" or "style" |
| `priority` | string | "high", "medium", or "low" |
| `description` | string | Brief description of the feature |
| `steps` | array | Test steps to verify the feature |
| `passes` | boolean | Whether the feature passes all tests |

## Requirements

- Cover every feature in the scope's spec
- Use "functional" and "style" categories
- ALL features start with `"passes": false`
- Each feature has a unique numeric `id` (unique within scope)

## Critical Rules

**NEVER:**
- Remove or edit feature descriptions
- Remove or edit test steps
- Weaken or delete tests
- Change a passing feature back to failing (unless genuine regression)

**ONLY:**
- Change `"passes": false` to `"passes": true` after thorough verification

## Priority Order

Work on features in this order:
1. **high** priority first
2. **medium** priority second
3. **low** priority last
4. Within same priority, work in order they appear in the file

## Best Practices for Test Steps

### Include UX Verification Steps

Every feature's test steps should include at least one UX quality verification step. This ensures E2E tests enforce visual quality, not just functionality:

- "Step N: Verify loading skeleton appears while data loads"
- "Step N: Verify empty state shows icon, message, and CTA when no items exist"
- "Step N: Verify the page renders correctly at mobile width (375px)"
- "Step N: Verify hover effect on interactive elements"
- "Step N: Verify error state is styled with red text and borders"

### Include Screenshot Steps

Every feature should have steps that naturally produce screenshots at key states:

- Initial page state (before any user action)
- After user action (form filled, item created, filter applied)
- Error states (validation errors, failed operations)
- Empty states (no data scenarios)

## Example

```json
[
  {
    "id": 1,
    "category": "functional",
    "priority": "high",
    "description": "User can register with email and password",
    "steps": [
      "Step 1: Navigate to /register",
      "Step 2: Verify registration form loads with proper layout and field grouping",
      "Step 3: Submit empty form and verify inline validation errors (red text, red borders)",
      "Step 4: Fill in email and password fields",
      "Step 5: Click Register button and verify loading state on button",
      "Step 6: Verify redirect to dashboard",
      "Step 7: Verify welcome message shows username"
    ],
    "passes": false
  },
  {
    "id": 2,
    "category": "functional",
    "priority": "high",
    "description": "User can login with existing credentials",
    "steps": [
      "Step 1: Navigate to /login",
      "Step 2: Enter valid email and password",
      "Step 3: Click Login button",
      "Step 4: Verify redirect to dashboard",
      "Step 5: Verify dashboard shows loading skeleton then populated data"
    ],
    "passes": false
  },
  {
    "id": 3,
    "category": "style",
    "priority": "medium",
    "description": "Login form has proper validation styling and empty state",
    "steps": [
      "Step 1: Navigate to /login",
      "Step 2: Submit empty form",
      "Step 3: Verify error messages are red with field borders highlighted",
      "Step 4: Verify error messages appear below each input",
      "Step 5: Fill in fields and verify errors disappear",
      "Step 6: Verify form looks correct at mobile width (375px)"
    ],
    "passes": false
  }
]
```
