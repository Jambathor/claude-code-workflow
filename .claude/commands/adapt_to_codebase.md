---
description: Scan a repo, detect the stack, and write a project-specific CLAUDE.md with conventions and commands.
model: opus
---

# Adapt to Codebase

You are onboarding to a new (or existing) codebase. Your job is to understand the stack and produce a lean, accurate project CLAUDE.md.

## Process

### Step 1: Detect Stack

Look for these files in the repo root:
- `package.json` → Node/TypeScript project (check for framework: next, express, etc.)
- `pyproject.toml` / `requirements.txt` / `setup.py` → Python project
- `Cargo.toml` → Rust project
- `go.mod` → Go project
- `Makefile` → check what commands are available
- `docker-compose.yml` / `Dockerfile` → containerized
- `.github/workflows/` → CI/CD setup
- `tsconfig.json` → TypeScript
- `jest.config.*` / `vitest.config.*` / `pytest.ini` → test runner
- `.eslintrc.*` / `.prettierrc.*` / `ruff.toml` → linter/formatter

### Step 2: Understand Conventions

Spawn parallel sub-agents:

**codebase_locator:** "Map the directory structure. Identify: source code, tests, config, docs, scripts. Report the top-level organization."

**codebase_pattern_finder:** "Find 2-3 examples of: how tests are written, how modules/components are structured, how errors are handled. Show actual code patterns."

### Step 3: Identify Commands

From package.json scripts, Makefile targets, or CI config, extract:
- How to run tests
- How to lint/format
- How to build
- How to start dev server
- Any other common commands

### Step 4: Write CLAUDE.md

Write to the project root `CLAUDE.md`:

```markdown
# {Project Name}

## What
{One sentence — what this project is}

## Why
{One sentence — who uses it and what problem it solves}

## Stack
{language}, {framework}, {key libraries}
Package manager: {npm/pnpm/bun/uv/cargo}

## Structure
- `{dir}/` — {purpose}
- `{dir}/` — {purpose}

## Commands
- Test: `{exact command}`
- Lint: `{exact command}`
- Format: `{exact command}`
- Build: `{exact command}`
- Dev: `{exact command}`

## Conventions
- {pattern from actual code — file:line ref}
- {pattern from actual code — file:line ref}
- {pattern from actual code — file:line ref}

## File Boundaries
- Safe to edit: `src/`, `tests/`, `docs/`
- Never touch: `.git/`, `node_modules/`, `dist/`

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

### Step 5: Report

Tell the user:
- What stack was detected
- What conventions were observed
- Where CLAUDE.md was written
- Suggest running `/setup_hooks` next (when available) to wire up automated checks

## Critical Rules

- Conventions must come from ACTUAL CODE, not generic best practices. Reference file:line.
- Commands must be VERIFIED to work. Run them and confirm.
- Keep CLAUDE.md LEAN. Every line must earn its place.
- If an existing CLAUDE.md exists, read it first and merge — don't overwrite user customizations.
