---
description: Log an unusually successful outcome to capture what worked and why.
model: opus
---

# Log Success

You are helping the user capture what made this session unusually successful. The goal is to identify repeatable patterns.

## Process

### Step 1: Interview

Ask:
- "What was the prompt or sequence that led to this outcome?"
- "What did you do differently this time?"
- "How was context managed during this session?"
- "Were there any close calls or moments where it almost went wrong?"
- "What would you tell past-you to do again?"

### Step 2: Log

Write to `docs/logs/successes/success-{NNN}.md`:

```markdown
# Success #{NNN}: {Short Name}
**Date:** {date}
**Project:** {project}

## What Happened
{2-3 sentences — what was accomplished and why it was notable}

## What Made It Work

### Prompt Quality
{what was good about the prompts used}

### Context Management
{how context was managed — /clear timing, compaction, handoffs}

### Workflow
{what workflow choices contributed — sub-agents, validation, tools}

## The Key Prompt(s)
```
{the prompt(s) that worked well — verbatim}
```

## Why This Prompt Worked
{analysis of what made it effective}

## Repeatable Pattern
{distilled into a pattern that can be reused}

## Should This Become a Rule?
{yes/no — if yes, suggest an addition to CLAUDE.md or a new command}
```

## Critical Rules

- Get verbatim prompts. Good prompts are as valuable to study as bad ones.
- Focus on what's REPEATABLE, not what was lucky.
- If a pattern emerges across multiple successes, it should become a command or CLAUDE.md rule.
- Check `docs/logs/successes/` for patterns across entries.
