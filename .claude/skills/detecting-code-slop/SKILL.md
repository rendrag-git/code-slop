---
name: detecting-code-slop
description: Use when reviewing AI-generated code changes for quality issues — stubs, duplication, over-engineering, sycophantic comments, abandoned branches, copy-paste artifacts, scope divergence, missing guardrails, over-specification, and wrong-context patterns. Triggers manually or as a self-check after implementation.
---

# Detecting Code Slop

Code slop: characteristic low-quality patterns in AI-generated code. Two-phase detection: mechanical scan, then LLM judgment.

## Scope

| Scope | Command |
|-------|---------|
| Uncommitted (default) | `git diff` |
| Staged only | `git diff --cached` |
| Branch diff | `git diff main...HEAD` |
| Custom | Specific files or commit range |

## Phase 1: Mechanical Scan

Read `patterns.md` in this skill directory for full grep patterns and signals.

Scan the diff for Tier 1 patterns: **stubs & placeholders, sycophantic comments, copy-paste artifacts, commented-out code.** Record every hit as `file:line — matched text`.

## Phase 2: LLM Review

Read `patterns.md` for detailed signals per pattern.

Check EVERY Tier 2 pattern against the diff — do not skip any: **scope divergence, semantic duplication, over-engineering, abandoned branches, premature generalization, missing guardrails, over-specification, wrong-context patterns.**

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

Verdict is CLEAN only when zero HIGH and zero MED findings remain. Do not add extra sections — findings should be self-explanatory.

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
