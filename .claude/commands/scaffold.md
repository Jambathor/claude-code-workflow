---
description: Initialize a new project from scratch — creates repo structure, installs deps, configures CLAUDE.md and hooks. Works for non-technical users.
model: opus
---

# Scaffold

You are setting up a brand new project from scratch. Your job is to get the user from zero to a working, configured project — handling every step including the ones a non-technical person wouldn't know to do.

## Input

$ARGUMENTS should describe what they want to build. Can be vague ("a web app for tracking habits") or specific ("a Next.js app with Supabase and Tailwind").

If no arguments: ask "What do you want to build? Describe it in plain language."

## Process

### Phase 1: Understand What They Want

Ask the user up to 3 questions max. Focus on decisions that change the project structure:

1. **What is this?** (if not clear from input) — "Is this a web app, an API, a CLI tool, a Chrome extension, a game, something else?"
2. **Any tech preferences?** — "Do you have a preferred language or framework, or should I choose based on what fits?"
3. **How should it be hosted?** — "Local only, or will this be deployed somewhere?"

If the user says "you choose" or doesn't know — make opinionated choices. Document why in CLAUDE.md. Don't ask follow-ups about things you can decide.

**Default stack choices when user has no preference:**
- Web app → Next.js + TypeScript + Tailwind
- API → Node.js + TypeScript + Hono
- CLI tool → Node.js + TypeScript
- Python project → Python + uv + ruff
- Game → depends on scope (Phaser for 2D web, Three.js for 3D web)
- Chrome extension → TypeScript + Vite
- Unsure → ask one more question, then decide

### Phase 2: Create Project Structure

```bash
# 1. Init repo
git init {project-name}
cd {project-name}

# 2. Init package manager / language tooling
# (examples — adapt to detected stack)
# Node: npm init -y / pnpm init / bun init
# Python: uv init / poetry init
# Rust: cargo init

# 3. Install core deps
# Be specific — install what the spec needs, not a kitchen sink

# 4. Create directory structure
mkdir -p src tests docs docs/plans docs/research docs/handoffs docs/logs/errors docs/logs/successes .claude/hooks

# 5. Create minimal starter files
# - src/index.ts (or equivalent) with hello world
# - tests/ with one example test that passes
# - .gitignore appropriate for the stack
```

**Critical: verify each step works before moving on.**
- After installing deps: confirm no errors
- After creating files: confirm they exist
- After writing a test: run it and confirm it passes

### Phase 3: Configure CLAUDE.md

Write `CLAUDE.md` to the project root using this structure:

```markdown
# {Project Name}

## What
{One sentence — what this project is}

## Why
{One sentence — what problem it solves}

## Stack
{language}, {framework}, {key libraries}
Package manager: {npm/pnpm/bun/uv/cargo}

## Structure
- `src/` — source code
- `tests/` — test files
- `docs/` — plans, research, handoffs, learnings, logs

## Commands
- Test: `{exact command}`
- Lint: `{exact command}`
- Format: `{exact command}`
- Build: `{exact command}`
- Dev: `{exact command}`

## Conventions
- {convention derived from chosen stack — e.g., "tests colocated: foo.ts → foo.test.ts"}
- {convention from framework — e.g., "app router: pages in src/app/"}

## Subagents
Always launch opus subagents unless specified otherwise.

## Workflow
- /draft for complex features → /specify → /clarify → /clear → /create_plan → /clear → /implement_plan
- /specify for clear features → /clarify → /clear → /create_plan → /clear → /implement_plan
- /validate_plan to check for drift
- /create_handoff before stopping, /resume_handoff to continue
- /log_error when things go wrong, /log_success when things go right

## Rules
- When running workflow commands (/specify, /create_plan, /implement_plan, etc.), follow ONLY the command instructions. Do not invoke additional skills or override the command's process.
- Always /clear between spec→plan and plan→implement phases. The files on disk are the handoffs.
```

**Verify every command listed actually works.** Run them. If any fail, fix the config or remove the command.

### Phase 4: Setup Hooks

Run the equivalent of `/setup_hooks` logic:
1. Detect formatter, linter, test runner from what was just installed
2. Create `.claude/hooks/` scripts (format, block_dangerous, test_related, checkpoint)
3. Write `.claude/settings.json` with hook configuration

### Phase 5: Initial Commit

```bash
git add -A
git commit -m "Initial project setup: {stack description}"
```

### Phase 6: Report

Tell the user:

```
Project ready: {project-name}/

Stack: {language} + {framework}
Package manager: {manager}
Test runner: {runner}
Formatter: {formatter}

What's set up:
- ✓ Project structure with src/, tests/, docs/
- ✓ Dependencies installed
- ✓ One passing test
- ✓ CLAUDE.md configured
- ✓ Hooks configured (format, block dangerous, async tests, git checkpoint)
- ✓ Initial commit

Next steps:
1. Write a spec: describe what you want to build
2. Run /specify to generate a structured spec
3. Run /clarify to resolve ambiguities
4. Run /create_plan to plan implementation
5. Run /implement_plan to build it

Or just tell me what feature you want to start with.
```

## Critical Rules

- **Verify everything.** Don't tell the user "tests are set up" unless you ran the test and it passed.
- **Be opinionated.** Non-technical users don't want 5 options for a test runner. Pick one and move on.
- **Document choices.** Every "I picked X over Y" goes in CLAUDE.md conventions section with a one-line reason.
- **Start small.** One passing test, one source file, working commands. The user adds complexity through features, not through initial scaffolding.
- **Don't over-install.** Only install what the project needs NOW. More deps = more things that break.
