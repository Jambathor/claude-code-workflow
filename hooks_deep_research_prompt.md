# Deep Research Prompt: Claude Code Hooks

Use this prompt with a deep research agent (Claude with web search, or Perplexity, or similar). The goal is to find real-world implementations, best practices, and gotchas for Claude Code hooks — particularly for building a `/setup_hooks` command that auto-configures hooks based on detected project stack.

---

## Research Prompt

I need comprehensive research on Claude Code hooks — the mechanism that runs code before/after Claude takes actions. I want to build a `/setup_hooks` command that detects a project's stack and automatically configures appropriate hooks. Research the following areas thoroughly:

### 1. Official Documentation
- Search https://docs.claude.com for all hook-related documentation
- What hook types exist? (PreToolUse, PostToolUse, Notification, Stop, etc.)
- What's the exact configuration format in `.claude/settings.json`?
- What matcher patterns are available? (Write, Edit, Bash, etc.)
- What environment variables are passed to hook scripts?
- What are the return value conventions? (exit codes, JSON output)
- Can hooks block/modify/cancel tool use?
- What's the execution model — sync or async? Timeout behavior?

### 2. Real-World Implementations
Search GitHub for:
- `".claude/settings.json" hooks` — find repos with hook configurations
- `"PostToolUse" "PreToolUse" claude` — find actual hook implementations
- `claude-code hooks` — blog posts, tutorials, discussions
- `dangerously-skip-permissions hooks claude` — the skip-permissions + hooks pattern

For each implementation found, capture:
- What does the hook do?
- What's the trigger (matcher)?
- What's the actual script/command?
- Does it work well? Any issues reported?

### 3. Specific Hook Patterns I Want to Implement

Research how people have implemented these specific patterns:

**a. Auto-lint/format on file write**
- PostToolUse on Write/Edit → runs project linter/formatter
- How to detect which linter (eslint, prettier, ruff, rustfmt, gofmt)?
- How to run only on the changed file (not the whole project)?
- How to handle the case where formatting changes the file Claude just wrote?

**b. Auto-test on file write**
- PostToolUse on Write/Edit → runs relevant tests
- How to map a changed file to its test file?
- How to run only relevant tests (not full suite)?
- Jest --findRelatedTests, pytest with specific file, etc.
- How to handle test failures — does the hook output feed back to Claude?

**c. Dangerous command blocker**
- PreToolUse on Bash → blocks rm -rf, force push, DROP TABLE, etc.
- What's the pattern list people use?
- How to handle false positives (legitimate rm commands)?
- Can the hook ask for confirmation, or does it only block/allow?

**d. Context/token monitoring**
- Is there a hook that fires on context thresholds?
- Can hooks access current token usage?
- How to trigger a reminder to /clear or create a handoff?

**e. Auto-commit or checkpoint**
- PostToolUse hooks that create git checkpoints
- How to avoid too-frequent commits?
- Patterns for "commit after passing tests" automation

**f. Type-checking on write**
- PostToolUse on Write/Edit → runs tsc --noEmit or equivalent
- How to scope to changed files?

### 4. The `/setup_hooks` Command Design

Research these design questions:
- How do existing tools auto-detect project stack? (look at how tools like lint-staged, husky, lefthook detect configs)
- What's the best way to write hooks — inline shell commands vs. script files?
- How to make hooks fast enough to not slow Claude down?
- How to handle monorepos with multiple stacks?
- How to avoid hooks that conflict with each other?
- Should hooks be in `.claude/settings.json` or in separate script files that settings.json references?

### 5. Community Discussion & Tips
Search these sources:
- Reddit r/ClaudeAI, r/ClaudeCode for hook discussions
- Hacker News threads about Claude Code hooks
- Discord communities (if indexable)
- Twitter/X posts about Claude Code hooks
- YouTube videos demonstrating hook setups
- The Anthropic community forums

For each discussion, capture:
- What hook configuration did they share?
- What worked? What didn't?
- Any surprising behaviors or gotchas?
- Performance impact observations?

### 6. Edge Cases & Gotchas
- What happens when a hook fails? Does Claude see the error?
- What happens when a hook is slow? Timeout behavior?
- Do hooks run in sub-agent contexts or only main context?
- Can hooks modify the tool's arguments before execution?
- How do hooks interact with `dangerously-skip-permissions`?
- Are there hooks that people regret adding? Why?
- Memory/performance impact of aggressive hook usage?

### 7. Adjacent Systems to Learn From
- Git hooks (pre-commit, husky, lefthook) — mature ecosystem, what patterns transfer?
- lint-staged — the "run tool only on changed files" pattern
- GitHub Actions — workflow trigger patterns
- VS Code tasks — similar "run on save" concept

---

## Output Format

For each area, provide:
1. **What exists** — facts, links, code examples
2. **What works** — validated by community usage
3. **What doesn't work** — gotchas, failures, warnings
4. **What's unknown** — gaps in documentation or community knowledge

Include exact URLs for every source. Prefer GitHub repos with actual code over blog posts with opinions.
