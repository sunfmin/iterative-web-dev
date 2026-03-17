# Session Handoff Standards

Before ending any session, the codebase must meet these standards. These are auditable — a verification subagent can check every item.

## Clean Codebase

### No Debug Code
- No `console.log` statements left in source code (test files excluded)
- No `console.debug`, `console.warn`, `console.error` used for debugging (legitimate error handling excluded)
- No `debugger` statements
- No commented-out code blocks (small inline comments explaining "why" are fine)
- No `TODO` or `FIXME` comments without a corresponding feature list item

### No Temporary Files
- No `.tmp`, `.bak`, or `.orig` files
- No editor swap files (`.swp`, `.swo`)
- No test output files left in source directories

## Git State

### Clean Working Tree
- `git status` shows clean working tree (no untracked, modified, or staged files)
- All work committed with descriptive commit messages
- No merge conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) in any file

### Gitignore Compliance
- All patterns from `references/gitignore-standards.md` present in `.gitignore`
- No build artifacts, dependencies, secrets, or generated files tracked

## Progress Tracking

### progress.txt Updated
- Contains summary of what was done this session
- Lists features completed (IDs and descriptions)
- Shows current pass count (e.g., "12/20 features passing")
- Notes any issues encountered or features skipped

### feature_list.json Accurate
- Every completed feature has `"passes": true`
- No feature marked passing that wasn't actually verified
- No features removed or descriptions edited

## Verification Commands

An audit subagent can verify these standards with:

```bash
# Clean working tree
git status --porcelain | wc -l  # Should be 0

# No console.log in source (exclude test files, node_modules)
grep -r "console\.\(log\|debug\)" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" --exclude-dir=node_modules --exclude-dir=test --exclude-dir=e2e --exclude-dir=__tests__ -l

# No debugger statements
grep -r "debugger" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" --exclude-dir=node_modules -l

# No merge conflict markers
grep -r "^<<<<<<< \|^=======$\|^>>>>>>> " --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" --include="*.go" --include="*.py" -l

# No TODO without feature list item
grep -rn "TODO\|FIXME" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" --exclude-dir=node_modules -l

# progress.txt exists and was recently updated
ls -la progress.txt
tail -20 progress.txt
```
