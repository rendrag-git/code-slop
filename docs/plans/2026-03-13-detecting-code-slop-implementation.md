# detecting-code-slop Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a lightweight Claude Code skill that detects AI-generated code slop in git changes — stubs, duplication, over-engineering, sycophantic comments, abandoned branches, copy-paste artifacts, and scope divergence.

**Architecture:** Two-phase detection (mechanical grep scan + LLM review) with configurable git scope, unified report output, optional fix loop. Single SKILL.md file at `~/.claude/skills/detecting-code-slop/SKILL.md`.

**Tech Stack:** Claude Code skill (markdown), git, grep. No external dependencies.

---

### Task 1: Create Skill Directory

**Files:**
- Create: `~/.claude/skills/detecting-code-slop/SKILL.md` (placeholder)

**Step 1: Create directory**

```bash
mkdir -p ~/.claude/skills/detecting-code-slop
```

**Step 2: Create minimal placeholder**

Create `~/.claude/skills/detecting-code-slop/SKILL.md` with just the frontmatter:

```markdown
---
name: detecting-code-slop
description: Use when reviewing AI-generated code changes for quality issues — stubs, duplication, over-engineering, sycophantic comments, abandoned branches, copy-paste artifacts, and scope divergence. Triggers manually or as a self-check after implementation.
---

# Detecting Code Slop

(placeholder — will be replaced after baseline testing)
```

**Step 3: Commit**

```bash
cd ~/.claude/skills
git init detecting-code-slop 2>/dev/null || true
cd detecting-code-slop
git add SKILL.md
git commit -m "chore: scaffold detecting-code-slop skill"
```

---

### Task 2: RED — Baseline Testing Without Skill

Per the writing-skills TDD process: run pressure scenarios WITHOUT the skill content to document what Claude does naturally. This establishes the baseline.

**Step 1: Design 3 pressure scenarios**

Create test prompts that contain obvious slop. Each scenario should include multiple slop patterns to see which ones Claude catches naturally vs misses.

**Scenario A — Stub-heavy code:**
Give Claude a diff containing TODO stubs, empty function bodies, and NotImplementedError. Ask it to review the changes. Document: does it flag these?

**Scenario B — Over-engineered + scope divergence:**
Give Claude a diff where the task was "add a login button" but the changes include a new utility library, an abstraction layer, and refactored unrelated files. Ask it to review. Document: does it notice the divergence?

**Scenario C — Sycophantic comments + semantic duplication:**
Give Claude a diff with comments like `// This function validates the user` on a function called `validateUser`, plus two functions that do the same thing with different names. Ask it to review. Document: does it catch both patterns?

**Step 2: Run each scenario as a subagent**

For each scenario, dispatch a subagent with:
- The scenario diff
- "Review this code for quality issues"
- NO reference to the detecting-code-slop skill

**Step 3: Document baseline results**

Record verbatim:
- Which patterns each scenario caught
- Which patterns each scenario missed
- What rationalizations were used (if any)
- What language was used in the findings

Save results to `docs/plans/2026-03-13-baseline-results.md`.

**Step 4: Commit baseline**

```bash
git add docs/plans/2026-03-13-baseline-results.md
git commit -m "test: document baseline slop detection without skill"
```

---

### Task 3: GREEN — Write the Skill

Based on baseline gaps, write `SKILL.md` that addresses the specific patterns Claude missed.

**Files:**
- Modify: `~/.claude/skills/detecting-code-slop/SKILL.md`

**Step 1: Write the frontmatter**

```yaml
---
name: detecting-code-slop
description: Use when reviewing AI-generated code changes for quality issues — stubs, duplication, over-engineering, sycophantic comments, abandoned branches, copy-paste artifacts, and scope divergence. Triggers manually or as a self-check after implementation.
---
```

**Step 2: Write the Overview section**

- What is code slop (1-2 sentences)
- Core principle: two-phase detection (mechanical + LLM)
- When to use: after implementation, before commits/PRs, as self-check

**Step 3: Write the Scope Configuration section**

Table of git diff options with defaults. Include exact commands:
- `git diff` (default — uncommitted)
- `git diff --cached` (staged only)
- `git diff main...HEAD` (branch diff)
- Custom files/commit range

**Step 4: Write the Tier 1 Mechanical Scan section**

For each pattern, provide exact grep/regex patterns to run against the diff:

| Pattern | Grep commands |
|---------|--------------|
| Stubs & placeholders | `grep -n 'TODO\|FIXME\|NotImplementedError\|throw new Error.*not implemented'` + empty body detection |
| Sycophantic comments | `grep -n '// This function\|// This method\|# This function'` + docstring-to-function-name ratio |
| Copy-paste artifacts | Fuzzy line matching for 3+ similar blocks |
| Commented-out code | `grep -n '^+.*//.*[{(;]$\|^+.*#.*def \|^+.*#.*class '` |

**Step 5: Write the Tier 2 LLM Review section**

For each pattern, provide:
- What to look for (specific signals)
- How to evaluate (comparison against task context)
- Confidence calibration (when HIGH vs MED vs LOW)

Patterns: scope divergence, semantic duplication, over-engineering, abandoned branches, premature generalization.

**Step 6: Write the Output Format section**

Exact report template with findings by severity, summary, and verdict.

**Step 7: Write the Fix Loop section**

Optional opt-in flow: sort findings → fix top → rescan → report delta → repeat. Include the constraint that fixes must not introduce new slop.

**Step 8: Write the Integration Points section**

How to trigger: manual, auto (verification-before-completion, requesting-code-review), self-check.

**Step 9: Write the Red Flags / Common Mistakes section**

Based on baseline testing: what Claude tends to miss or rationalize away.

**Step 10: Commit**

```bash
cd ~/.claude/skills/detecting-code-slop
git add SKILL.md
git commit -m "feat: write detecting-code-slop skill"
```

---

### Task 4: GREEN — Test With Skill Present

**Step 1: Re-run the same 3 pressure scenarios from Task 2**

This time, the skill is present. Each subagent should have access to the skill.

**Step 2: Compare results to baseline**

For each scenario, document:
- Patterns now caught that were missed before
- Any patterns still missed
- Quality of the report format
- Whether the two-phase flow was followed

**Step 3: Determine pass/fail**

GREEN passes if: all 6 core patterns are caught across the 3 scenarios, report format matches spec, two-phase flow is followed.

**Step 4: Document results**

Save to `docs/plans/2026-03-13-green-test-results.md`.

**Step 5: Commit**

```bash
git add docs/plans/2026-03-13-green-test-results.md
git commit -m "test: verify skill catches slop patterns (GREEN)"
```

---

### Task 5: REFACTOR — Close Loopholes

**Step 1: Identify gaps from GREEN testing**

Review Task 4 results. For each pattern still missed or inconsistently caught:
- What rationalization did Claude use?
- What wording in the skill allowed the loophole?

**Step 2: Update SKILL.md to close loopholes**

Add explicit counters for each rationalization found. Add to red flags list. Tighten ambiguous language.

**Step 3: Re-test with updated skill**

Run scenarios again. Repeat until all patterns are reliably caught.

**Step 4: Commit**

```bash
cd ~/.claude/skills/detecting-code-slop
git add SKILL.md
git commit -m "refactor: close loopholes from testing"
```

---

### Task 6: Final Verification & Cleanup

**Step 1: Word count check**

```bash
wc -w ~/.claude/skills/detecting-code-slop/SKILL.md
```

Target: under 500 words (per writing-skills guidelines for non-frequently-loaded skills).

**Step 2: CSO verification**

- Description starts with "Use when..."
- No workflow summary in description
- Keywords cover: slop, stubs, duplication, over-engineering, comments, divergence, TODO, placeholder
- Name uses hyphens only

**Step 3: Cross-reference check**

Verify any references to other skills use `skill-name` format, not `@` links.

**Step 4: Final commit**

```bash
cd ~/.claude/skills/detecting-code-slop
git add SKILL.md
git commit -m "docs: finalize detecting-code-slop skill"
```
