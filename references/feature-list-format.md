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
      "Step 2: Fill in email and password fields",
      "Step 3: Click Register button",
      "Step 4: Verify redirect to dashboard",
      "Step 5: Verify welcome message shows username"
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
      "Step 4: Verify redirect to dashboard"
    ],
    "passes": false
  },
  {
    "id": 3,
    "category": "style",
    "priority": "medium",
    "description": "Login form has proper validation styling",
    "steps": [
      "Step 1: Navigate to /login",
      "Step 2: Submit empty form",
      "Step 3: Verify error messages are red",
      "Step 4: Verify error messages appear near inputs"
    ],
    "passes": false
  }
]
```
