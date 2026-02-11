---
description: Log a failure that occurred during agentic coding. Focus on what YOU did wrong, not the model.
model: opus
---

# Log Error

You are helping the user log a failure. The PRIMARY goal is to identify what the USER did wrong in their prompting, context management, or workflow. This builds USER skill — it's not about cataloging model failures.

## Core Philosophy

Errors in agentic coding are almost always traceable to:
1. **Bad Prompt** — ambiguous, missing constraints, too verbose, wrong structure
2. **Context Rot** — didn't /clear, conversation too long, stale context polluting
3. **Bad Harnessing** — wrong agent type, didn't pass context to sub-agents, missing guardrails

The model is the constant. The user's input is the variable. Focus on the variable.

## Process

### Step 1: Interview (5-8 Pointed Questions)

Ask hard questions focused on USER behavior:
- "What was the exact prompt that triggered this?"
- "How full was your context when this happened?"
- "Did you specify what NOT to do, or only what to do?"
- "What constraints were in your head but not in the prompt?"
- "When did you last /clear?"
- "Did you verify the sub-agents received the critical context?"
- "Was this reference material or explicit requirements?"
- "What did 'done' look like in your head? Did you write that down?"

Be CRITICAL. The user is logging this to learn, not to feel good.

### Step 2: Get the Triggering Prompt

Get the EXACT prompt — verbatim, not paraphrased. This is the most valuable piece of data.

### Step 3: Log

Write to `docs/logs/errors/error-{NNN}.md`:

```markdown
# Error #{NNN}: {Short Name}
**Date:** {date}
**Project:** {project}

## What Happened
{2-3 sentences}

## User Error Category
**Primary cause:** {pick ONE}

### Prompt Errors
- [ ] Ambiguous instruction — could be interpreted multiple ways
- [ ] Missing constraints — didn't specify what NOT to do
- [ ] Too verbose — buried key requirements in walls of text
- [ ] Implicit expectations — requirements in head, not in prompt
- [ ] No success criteria — didn't define what "done" looks like
- [ ] Wrong abstraction level — too high-level or too detailed

### Context Errors
- [ ] Context rot — conversation too long, should have /cleared
- [ ] Stale context — old information polluting new responses
- [ ] Context overflow — too much info degraded performance
- [ ] Wrong context — irrelevant information drowning signal

### Harness Errors
- [ ] Wrong agent type — used wrong specialized agent for task
- [ ] No guardrails — didn't constrain agent behavior
- [ ] Parallel when sequential needed — launched agents with dependencies
- [ ] Missing validation — no check that agent output was correct
- [ ] Trusted without verification — accepted output without review

### Meta Errors
- [ ] Didn't ask clarifying questions — could have caught this earlier
- [ ] Rushed to implementation — skipped planning
- [ ] Assumed competence — expected Claude to infer too much

## The Triggering Prompt
```
{exact prompt — verbatim}
```

## What Was Wrong With This Prompt
{specific and critical — what should have been different?}

## What The User Should Have Said Instead
```
{rewritten prompt that would have prevented this error}
```

## The Gap
- **Expected:** {what user expected}
- **Got:** {what actually happened}
- **Why:** {direct connection to user error above}

## Prevention
1. {specific action for next time}
2. {consider adding to CLAUDE.md or workflow}

## One-Line Lesson
{actionable takeaway — not about model behavior}
```

## Critical Rules

- **Be CRITICAL.** 80% focus on user error, 20% on model behavior.
- **Get the verbatim prompt.** Paraphrased prompts are useless for learning.
- **Pick ONE primary cause.** Multiple causes dilute the lesson.
- **The rewritten prompt is the most valuable output.** Make it good.
- **Check for patterns.** If `docs/logs/errors/` has similar errors, call it out — this is a habit to break.
