# iterative-web-dev

An AI skill for iterative web development with AI agents. Supports **Claude Code** (with subagents) and **Windsurf**.

## Installation

```bash
npx skills add https://github.com/sunfmin/iterative-web-dev
```

## Overview

This skill provides a complete workflow for AI agents working on long-running web development projects across multiple sessions. It ensures **incremental, reliable progress** with proper handoffs between sessions.

### Claude Code Features

- **Subagent per feature** — Each feature is implemented in its own subagent using the Agent tool, keeping context clean and isolated
- **Autonomous loop** — The agent keeps working through ALL features without stopping, even if the human is away
- **Self-directed decisions** — Handles ambiguity, errors, and blockers autonomously using decision-making guidelines
- **Commit after each feature** — Every completed feature is committed independently for clean git history

## Core Principles

1. **Incremental progress** — Work on ONE feature at a time. Finish, test, and commit before moving on.
2. **Feature list is sacred** — `feature_list.json` is the single source of truth.
3. **Git discipline** — Commit after every completed feature.
4. **Clean handoffs** — Every session ends with committed work and updated progress notes.
5. **Test before build** — Verify existing features work before implementing new ones.
6. **Autonomous execution** — Make all decisions yourself. Never stop to ask the human.
7. **Subagent isolation** — Each feature runs in its own subagent for clean context.

## Workflows

| Workflow | Use When |
|----------|----------|
| **init-scope** | Starting a new scope, switching scopes, or setting up project structure |
| **continue** | Every session after init — autonomously implements ALL remaining features |
| **e2e-screenshot-verification** | Verifying features visually, reviewing UI, catching UX issues |

## Key Files

- `spec.md` — Project specification (symlink to active scope)
- `feature_list.json` — Feature tracking with pass/fail status
- `progress.txt` — Session progress log
- `init.sh` — Development environment setup script

## How It Works (Claude Code)

1. Agent reads `feature_list.json` to find incomplete features
2. For each feature, launches a **subagent** (via Agent tool) with full context
3. Subagent implements the feature and writes E2E tests
4. Parent agent runs tests, verifies, commits
5. **Loops back** to pick the next feature
6. Only stops when ALL features have `"passes": true`

## License

MIT
