# Baseline Testing Results — detecting-code-slop

## Summary

Claude naturally catches most individual slop patterns without a skill. The skill's value is in enforcing **structure, systematic coverage, and process** — not teaching pattern recognition.

## Scenario A: Stub-Heavy Auth Code

**Patterns caught:**
- Stubs & placeholders (TODOs, empty bodies, NotImplementedError) — flagged as "too many unimplemented functions"
- Sycophantic comments — flagged as "comments restate the function name"
- Missing guardrails — flagged no input validation, JWT secret issue
- Security holes — flagged validateSession returning true, error message info leakage
- Python syntax in TS (`pass;`) — caught as compile error

**Patterns missed:**
- No structured severity categorization
- No verdict/pass-fail
- No categorization by slop type

## Scenario B: Over-Engineering + Scope Divergence

**Patterns caught:**
- Scope divergence — flagged as "scope creep, unrelated changes" (stringHelpers, formatDate)
- Over-engineering — flagged Button as "speculative generality"
- Dead code — flagged stringHelpers as unused
- Missing import — flagged Spinner not imported
- Wrong-context pattern — flagged hardcoded en-US locale

**Patterns missed:**
- Didn't flag Button's sycophantic docstring
- No structured output format
- No systematic checklist approach

## Scenario C: Sycophantic Comments + Semantic Duplication

**Patterns caught:**
- Semantic duplication — flagged two email validators doing the same thing
- Sycophantic comments — flagged as "noise comments" and "bloated JSDoc"
- Missing tests — flagged no unit tests for validation logic

**Patterns missed:**
- No structured output format
- No fix loop suggestion

## Key Insight

The skill does NOT need to teach Claude what slop looks like — it already recognizes these patterns. The skill needs to enforce:

1. **Structured output** — severity levels (HIGH/MED/LOW), categories, summary, verdict
2. **Systematic coverage** — check every category, not just the obvious ones
3. **Two-phase process** — mechanical scan first (grep/regex), then LLM judgment
4. **Fix loop** — actionable remediation cycle
5. **Self-triggering** — integration with verification/review workflows
6. **Slop framing** — name the patterns explicitly so they're trackable across reviews
