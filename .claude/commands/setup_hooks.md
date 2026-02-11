---
description: Detect project stack and configure Claude Code hooks (format, block dangerous commands, async tests, git checkpoint).
model: opus
---

# Setup Hooks

You are configuring Claude Code hooks for this project. You will detect the stack, generate script files, and write the settings.json configuration.

## Input

$ARGUMENTS can be:
- Empty — auto-detect everything
- A stack hint — e.g., "nextjs with pnpm" to skip detection

## Process

### Step 1: Detect Stack (4-Layer Algorithm)

Scan the repo root. Check in order — stop at first confident match per category.

**Language + Framework (check manifest files):**

| File | Stack | Framework Detection |
|------|-------|-------------------|
| `package.json` | Node/TypeScript | Parse deps for `next`, `react`, `vue`, `svelte`, `express`, `fastify`, `hono` |
| `tsconfig.json` | TypeScript (confirms TS if package.json found) | |
| `pyproject.toml` | Python | Parse for `django`, `fastapi`, `flask` |
| `requirements.txt` | Python | Scan for framework names |
| `Cargo.toml` | Rust | |
| `go.mod` | Go | |
| `Gemfile` | Ruby | Parse for `rails`, `sinatra` |

**Package manager (check lockfiles):**

| File | Manager |
|------|---------|
| `bun.lock` or `bun.lockb` | bun |
| `pnpm-lock.yaml` | pnpm |
| `yarn.lock` | yarn |
| `package-lock.json` | npm |
| `poetry.lock` | poetry |
| `uv.lock` | uv |
| Also check `package.json` `"packageManager"` field | |

**Formatter/Linter (check config files):**

| File | Tool |
|------|------|
| `biome.json` or `biome.jsonc` | Biome (preferred — fast) |
| `.prettierrc*` or `prettier.config.*` | Prettier |
| `eslint.config.*` or `.eslintrc.*` | ESLint |
| `ruff.toml` or `[tool.ruff]` in pyproject.toml | Ruff |
| `.golangci.yml` | golangci-lint |
| `rustfmt.toml` | rustfmt |

**Test runner:**

| File/Pattern | Runner |
|-------------|--------|
| `vitest.config.*` | Vitest |
| `jest.config.*` or `"jest"` in package.json | Jest |
| `pytest.ini` or `[tool.pytest]` in pyproject.toml | pytest |
| `Cargo.toml` (Rust) | cargo test |
| `*_test.go` files | go test |

**Existing hook infrastructure (avoid conflicts):**

Check for `.husky/`, `lefthook.yml`, `.pre-commit-config.yaml`, existing `.claude/settings.json` with hooks.

### Step 2: Present Detection Results

Tell the user what was detected:
```
Stack detected:
- Language: TypeScript
- Framework: Next.js
- Package manager: pnpm
- Formatter: Biome (biome.json found)
- Linter: ESLint (eslint.config.js found)
- Test runner: Vitest (vitest.config.ts found)
- Existing hooks: None

I'll configure:
1. PostToolUse formatter → biome format (with suppressOutput)
2. PreToolUse dangerous command blocker
3. PostToolUse async test runner → vitest --related
4. Stop git checkpoint (with stop_hook_active guard)

Proceed?
```

Wait for confirmation before writing files.

### Step 3: Generate Hook Scripts

Create `.claude/hooks/` directory in the project root. Write these script files:

**`.claude/hooks/format.sh`** — PostToolUse formatter

```bash
#!/bin/bash
# PostToolUse hook: auto-format changed files
# Reads tool_input.file_path from stdin JSON

FILE_PATH=$(cat | jq -r '.tool_input.file_path // empty')
if [ -z "$FILE_PATH" ] || [ ! -f "$FILE_PATH" ]; then
  exit 0
fi

EXTENSION="${FILE_PATH##*.}"

case "$EXTENSION" in
  # --- Node/TypeScript ---
  js|jsx|ts|tsx|json|css|scss|html|md)
    # DETECT_FORMATTER: replaced by setup_hooks based on detection
    DETECT_FORMATTER_COMMAND
    ;;
  # --- Python ---
  py)
    if command -v ruff &>/dev/null; then
      ruff format "$FILE_PATH" &>/dev/null
    elif command -v black &>/dev/null; then
      black -q "$FILE_PATH" &>/dev/null
    fi
    ;;
  # --- Go ---
  go)
    if command -v goimports &>/dev/null; then
      goimports -w "$FILE_PATH" &>/dev/null
    fi
    gofmt -w "$FILE_PATH" &>/dev/null
    ;;
  # --- Rust ---
  rs)
    if command -v rustfmt &>/dev/null; then
      rustfmt "$FILE_PATH" &>/dev/null
    fi
    ;;
esac

# Suppress output to prevent context window pollution
echo '{"suppressOutput": true}'
```

Replace `DETECT_FORMATTER_COMMAND` with the actual detected formatter:
- If Biome: `npx @biomejs/biome format --write "$FILE_PATH" &>/dev/null`
- If Prettier: `npx prettier --write "$FILE_PATH" &>/dev/null`
- If neither found: remove the JS/TS case or add a comment

**`.claude/hooks/block_dangerous.sh`** — PreToolUse Bash blocker

```bash
#!/bin/bash
# PreToolUse hook: block dangerous bash commands
# Exit 2 = block with feedback to Claude. Exit 0 = allow.

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Hard blocks — always prevent
HARD_BLOCKS=(
  'rm\s+-[rRf]'
  'sudo\s+rm'
  'chmod\s+777'
  'git\s+push\s+.*--force.*\s+(main|master)'
  'git\s+push\s+-f.*\s+(main|master)'
  'DROP\s+(TABLE|DATABASE)'
  'DELETE\s+FROM\s+\w+\s*;'
  'TRUNCATE\s+TABLE'
  'curl\s+.*\|\s*(ba)?sh'
  'wget\s+.*\|\s*(ba)?sh'
  '>\s*/etc/'
  'mkfs\.'
  ':(){.*};'
)

for pattern in "${HARD_BLOCKS[@]}"; do
  if echo "$COMMAND" | grep -qPi "$pattern"; then
    echo "BLOCKED: Command matches dangerous pattern: $pattern" >&2
    exit 2
  fi
done

# Soft blocks — trigger permission dialog
SOFT_BLOCKS=(
  'git\s+reset\s+--hard'
  'git\s+clean\s+-[fdx]'
  'git\s+checkout\s+\.\s*$'
  'rm\s+-[^rRf]*[rR]'
  'DELETE\s+FROM\s+\w+\s+WHERE'
)

for pattern in "${SOFT_BLOCKS[@]}"; do
  if echo "$COMMAND" | grep -qPi "$pattern"; then
    echo '{"permissionDecision": "ask", "message": "This command could cause data loss. Please confirm."}'
    exit 0
  fi
done

exit 0
```

**`.claude/hooks/test_related.sh`** — PostToolUse async test runner

```bash
#!/bin/bash
# PostToolUse hook (async): run tests related to changed file
# Results delivered to Claude on next turn via additionalContext

FILE_PATH=$(cat | jq -r '.tool_input.file_path // empty')
if [ -z "$FILE_PATH" ] || [ ! -f "$FILE_PATH" ]; then
  exit 0
fi

# Skip test files themselves to avoid loops
if echo "$FILE_PATH" | grep -qE '\.(test|spec)\.(ts|tsx|js|jsx)$|_test\.(go|py)$|test_.*\.py$'; then
  exit 0
fi

# DETECT_TEST_RUNNER: replaced by setup_hooks based on detection
TEST_OUTPUT=$(DETECT_TEST_COMMAND 2>&1)
TEST_EXIT=$?

if [ $TEST_EXIT -ne 0 ]; then
  # Feed failure back to Claude
  ESCAPED=$(echo "$TEST_OUTPUT" | tail -30 | jq -Rs .)
  echo "{\"additionalContext\": \"Test failure for $FILE_PATH:\\n$( echo "$TEST_OUTPUT" | tail -30 )\"}"
  exit 0
fi

echo '{"suppressOutput": true}'
```

Replace `DETECT_TEST_COMMAND` based on detection:
- Vitest: `npx vitest related "$FILE_PATH" --run 2>&1`
- Jest: `npx jest --findRelatedTests "$FILE_PATH" --passWithNoTests 2>&1`
- pytest: `python -m pytest "$FILE_PATH" --no-header -q 2>&1` (needs file mapping logic)
- cargo test: `cargo test --quiet 2>&1`
- go test: `go test ./$(dirname "$FILE_PATH")/... 2>&1`

**`.claude/hooks/checkpoint.sh`** — Stop git checkpoint

```bash
#!/bin/bash
# Stop hook: create lightweight git checkpoint when Claude finishes responding

INPUT=$(cat)

# CRITICAL: prevent infinite loops
STOP_HOOK_ACTIVE=$(echo "$INPUT" | jq -r '.stop_hook_active // false')
if [ "$STOP_HOOK_ACTIVE" = "true" ]; then
  exit 0
fi

# Only checkpoint if there are changes
if git diff --quiet && git diff --cached --quiet; then
  exit 0
fi

# Create checkpoint (doesn't affect working tree or staging)
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BRANCH=$(git branch --show-current 2>/dev/null || echo "detached")
git stash push -m "claude-checkpoint-${BRANCH}-${TIMESTAMP}" --include-untracked &>/dev/null
git stash pop &>/dev/null

echo '{"suppressOutput": true}'
```

### Step 4: Write settings.json

Create or merge into `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/block_dangerous.sh\"",
            "timeout": 5
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/format.sh\"",
            "timeout": 15
          },
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/test_related.sh\"",
            "timeout": 120,
            "async": true
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/checkpoint.sh\"",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

If `.claude/settings.json` already exists, MERGE the hooks key — do NOT overwrite existing settings (permissions, env, etc.).

### Step 5: Make Scripts Executable and Report

```bash
chmod +x .claude/hooks/*.sh
```

Tell the user:
```
Hooks configured:
1. ✓ Format on write — {formatter} (suppressOutput enabled)
2. ✓ Dangerous command blocker — hard blocks + soft blocks with confirmation
3. ✓ Async test runner — {test runner} --related
4. ✓ Git checkpoint on Stop — with infinite loop guard

Files created:
- .claude/hooks/format.sh
- .claude/hooks/block_dangerous.sh
- .claude/hooks/test_related.sh
- .claude/hooks/checkpoint.sh
- .claude/settings.json (merged)

⚠️ Restart Claude Code or run /hooks to activate.
```

## Critical Rules

- ALWAYS use `suppressOutput: true` on formatting hooks to prevent context pollution.
- ALWAYS use `async: true` on test hooks to prevent blocking Claude.
- ALWAYS check `stop_hook_active` in Stop hooks to prevent infinite loops.
- Use `""` (empty string) for wildcard matcher, NEVER `*`.
- Use script files, not complex inline commands.
- MERGE into existing settings.json, never overwrite.
- Use `$CLAUDE_PROJECT_DIR` for path resolution in settings.json.
- Timeout: 5s for blocker, 15s for formatter, 120s for tests, 10s for checkpoint.
