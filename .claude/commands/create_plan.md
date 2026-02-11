---
description: Create a detailed implementation plan through interactive research and iteration.
model: opus
---

# Create Plan

You are creating an implementation plan. Your goal is a document specific enough that any developer (or agent) could execute it without asking questions.

## Input

$ARGUMENTS should describe the feature/task, or point to a spec file. If a spec file exists, read it FULLY first.

## Process

### Phase 1: Context Gathering

1. **Read all mentioned files completely** — use Read without limit/offset.
2. **Spawn parallel research sub-agents:**
   - `codebase_locator` — find files related to this feature area
   - `codebase_analyzer` — understand current implementation in relevant areas
   - `codebase_pattern_finder` — find similar features already built (to match patterns)
3. **Read all files identified by research** into main context.
4. **Present your understanding** of the current state to the user with file:line references.
5. **Ask focused questions** about anything unclear. Do not guess.

### Phase 2: Research and Discovery

1. If the user corrects your understanding, spawn NEW research tasks to verify — do not just accept corrections blindly.
2. Spawn additional targeted research as needed.
3. Wait for ALL sub-tasks to complete.
4. Present findings and design options (if multiple valid approaches exist).

### Phase 3: Plan Writing

Write the plan to `docs/plans/YYYY-MM-DD-{description}.md`.

Use this structure:

```markdown
# Plan: {Feature Name}
**Date:** {date}
**Status:** Draft

## Overview
{What we're building and why, in 2-3 sentences}

## Current State
{What exists today, with file:line references}

## What We're NOT Doing
{Explicit scope boundaries — critical for preventing drift}

## Implementation

### Phase 1: {Name}

#### Changes Required

**File:** `{exact/path/to/file.ext}`
**What:** {description of changes}
```{language}
// Specific code to add/modify — actual code, not pseudocode
```

**File:** `{exact/path/to/another/file.ext}`
**What:** {description}

#### Success Criteria

**Automated (run by agent):**
- [ ] `{exact command}` passes
- [ ] `{exact command}` passes

**Manual (verified by human):**
- [ ] {description of what to check}

#### Implementation Note
Complete this phase and verify ALL automated criteria before proceeding.
If automated checks fail, spawn a new sub-agent to diagnose — do NOT debug in main context.

---

### Phase 2: {Name}
{same structure}

## Open Questions
{anything still unresolved — ideally empty}
```

### Phase 4: Review

Present the plan location to the user. Iterate based on feedback.

## Critical Rules

- **Exact file paths** in every change. Never "update the auth module" — always `src/middleware/auth.ts`.
- **Actual code** in changes, not pseudocode or descriptions. Show what to write.
- **Success criteria split** into automated (commands) and manual (human checks). Always.
- **Scope boundaries** explicitly stated. "What we're NOT doing" prevents drift.
- **One phase completes fully before the next starts.** No parallel phases.
- **Phase success criteria must be verified before proceeding.** This is a gate, not a suggestion.
- If the user has a spec file, read it. If it has `[NEEDS CLARIFICATION]` markers, suggest running `/clarify` first.
