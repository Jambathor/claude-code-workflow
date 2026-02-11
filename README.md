# Claude Code Workflow

Commands, agents, and patterns for structured AI-assisted development with [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## What This Is

A set of Claude Code slash commands and custom agents that enforce a disciplined development workflow: **Spec → Plan → Implement → Validate**. Each phase produces a file on disk that serves as the handoff to the next phase, keeping context clean and work traceable.

## Installation

Copy or symlink the `.claude/` directory into your project root (or `~/.claude/` for global access):

```bash
# Per-project
cp -r .claude/ /path/to/your/project/.claude/

# Global
cp -r .claude/ ~/.claude/
```

The commands will appear as `/command-name` in Claude Code. Agents are spawned automatically by commands that need them.

## What's Included

### Commands (14)

**Project Setup**
| Command | Purpose |
|---------|---------|
| `/scaffold` | Create a new project from scratch with CLAUDE.md, hooks, and passing tests |
| `/adapt_to_codebase` | Scan an existing repo and generate a project-specific CLAUDE.md |
| `/setup_hooks` | Auto-detect stack and configure format, test, and safety hooks |

**Specification**
| Command | Purpose |
|---------|---------|
| `/draft` | Collaborative design session for complex features (interview-style) |
| `/specify` | Turn a feature description into a structured spec with user stories and ACs |
| `/clarify` | Resolve ambiguities in specs through targeted questions (max 5 per pass) |

**Planning & Implementation**
| Command | Purpose |
|---------|---------|
| `/create_plan` | Convert spec into phased implementation plan with exact file paths and success criteria |
| `/implement_plan` | Orchestrate implementation — spawns sub-agents per phase, runs verification gates |
| `/validate_plan` | Check for drift between plan and actual implementation |

**Context Management**
| Command | Purpose |
|---------|---------|
| `/create_handoff` | Capture full session state for resuming later |
| `/resume_handoff` | Pick up work from a handoff document (validates repo state first) |

**Research & Learning**
| Command | Purpose |
|---------|---------|
| `/research_codebase` | Deep dive into specific areas with parallel sub-agent research |
| `/log_error` | Post-mortem on what went wrong — focuses on the prompt, not the model |
| `/log_success` | Capture what worked and why, extract repeatable patterns |

### Agents (5)

| Agent | Question It Answers |
|-------|-------------------|
| `codebase_locator` | **Where** does code live? Returns categorized file lists |
| `codebase_analyzer` | **How** does code work? Traces data flow with file:line refs |
| `codebase_pattern_finder` | **What** similar patterns exist? Returns actual code snippets to follow |
| `learn` | Captures hard-won insights to `docs/learnings.md` |
| `test_app` | Systematic QA via Chrome DevTools MCP (quick/normal/thorough) |

### Cheatsheet

`cheatsheet.md` is a scenario-based reference guide. It maps situations to commands:

- Starting a project (greenfield, brownfield, joining a team)
- Feature development loop
- Context management (when to clear, when to hand off)
- Debugging patterns
- Rate limits and interruptions
- Working with specs and plans
- Git and collaboration

## How It Works

The workflow is built around a few key ideas:

**Files on disk are the handoffs.** Specs, plans, and handoff docs live in `docs/`. When you `/clear` context between phases, the file is what carries state forward.

**Main context orchestrates, sub-agents implement.** During `/implement_plan`, the main context spawns a sub-agent per phase, then independently verifies their work. This catches hallucinated "all tests pass" claims.

**Verification gates between phases.** Each plan phase has automated success criteria (commands that must pass) and manual checks (human confirms). Both must pass before moving to the next phase.

**Context is managed aggressively.** The cheatsheet has rules for when to clear and hand off based on context usage. The recommended flow clears context between spec→plan and plan→implement.

## Typical Flow

```
/draft (if design is unclear)
  → /specify
    → /clarify (if needed)
      → /clear
        → /create_plan
          → /clear
            → /implement_plan (phases with gates)
              → /validate_plan
```

For existing codebases, start with `/adapt_to_codebase` and `/setup_hooks`.

## File Structure

```
.claude/
├── agents/
│   ├── codebase_analyzer.md
│   ├── codebase_locator.md
│   ├── codebase_pattern_finder.md
│   ├── learn.md
│   └── test_app.md
└── commands/
    ├── adapt_to_codebase.md
    ├── clarify.md
    ├── create_handoff.md
    ├── create_plan.md
    ├── draft.md
    ├── implement_plan.md
    ├── log_error.md
    ├── log_success.md
    ├── research_codebase.md
    ├── resume_handoff.md
    ├── scaffold.md
    ├── setup_hooks.md
    ├── specify.md
    └── validate_plan.md
cheatsheet.md
```

## The Five Rules

1. Never implement without a plan
2. Never debug in main context — sub-agents debug, main context orchestrates
3. Never push through degraded context (>60%) — hand off and clear
4. Never trust without verification — run the checks yourself
5. Never lose a learning — `/log_error` and `/log_success` exist for a reason

## License

MIT
