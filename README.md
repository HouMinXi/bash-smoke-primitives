# bash-smoke-primitives

Bash assertion primitives for smoke testing. 19 reusable functions. Zero dependencies beyond `jq`. One line to get started:

```bash
source primitives.sh
```

## Why This Exists

Bash has no standard test framework. Every AI-written shell test reinvents assertion logic from scratch — inconsistent, error-prone, and hard to review.

Five runtime bugs routinely evade `bash -n` and `shellcheck`:

| Footgun | What Happens | Why Static Analysis Misses It |
|---------|-------------|-------------------------------|
| Bash auto-reap | `wait` on already-exited children returns immediately, hiding zombie processes | Runtime behavior, no syntax error |
| `local` scope | Variables declared `local` in a function are invisible to subshells, silently producing empty output | Lexically valid, logically wrong |
| `((x++))` return value | `((0))` returns exit code 1, so `set -e` exits on zero-value expressions | Valid arithmetic, surprising semantics |
| `$(...)` multi-line | Command substitution strips trailing newlines, corrupting multi-line output comparisons | Valid syntax, data-dependent behavior |
| `jq -e` stdout pollution | `jq -e` writes matching output to stdout even in boolean mode, mixing data with control flow | Valid jq usage, unexpected side effect |

This library provides 19 tested, reviewed assertion functions so AI (and humans) write consistent, debuggable shell tests.

## What's Inside

### Helpers (3)
| Function | Purpose |
|----------|---------|
| `run_and_capture` | Execute a command, capture stdout/stderr/status |
| `run_concurrent` | Execute N instances in parallel |
| `concurrent_wait` | Wait for all concurrent instances, reaping zombies |

### Assertions (16)

**Execution (3):** `assert_success`, `assert_failure`, `assert_exit_code`

**Output (4):** `assert_output_contains`, `assert_output_not_contains`, `assert_stderr_empty`, `assert_stderr_contains`

**File/State (3):** `assert_file_exists`, `assert_file_not_exists`, `assert_file_contains`

**Security (3):** `assert_no_command_exec`, `assert_no_command_exec_json`, `assert_no_path_traversal`

**Data (1):** `assert_json_valid`

**Concurrency (1):** `assert_no_zombie`

**Safety (1):** `assert_temp_clean`

## Design Principles

1. **Act before Assert** — run the command first (`run_and_capture`), then assert on captured state. Never assert on live `$?`.
2. **Values through parameters** — captured stdout/stderr/status go directly into assertion parameters, not global regex.
3. **One assertion per expected behavior** — don't combine checks into one `[[ ... ]]`.
4. **Zero dependencies** — only requires `jq` for JSON validation (skips gracefully if unavailable).

## Install

```bash
# Option 1: git clone
git clone https://github.com/houminxi/bash-smoke-primitives.git
source bash-smoke-primitives/test-library/shell/primitives.sh

# Option 2: curl single file
curl -O https://raw.githubusercontent.com/houminxi/bash-smoke-primitives/main/test-library/shell/primitives.sh
source primitives.sh
```

## Claude Code Integration

This repository includes a `SKILL.md` for Claude Code users. Install it as a skill:

```bash
mkdir -p ~/.claude/skills/smoke-test/
cp SKILL.md ~/.claude/skills/smoke-test/
cp -r test-library/ references/ ~/.claude/skills/smoke-test/
```

The skill provides automated decision tables for selecting the right primitives based on change type (CLI parameter, error handling, file I/O, concurrency, security patch).

Non-Claude-Code users can ignore `SKILL.md` and use `source primitives.sh` directly.

## Evidence

- **46 PASS, 0 FAIL** self-tests covering every primitive function
- **12 design iterations** including multi-AI peer review across 3 independent models
- Methodology documented in `evidence/design-iterations.md`

## What This Is NOT

- **Not a cross-language framework** — Python uses `pytest`, Go uses `go test`. This library exists because bash has no standard test framework. For other languages, use their native tools.
- **Not a unit test framework** — designed for smoke testing (runtime verification after code changes), not TDD-style unit tests.
- **Not POSIX shell** — uses bash features (`[[ ]]`, `(( ))`, arrays). Requires bash 4.0+.

## License

MIT — see [LICENSE](LICENSE).
