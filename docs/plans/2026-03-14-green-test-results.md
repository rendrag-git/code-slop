# GREEN Test Results — detecting-code-slop

## Summary

The skill dramatically improves output structure, systematic coverage, and consistency compared to baseline. All core slop categories were caught across the 3 scenarios.

## Comparison: Baseline vs With Skill

| Aspect | Baseline | With Skill |
|--------|----------|------------|
| Output format | Free-form prose | Structured: severity, summary, verdict |
| Two-phase process | No | Yes — Phase 1 then Phase 2 |
| Category labeling | Informal | Named slop categories consistently |
| Systematic coverage | Caught obvious, missed subtle | Checked every category |
| Severity levels | None | HIGH/MED/LOW |
| Verdict | None | NEEDS WORK / CLEAN |
| File:line references | Sometimes | Consistently |

## Categories Caught

All 10 core categories were caught across the 3 scenarios:
- Stubs & placeholders, Sycophantic comments, Copy-paste artifacts (Tier 1)
- Scope divergence, Semantic duplication, Over-engineering, Abandoned branches, Premature generalization, Missing guardrails, Wrong-context patterns (Tier 2)

Not tested (no scenario had these): Commented-out code, Over-specification

## Minor Gaps for REFACTOR

1. **Phase 1 not always literal grep** — some agents did mental pattern matching instead of running actual grep commands. Skill should clarify whether Phase 1 must be actual shell commands or can be pattern-matched by reading the diff.
2. **Extra output sections** — Scenario C added "Recommended fixes" not in spec. Harmless but inconsistent. Could standardize by making it optional.
3. **Commented-out code and Over-specification** — not tested. Could add a scenario that includes these patterns.
