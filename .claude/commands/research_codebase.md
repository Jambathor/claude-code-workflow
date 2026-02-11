---
description: Research codebase comprehensively using parallel sub-agents. Returns file:line references.
model: opus
---

# Research Codebase

You are conducting deep codebase research. Your job is to DOCUMENT what exists, not suggest improvements.

## Input

The user's research query is: $ARGUMENTS

If no arguments provided, ask what they want to understand about the codebase.

## Process

### Phase 1: Read Mentioned Files First

If the user mentions specific files or paths, read them FULLY before spawning any sub-agents:
- Use the Read tool WITHOUT limit/offset parameters
- Read these yourself in main context — do NOT delegate this to sub-agents

### Phase 2: Decompose and Research

Break the query into composable research areas. Create a todo list to track progress.

Spawn parallel sub-agent tasks using these agent types:

- **codebase_locator** — find WHERE relevant files live (file paths organized by purpose)
- **codebase_analyzer** — understand HOW specific code works (data flow, logic, patterns with file:line refs)
- **codebase_pattern_finder** — find EXAMPLES of similar implementations (actual code to model after)

Be EXTREMELY specific in sub-agent prompts:
- Name exact directories to search
- Specify what information to extract
- Request file:line references in responses
- Limit each agent to one focused area

Example of spawning:
```
Task 1 (codebase_locator): "Find all files related to authentication in src/. Categorize by: route handlers, middleware, models, tests, config."

Task 2 (codebase_analyzer): "Read src/middleware/auth.ts and trace the JWT verification flow. Document entry point, validation steps, error handling, and return types with file:line references."

Task 3 (codebase_pattern_finder): "Find 2-3 examples of how API endpoints handle authorization checks. Show the actual code patterns used."
```

### Phase 3: Wait and Synthesize

Wait for ALL sub-agents to complete before synthesizing.

If a sub-agent returns unexpected results, spawn a follow-up task to verify. Cross-check findings against actual code — don't accept results that seem wrong.

### Phase 4: Produce Research Document

Write findings to `docs/research/YYYY-MM-DD-{description}.md` in the project root.

Structure:
```markdown
# Research: {Topic}
**Date:** {date}
**Query:** {original query}

## Summary
{2-3 sentence overview}

## Findings

### {Area 1}
{findings with file:line references}

### {Area 2}
{findings with file:line references}

## Code References
{consolidated list of all file:line references}

## Open Questions
{anything unclear or needing further investigation}
```

## Critical Rules

- All agents are DOCUMENTARIANS, not critics. Do not suggest improvements, critique quality, or propose enhancements unless explicitly asked.
- Every finding must include file:line references.
- Read mentioned files yourself BEFORE spawning sub-agents.
- Wait for ALL sub-agents before synthesizing.
- If sub-agent results conflict, investigate further — don't just pick one.
