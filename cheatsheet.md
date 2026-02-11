# Workflow Cheat Sheet

Every scenario you'll hit, what to do, and what NOT to do.

---

## STARTING A PROJECT

### Greenfield (new project, nothing exists yet)

```
1. /scaffold "description of what you want to build"
   → Asks up to 3 questions (what, tech preferences, hosting)
   → Creates project structure, installs deps, writes one passing test
   → Generates CLAUDE.md with verified commands
   → Configures hooks (format, block, test, checkpoint)
   → Makes initial commit
   → You go from zero to configured project in one command

2. /specify "first feature you want to build"
   → Generates structured spec with user stories, requirements, acceptance criteria
   → Makes assumptions where you didn't specify (documented, changeable)
   → Max 3 [NEEDS CLARIFICATION] markers

3. /clarify docs/specs/0001-feature.md   → resolves ambiguities
4. /create_plan docs/specs/0001-feature.md → implementation plan
5. /implement_plan docs/plans/plan.md     → builds it phase by phase
```

**Non-technical?** `/scaffold` handles everything — you don't need to know what npm is or how to init a repo. Just describe what you want to build.

**What NOT to do:** Don't say "build me a todo app" and let Claude freestyle. Use `/specify` first — even for simple features. The 2 minutes writing a spec saves 20 minutes of "that's not what I meant."

### Brownfield (existing codebase, first time using the workflow)

```
1. /adapt_to_codebase          → scans repo, writes CLAUDE.md
2. /setup_hooks                → hooks configured to your stack
3. /research_codebase "overall architecture, key entry points, how tests are structured"
4. Read the research output — this is YOUR onboarding too
5. Now proceed as normal: spec → /clarify → /create_plan → /implement_plan
```

**What NOT to do:** Don't skip step 3. If Claude doesn't understand the codebase patterns, it'll generate code that doesn't match existing conventions. The 5 minutes spent on research saves hours of rework.

### Joining someone else's project

```
1. /adapt_to_codebase
2. /research_codebase "conventions, patterns, where things live"
3. Read their CLAUDE.md if it exists — merge with yours, don't replace
4. /setup_hooks (check for existing hooks first — don't duplicate)
```

---

## FEATURE DEVELOPMENT (the main loop)

### Starting a new feature

```
For complex features (unclear requirements, big scope):
0. /draft "feature name"
   → Collaborative design doc — AI interviews you, writes structured doc
   → Output feeds into /specify as input
   → Use when you need to think through the design first

For features you can describe clearly:
1. /specify "what you want to build"
   → Creates structured spec with user stories, requirements, acceptance criteria
   → Review the Assumptions section — change anything that's wrong
   → Check Out of Scope — make sure it matches your intent

2. /clarify docs/specs/NNNN-feature.md   → max 5 questions, answers encoded back

   *** /clear here — spec phase is done, plan phase starts with fresh context ***
   (The spec file on disk IS the handoff between phases)

3. /create_plan docs/specs/NNNN-feature.md
   → Plan with phases, success criteria, exact file paths
   → Review yourself:
     - Do the file paths look right?
     - Do the success criteria actually verify what you care about?
     - Is "What we're NOT doing" accurate?

   *** /clear here — plan phase is done, implementation starts with fresh context ***
   (The plan file on disk IS the handoff between phases)

4. /implement_plan docs/plans/plan.md
   → Orchestrator spawns subagents per phase
   → Deterministic gates verify each phase independently
   → Pauses for manual verification between phases

5. /validate_plan docs/plans/plan.md     → drift check after implementation
```

**When to /clear between phases:** Always between spec→plan and plan→implement. The spec and plan files on disk are the handoffs — you don't need /create_handoff for phase transitions. Only use /create_handoff when stopping mid-implementation.

### Resuming work on an existing feature

```
If you have a handoff:
  /resume_handoff docs/handoffs/latest.md

If you don't:
  /research_codebase "current state of {feature}"
  Read the plan in docs/plans/
  /implement_plan plan.md      → picks up from last checked-off phase
```

### Small bug fix (no plan needed)

```
1. Describe the bug clearly, including how to reproduce
2. Let Claude fix it
3. Hooks auto-run tests + format
4. If fix was hard: /log_error or /log_success
```

No need for /create_plan on a bug fix. Don't add ceremony where it doesn't earn its place.

### Refactoring

```
1. /research_codebase "all usages of {thing being refactored}"
2. /create_plan "refactor {thing}: {why and what it should become}"
   - Plan MUST include "What we're NOT doing" — refactors love to scope-creep
3. /implement_plan → smaller phases than features, more validation gates
4. /validate_plan → critical here — refactors are where drift happens most
```

---

## CONTEXT MANAGEMENT

### Context is at ~50-60% (yellow zone)

```
Options (pick one):
a. Finish current phase → /create_handoff → /clear → /resume_handoff
b. Finish current phase → manual /compact with specific notes about what to keep
c. If mid-debug: double-escape → restore conversation only (keep code, drop noise)
```

**Prefer (a) over (b).** Compaction is lossy. Handoff + clear gives you fresh context with explicit state.

### Context is at ~80%+ (red zone)

```
STOP what you're doing.
/create_handoff              → captures everything
/clear                       → fresh start
/resume_handoff              → back in business
```

Do not try to squeeze in "one more thing." Degraded context = degraded output = more work later.

### Context filled up mid-implementation

```
Don't panic. The code changes are on disk, they're not lost.
/create_handoff "was implementing phase N of plan.md, automated checks X Y passed, working on Z"
/clear
/resume_handoff
```

### After a long debugging session (bug is fixed)

```
Double-escape → restore CONVERSATION only (not code)
Claude now has clean context + working code
The debugging history is gone — this is good
```

### After Claude went on a tangent

```
Double-escape → restore BOTH code and conversation
You're back to before the tangent
Try a different approach with a more specific prompt
```

---

## DEBUGGING

### Tests failing after implementation

```
DO NOT debug in main context. The /implement_plan command handles this:
- It spawns a sub-agent to diagnose
- Sub-agent fixes and re-runs tests
- If 2 attempts fail, it escalates to you
- Main context stays clean

If you're debugging manually:
1. Be specific: "test X fails with error Y, the relevant code is in Z"
2. After fix: double-escape → restore conversation only
```

### Claude is looping (trying the same fix repeatedly)

```
1. Double-escape → restore both code and conversation
2. Rephrase the problem differently
3. Add constraints: "do NOT try {approach that failed}"
4. If still stuck: /clear, write a fresh prompt with the error message and relevant file paths
```

### Claude "fixed" something but broke something else

```
1. git stash or git checkout the broken files (hooks created checkpoints)
2. Double-escape → restore conversation only
3. This time: "fix X without modifying Y — Y must continue to pass {test command}"
```

### You don't understand the error

```
1. Paste the FULL error message (not a summary)
2. Include the file path and line number
3. "Explain this error in the context of {what I was trying to do}"
4. THEN: "Fix it"

Don't ask Claude to fix what you don't understand — you won't know if the fix is right.
```

### Flaky test / intermittent failure

```
1. "Run {test command} 5 times and report which runs pass/fail"
2. If flaky: "This test is flaky. Identify the non-determinism — likely: timing, state leak, or external dependency"
3. Don't let Claude just add retries — that hides the bug
```

---

## RATE LIMITS & INTERRUPTIONS

### Hit rate limit mid-work

```
If mid-implementation:
  - Your code changes are on disk, nothing is lost
  - Hook checkpoints mean you have git snapshots
  - When rate limit clears: /resume_handoff (if you created one) or just continue

If rate limit is long (hours):
  /create_handoff before closing
  When you return: /resume_handoff

Prevent: Use sub-agents aggressively — they spread load across model calls
instead of one long sequential session
```

### Claude Code crashed / terminal closed

```
1. Code changes are on disk — check git status
2. Hook checkpoints: git stash list → find claude-checkpoint-*
3. Check docs/handoffs/ for recent handoffs
4. If none: /research_codebase "recent changes" + read the plan
5. Continue from last known good state
```

### Need to switch to a different task urgently

```
/create_handoff "stopping mid-{task} to handle {urgent thing}"
/clear
Handle urgent thing
When done: /resume_handoff
```

---

## SUB-AGENT ISSUES

### Sub-agent hallucinated / returned wrong info

```
Never trust sub-agent output blindly.
/validate_plan catches this at the phase level.

For immediate verification:
- Ask Claude to "verify {claim} by reading {file:line}"
- Or spawn another sub-agent with the same question — compare answers
- Deterministic check: run the actual command if the claim is testable
```

### Sub-agent is taking too long

```
Sub-agents have no timeout by default.
If stuck: the main agent will eventually report.
Prevention: keep sub-agent tasks focused and small.
"Analyze the entire codebase" = bad.
"Analyze src/auth/middleware.ts and trace the JWT flow" = good.
```

### Main agent tries to debug instead of spawning sub-agent

```
This violates /implement_plan rules but Claude may ignore them.
Interrupt (Escape) → tell Claude:
"Do not debug this yourself. Spawn a sub-agent to diagnose and fix."
If persistent: add to CLAUDE.md: "NEVER debug test failures in main context. Always spawn a sub-agent."
```

---

## WORKING WITH SPECS & PLANS

### Spec is too vague to plan from

```
/clarify spec.md              → will ask up to 5 targeted questions
Run /clarify again if ambiguities remain
Repeat until no [NEEDS CLARIFICATION] markers left
THEN: /create_plan
```

### Plan doesn't match what you actually wanted

```
Don't implement a wrong plan. Fix it first.
1. Tell Claude specifically what's wrong
2. It will update the plan file
3. /clarify plan.md if new ambiguities emerged
4. Only /implement_plan after you've read and approved
```

### Plan is too big (too many phases)

```
Split it. Create two plans:
- plan-v1-{feature}-core.md     → minimum viable version
- plan-v2-{feature}-complete.md → everything else

Implement v1 fully. Validate. Then do v2.
Big plans = more drift = more rework.
```

### Scope creep during implementation

```
/validate_plan → will report "Implemented but NOT in plan" items
Decision:
a. Revert the out-of-scope changes → continue with plan
b. Update the plan to include them → then continue
Never just let scope creep accumulate silently.
```

---

## LEARNING & IMPROVEMENT

### Something went wrong and you don't know why

```
/log_error
- It will interview you about what happened
- Forces you to identify the triggering prompt
- Captures the pattern so you can spot it next time
```

### Something went unusually well

```
/log_success
- Captures the prompt and conditions that worked
- If a pattern repeats across multiple successes → promote to CLAUDE.md or a new command
```

### CLAUDE.md feels stale or bloated

```
1. Read it top to bottom — if any line doesn't apply to EVERY task, remove it
2. Move project-specific learnings to docs/learnings.md
3. Keep CLAUDE.md for universal rules only
4. Run /adapt_to_codebase to regenerate from scratch if needed (it merges, doesn't overwrite)
```

### Recurring mistakes

```
Check docs/logs/errors/ for patterns.
If the same category (e.g., "implicit expectations") keeps appearing:
1. Write a specific rule in CLAUDE.md to prevent it
2. Or create a new command that forces the missing step
3. Or add a hook that catches it automatically
```

---

## GIT & COLLABORATION

### Before committing

```
/validate_plan → confirm implementation matches plan
Review git diff yourself — hooks formatted the code, but verify the logic
Commit with descriptive message (Claude can help draft it)
```

### Creating a PR

```
1. /validate_plan → ensure everything's clean
2. Ask Claude to draft PR description from the plan + git diff
3. Link to the plan in the PR description
```

### Merge conflicts

```
1. Don't let Claude auto-resolve blindly
2. Show the conflicts: git diff --name-only --diff-filter=U
3. For each file: "resolve the merge conflict in {file} — keep {our/their} approach for {reason}"
4. Be explicit about which side wins for each conflict
```

### Code review feedback

```
1. Paste the feedback (or link to PR comments)
2. /research_codebase if the feedback references code you're not familiar with
3. Make changes
4. /validate_plan → confirm you didn't break the plan's intent while addressing feedback
```

---

## EDGE CASES

### Working offline / poor connection

```
Claude Code needs connectivity. If unstable:
- /create_handoff frequently (every 30 min)
- Use smaller, more focused sessions
- Avoid long sub-agent chains — they're more likely to fail mid-flight
```

### Large file changes (migrations, bulk updates)

```
1. Create the plan with explicit file list
2. Implement in small batches — not one massive commit
3. Hook test runner uses --related so it won't run full suite
4. /validate_plan after each batch
```

### Need to work on multiple features simultaneously

```
Use git worktrees:
  git worktree add ../feature-auth feature/auth
  git worktree add ../feature-payments feature/payments

Each worktree gets its own Claude session.
Each has its own plan, its own handoff, its own context.
/create_handoff in one before switching to the other.
```

### Unfamiliar library / API

```
1. Ask Claude directly first (it may know)
2. If Claude's knowledge is stale: "search for {library} docs 2026"
3. If using Context7 MCP: pull the docs into context
4. /research_codebase to see how the library is already used in the project
5. Codebase_pattern_finder agent specifically looks for usage examples
```

### Claude is confidently wrong

```
Don't argue. Don't re-explain.
1. /clear
2. Rephrase with more constraints and explicit expectations
3. Include a concrete example of what you want
4. "Here is an example of the pattern I want: {actual code}. Apply this pattern to {task}."
```

### You're stuck and don't know what to ask

```
/research_codebase "how does {area} work"
Read the output.
The research often reveals the question you didn't know how to ask.
```

---

## COMMAND QUICK REFERENCE

| Scenario | Command |
|----------|---------|
| New project from scratch | `/scaffold` |
| Existing project, first time | `/adapt_to_codebase` → `/setup_hooks` |
| Complex feature (unclear design) | `/draft` → `/specify` → `/clarify` → `/clear` → `/create_plan` → `/clear` → `/implement_plan` |
| Clear feature | `/specify` → `/clarify` → `/clear` → `/create_plan` → `/clear` → `/implement_plan` |
| Resume work | `/resume_handoff` |
| Context filling up | `/create_handoff` → `/clear` → `/resume_handoff` |
| Check for drift | `/validate_plan` |
| Test UI manually | invoke `test_app` agent (quick/normal/thorough) |
| Something broke | Double-escape → restore appropriately |
| Debug noise in context | Double-escape → restore conversation only |
| Claude looping | Double-escape → restore both → rephrase |
| Learned something | invoke `learn` agent |
| Something went wrong | `/log_error` |
| Something went great | `/log_success` |
| Understand codebase | `/research_codebase` |
| Spec is vague | `/clarify` |
| Stopping for the day | `/create_handoff` |
| Someone else's project | `/adapt_to_codebase` → `/research_codebase` |
| Hooks not working | Restart Claude Code or `/hooks` review |

---

## THE 5 RULES (tattoo these)

1. **Never implement without a plan.** Even a 10-line plan. The plan is the contract between your intent and Claude's execution.

2. **Never debug in main context.** Sub-agents debug. Main agent orchestrates. Debugging noise is the #1 context killer.

3. **Never push through degraded context.** If you're past 60%, create a handoff and clear. The 2 minutes spent on handoff saves 20 minutes of confused output.

4. **Never trust without verification.** Hooks run tests automatically. /validate_plan checks drift. Sub-agent C validates sub-agent A. Trust is earned through deterministic checks, not vibes.

5. **Never lose a learning.** /log_error, /log_success, the learn agent. Every mistake and every win is data. The system gets better only if you feed it.
