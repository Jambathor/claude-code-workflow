---
description: Transform a natural language feature description into a structured, numbered specification with acceptance criteria.
model: opus
---

# Specify

You are creating a feature specification from a user's description. Your output is a structured spec that's clear enough to plan and implement from — focused on WHAT and WHY, never HOW.

## Input

$ARGUMENTS should describe the feature in natural language. Can be one sentence ("add user login") or a paragraph. If empty, ask: "What feature do you want to build?"

## Process

### Step 1: Understand Intent

Read the description. Identify:
- **Who** is this for? (user type, persona)
- **What** should they be able to do?
- **Why** does this matter? (the problem it solves)
- **What's out of scope?** (equally important)

### Step 2: Make Informed Guesses

For anything not specified, make a reasonable default choice and document it as an assumption. Do NOT ask the user about every detail.

**Good assumptions:**
- "Login" → email/password unless specified otherwise
- "Dashboard" → responsive, works on desktop and mobile
- "API" → RESTful with JSON unless GraphQL is mentioned
- "Database" → whatever the project already uses

Mark assumptions clearly: `**Assumption:** {X}. Change this if wrong.`

### Step 3: Write the Spec

Write to `docs/specs/NNNN-{feature-name}.md` where NNNN is the next available number (scan existing specs in `docs/specs/`).

```markdown
# Spec NNNN: {Feature Name}
**Date:** {date}
**Status:** Draft
**Branch:** feature/NNNN-{feature-name}

## Problem
{1-3 sentences: what problem does this solve, for whom, and why now}

## User Stories

### US-1: {Title} (P1)
**As a** {user type}
**I want to** {action}
**So that** {benefit}
**Independent test:** {how to verify this story works in isolation}

### US-2: {Title} (P1)
{same format}

### US-3: {Title} (P2)
{same format — P2 = nice to have for V1}

## Requirements

### Functional
- **FR-001:** {requirement}
- **FR-002:** {requirement}
- **FR-003:** {requirement}

### Non-Functional
- **NFR-001:** {performance, security, accessibility requirement if relevant}

## Acceptance Criteria

### AC-1: {Scenario Name}
**Given** {precondition}
**When** {action}
**Then** {expected result}

### AC-2: {Scenario Name}
{same format}

## Edge Cases
- {What happens when input is empty/null?}
- {What happens when the user is not authenticated?}
- {What happens under error conditions?}
- {What happens with concurrent access?}

## Assumptions
- {Assumption 1} — change if wrong
- {Assumption 2} — change if wrong

## Out of Scope
- {Explicit boundary 1}
- {Explicit boundary 2}

## Success Criteria
- {Measurable outcome 1 — no implementation details}
- {Measurable outcome 2}

## Open Questions
- [NEEDS CLARIFICATION: {specific question}]
```

### Step 4: Validate the Spec

Self-check:
1. Can each user story be tested independently?
2. Are acceptance criteria specific enough to write a test from?
3. Are there more than 3 `[NEEDS CLARIFICATION]` markers? If yes, resolve the lowest-impact ones yourself with assumptions.
4. Is "Out of Scope" explicit? (This prevents scope creep later)
5. Does every FR have at least one AC that verifies it?

Fix any issues found.

### Step 5: Create Branch (Optional)

If the project uses git:
```bash
git checkout -b feature/NNNN-{feature-name}
```

### Step 6: Present to User

```
Spec created: docs/specs/NNNN-{feature-name}.md

{number} user stories ({P1 count} required, {P2 count} nice-to-have)
{number} functional requirements
{number} acceptance criteria
{number} assumptions (review these — change if wrong)
{number} items needing clarification

Next:
- Review the spec, especially the Assumptions section
- Run /clarify docs/specs/NNNN-{feature-name}.md to resolve open questions
- Run /create_plan docs/specs/NNNN-{feature-name}.md when ready to implement
```

## Spec Quality Rules

- **WHAT and WHY only.** Never mention frameworks, libraries, database schemas, or API designs. That's the plan's job.
- **Independently testable stories.** Each user story must work as a standalone demo. If it can't be tested alone, break it down further.
- **Max 3 clarification markers.** If you have more than 3, you're being too cautious. Make assumptions and document them.
- **Edge cases are mandatory.** Every spec must consider: empty input, unauthorized access, error conditions, concurrent access. Even if "N/A", say so.
- **Measurable success criteria.** "Users can log in" is weak. "Login completes in under 2 seconds and returns a valid session token" is strong.
- **Numbered requirements.** FR-001, FR-002, etc. This makes them referenceable in plans and tests.
- **P1/P2 priority.** P1 = must have for V1. P2 = nice to have for V1, required for V2. Helps /create_plan scope the work.
