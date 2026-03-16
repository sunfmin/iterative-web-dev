# iterative-web-dev

A AI skill for iterative web development with AI agents.

## Installation

```bash
npx skills add https://github.com/sunfmin/iterative-web-dev
```

## Overview

This skill provides a complete workflow for AI agents working on long-running web development projects across multiple sessions. It ensures **incremental, reliable progress** with proper handoffs between sessions.

## Core Principles

1. **Incremental progress** — Work on ONE feature at a time. Finish, test, and commit before moving on.
2. **Feature list is sacred** — `feature_list.json` is the single source of truth.
3. **Git discipline** — Commit after every completed feature.
4. **Clean handoffs** — Every session ends with committed work and updated progress notes.
5. **Test before build** — Verify existing features work before implementing new ones.

## Workflows

| Workflow | Use When |
|----------|----------|
| **init-scope** | Starting a new scope, switching scopes, or setting up project structure |
| **continue** | Every session after init — picking up work and implementing features |
| **e2e-screenshot-verification** | Verifying features visually, reviewing UI, catching UX issues |

## Key Files

- `spec.md` — Project specification (symlink to active scope)
- `feature_list.json` — Feature tracking with pass/fail status
- `progress.txt` — Session progress log
- `init.sh` — Development environment setup script

## License

MIT
