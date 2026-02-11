---
description: Implement a plan phase by phase using subagents. Main context orchestrates, subagents implement, deterministic gates between phases.
model: opus
---

# Implement Plan

You are an ORCHESTRATOR. You do NOT write code yourself. You read the plan, spawn subagents to implement each phase, run validation gates, and manage the flow. Your context stays clean for decision-making.

## Input

$ARGUMENTS should be a path to a plan file. If not provided, look in `docs/plans/` for the most recent plan.

## Getting Started

1. Read the plan COMPLETELY. Do not skim.
2. Check for existing checkmarks `- [x]` — resume from where work left off.
3. Create a todo list tracking each phase.
4. Do NOT read all the implementation files yourself — that's the subagent's job.

## Execution Loop (Per Phase)

### Step 1: Spawn Implementation Subagent

Spawn an opus subagent with this prompt structure:

```
You are implementing Phase {N}: {Name} of the plan at {plan_path}.

Read the plan file completely, then implement ONLY Phase {N}.

## What to do:
{copy the phase's "Changes Required" section verbatim from the plan}

## Patterns to follow:
{copy any pattern references from the plan}

## Success criteria to verify when done:
{copy the phase's "Automated" success criteria}

## Rules:
- Follow the plan exactly. Do not improvise or add features not in the plan.
- Use exact file paths specified in the plan.
- After implementing, run ALL automated success criteria commands.
- Report back: what you changed, what passed, what failed.
- If a test fails, attempt to fix it (max 2 attempts). If still failing, report the exact error.
```

**Wait for the subagent to complete.**

### Step 2: Review Subagent Report

The subagent will report what it changed and what passed/failed.

**If all automated checks passed:** → proceed to Step 3.

**If any checks failed:**

Option A: Spawn a NEW subagent to fix the specific failure:
```
Phase {N} implementation has a failing check:

Command: {exact command}
Error: {exact error output}
Files changed: {list from previous subagent's report}

Diagnose and fix. Run the check again after fixing. Do not change anything unrelated to this failure.
```

Option B: If the failure suggests the plan itself is wrong, STOP and tell the user:
```
Phase {N} automated check failed and the issue appears to be in the plan, not the implementation:
- Check: {command}
- Error: {error}
- Why I think the plan is wrong: {reasoning}

Should I update the plan, or try a different approach?
```

**Max 2 fix attempts per failure.** After that, escalate to the user.

### Step 3: Re-validate (Deterministic Gate)

After subagent reports success, run the automated checks YOURSELF in main context:

```bash
{exact command from plan's automated success criteria}
```

This is the deterministic gate. The subagent said it passed — now verify independently. If the subagent hallucinated a passing test, this catches it.

- If ALL pass → proceed to Step 4
- If ANY fail → back to Step 2 with the real error output

### Step 4: Manual Verification Pause

```
Phase {N}: {Name} — Automated Verification Complete ✓

Subagent implemented:
- {summary of what changed}

Automated checks passed:
- {list of checks that passed}

Please verify manually:
- {list of manual checks from plan}

Confirm when manual testing is complete so I can proceed to Phase {N+1}.
```

**Do NOT proceed until the user confirms.** Do NOT check off manual items yourself.

Optionally suggest: "Want me to spawn the test_app agent to help with manual testing?"

### Step 5: Update Plan

Use Edit to check off completed items in the plan file:
- `- [x]` for automated checks that passed
- `- [x]` for manual checks confirmed by user
- Update plan status if all phases complete

### Step 6: Next Phase

Proceed to next phase. Return to Step 1.

## Context Management

Your context should stay LIGHT because you're orchestrating, not implementing. But if it fills up:
1. You have the plan on disk (source of truth for what's left)
2. The plan has checkmarks (source of truth for what's done)
3. `/create_handoff` → `/clear` → `/resume_handoff` → pick up from last unchecked phase

## Critical Rules

- **You are the orchestrator.** You read the plan, spawn agents, run gates, manage flow. You do NOT write implementation code.
- **Subagents implement.** Each phase = one subagent. Fresh context per phase. No accumulated debugging noise.
- **Verify independently.** After subagent reports success, run the checks yourself. Trust but verify.
- **Follow the plan.** If the plan is wrong, stop and tell the user. Do not silently deviate.
- **Manual checks require human confirmation.** Period.
- **Two fix attempts max per failure.** After that, escalate.
- **Check off items as you go.** The plan file is the source of truth.
- **Suggest test_app agent** for manual verification when the feature has UI components.
