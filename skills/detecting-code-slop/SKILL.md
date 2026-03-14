---
name: detecting-code-slop
description: Use when reviewing AI-generated code changes for quality issues — stubs, duplication, over-engineering, sycophantic comments, abandoned branches, copy-paste artifacts, scope divergence, missing guardrails, over-specification, and wrong-context patterns. Triggers manually or as a self-check after implementation.
---

# Detecting Code Slop

Code slop: characteristic low-quality patterns in AI-generated code. Two-phase detection: mechanical grep, then LLM judgment.

## Scope

| Scope | Command |
|-------|---------|
| Uncommitted (default) | `git diff` |
| Staged only | `git diff --cached` |
| Branch diff | `git diff main...HEAD` |
| Custom | Specific files or commit range |

## Phase 1: Mechanical Scan

Run these against the diff. Record every hit as `file:line — matched text`.

| Pattern | Grep/Regex |
|---------|-----------|
| Stubs & placeholders | `grep -nE 'TODO\|FIXME\|NotImplementedError\|throw new Error\(.(not implemented\|todo)'` and `pass` in non-`__init__` methods, empty function bodies `\{\s*\}` |
| Sycophantic comments | `grep -nE '^\+\s*(//\|#\|/\*\*?\|[*])\s*(This (function\|method\|class\|module) (does\|is\|will\|handles\|provides\|ensures\|represents))'` — also flag docstrings that restate the function name |
| Copy-paste artifacts | 3+ added blocks sharing >80% token similarity within the diff |
| Commented-out code | `grep -nE '^\+\s*(//\|#)\s*(if\|for\|while\|return\|const\|let\|var\|def\|class\|import) '` — blocks of commented real code, not explanatory comments |

## Phase 2: LLM Review

Check EVERY pattern below against the diff. Do not skip any.

| Pattern | Look for |
|---------|----------|
| Scope divergence | Files/functions unrelated to task, unrequested refactoring or features |
| Semantic duplication | Different-named functions doing the same thing |
| Over-engineering | Abstractions with one caller, config for one-time ops, speculative generality |
| Abandoned branches | Error handling for impossible cases, unreachable if/else paths |
| Premature generalization | Parameters nobody passes, unused options |
| Missing guardrails | No null checks, input validation, or exception handling on edge cases |
| Over-specification | Rigid narrow solution when a general approach would be simpler |
| Wrong-context patterns | Technically valid pattern that violates local conventions |

## Output Format

```
## Slop Check: [scope description]

### Findings (by severity)
- [HIGH] category: file:line — description
- [MED] category: file:line — description
- [LOW] category: file:line — description

### Summary
- N findings (X high, Y medium, Z low)
- Categories hit: list

### Verdict: NEEDS WORK | CLEAN
```

Verdict is CLEAN only when zero HIGH and zero MED findings remain.

## Fix Loop (Optional)

When requested: sort by severity, fix top finding, rescan, report delta (fixed/new/remaining), repeat until clean or 10 iterations. Fixes must not introduce new slop.

## Integration

- Run manually on any diff
- Auto-runs as pre-step in verification-before-completion and requesting-code-review skills
- Self-check: run on own output before reporting task complete

## Watch For

- **Sycophantic comments under-flagged** — check module-level docstrings and `@param` tags adding no information
- **Missed scope divergence** — compare changes against original task; flag anything not directly required
- **Stub cascades** — placeholder calling placeholder; count full chain as HIGH
