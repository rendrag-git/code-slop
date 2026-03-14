# Design: detecting-code-slop

A lightweight Claude Code skill for identifying AI-generated code quality issues in git changes.

## Problem

AI coding assistants produce characteristic "slop" patterns: stubs that never get implemented, semantic duplication, over-engineering, sycophantic comments, abandoned error branches, and scope divergence from the original request. Existing tools (desloppify) are too heavy (Python CLI, pip install) and not targeted enough on git changes. Existing Claude Code skills (code-review, code-simplifier, finding-duplicate-functions) are either general-purpose or narrow.

## Skill Identity

- **Name:** `detecting-code-slop`
- **Location:** `~/.claude/skills/detecting-code-slop/SKILL.md`
- **Description:** Use when reviewing AI-generated code changes for quality issues — stubs, duplication, over-engineering, sycophantic comments, abandoned branches, copy-paste artifacts, scope divergence, missing guardrails, over-specification, and wrong-context patterns. Triggers manually or as a self-check after implementation.
- **Dependencies:** git, grep, Claude. Nothing else.

## Audiences

- Claude self-checking its own output
- User reviewing their own AI-assisted work
- User reviewing someone else's AI-generated code

## Scope (Configurable)

| Default | Option |
|---------|--------|
| Uncommitted changes | `git diff` (staged + unstaged) |
| Staged only | `git diff --cached` |
| Branch diff | `git diff main...HEAD` |
| Custom | Specific files or commit range |

## Slop Pattern Catalog

### Tier 1: Mechanical Detection (grep/regex, high confidence)

| Pattern | Signals |
|---------|---------|
| **Stubs & placeholders** | `TODO`, `FIXME`, `NotImplementedError`, `pass` in non-`__init__`, empty function bodies, `throw new Error("not implemented")` |
| **Sycophantic comments** | Comments restating the function name, `// This function does X` where X is obvious, excessive docstrings on trivial code |
| **Copy-paste artifacts** | 3+ near-identical code blocks in the diff |
| **Commented-out code** | Blocks of commented code (not explanatory comments) |

### Tier 2: LLM Review (requires judgment)

| Pattern | What to look for |
|---------|-----------------|
| **Scope divergence** | New files/functions not related to the task, refactoring unasked-for code, adding unrequested features, "improving" unrelated code |
| **Semantic duplication** | Functions with different names doing the same thing (absorbs finding-duplicate-functions approach) |
| **Over-engineering** | Abstractions serving one caller, config for one-time ops, unnecessary indirection |
| **Abandoned branches** | Error handling for impossible cases, if/else paths that can never execute |
| **Premature generalization** | Parameters nobody passes, options nobody uses, "just in case" flexibility |
| **Missing guardrails** | Omits null checks, early returns, input validation, exception handling — code that works in the happy path but breaks on edge cases |
| **Over-specification** | Narrow, non-reusable solutions — code that solves one case rigidly when a general approach would be simpler and more maintainable |
| **Wrong-context patterns** | Uses a learned pattern in the wrong context — technically works but violates local conventions or best practices for this codebase |

### Backburner (not in v1)

- Dead code (linters handle it)
- Hallucinated imports (compilers catch it)
- Inconsistency with existing code (too subjective)
- Regression from prior state (too hard to detect)
- Excessive I/O (8x more common in AI PRs, but hard to detect from diffs alone)

## Two-Phase Detection Flow

### Phase 1: Mechanical Scan

1. Get diff based on configured scope
2. For each Tier 1 pattern, run regex/grep against the diff
3. Record: file:line, pattern name, matched text, confidence (high)
4. Output: list of concrete findings

### Phase 2: LLM Review

1. Feed the diff + Phase 1 findings + original task context (if available)
2. Evaluate each Tier 2 pattern against the changes
3. For each finding: file:line, pattern, explanation, confidence (high/medium/low)
4. Filter: only report medium+ confidence

## Output Format

```
## Slop Check: [scope description]

### Findings (by severity)
- [HIGH] stub: src/auth.ts:45 — empty function body `validateToken()`
- [MED]  divergence: src/utils/format.ts — new file not related to task
- [MED]  over-engineering: src/api/client.ts:22 — factory pattern for single implementation

### Summary
- 3 findings (1 high, 2 medium)
- Categories hit: stubs, divergence, over-engineering

### Verdict: NEEDS WORK | CLEAN
```

## Fix Loop (Optional, Opt-In)

1. Sort findings by severity (HIGH first)
2. Present top finding
3. Fix it
4. Re-run Phase 1 + Phase 2 on updated diff
5. Report delta: fixed, new, remaining
6. Repeat until clean, user stops, or max iterations (default: 10)

**Constraint:** Fix loop only touches code related to findings. It does not introduce new changes beyond resolving the slop.

## Integration Points

- **Manual:** User invokes skill directly
- **Auto:** Pre-step in `verification-before-completion` and `requesting-code-review`
- **Self-check:** Claude runs on its own output before claiming "done"

## Configurable Defaults

| Setting | Default |
|---------|---------|
| Scope | Uncommitted changes |
| Confidence threshold | Medium+ |
| Fix loop | Off |
| Max fix iterations | 10 |

## Prior Art

- **desloppify** (github.com/peteromallet/desloppify): Heavy Python CLI, gaming-resistant scoring, 29 languages. Too heavy, not git-change-focused.
- **finding-duplicate-functions** (superpowers-lab): Semantic duplication detection. Absorbed into this skill's Tier 2.
- **code-review, code-simplifier, pr-review-toolkit**: General-purpose quality tools. This skill complements them with AI-slop-specific detection.
