# Slop Pattern Reference

## Tier 1: Mechanical Patterns (grep/regex)

Scan the diff for these. Record every hit as `file:line — matched text`.

### Stubs & Placeholders

```bash
grep -nE 'TODO|FIXME|HACK|XXX'
grep -nE 'NotImplementedError|throw new Error\(.(not implemented|todo)'
grep -nE 'raise NotImplementedError'
```

Also flag:
- `pass` in non-`__init__` methods (Python stub)
- Empty function bodies `{ }` or `=> {}`
- Functions that only `return true`, `return null`, or `return undefined` with no logic
- `// Will implement later`, `// implement this`, `// placeholder`

### Sycophantic Comments

```bash
grep -nE '^\+\s*(//|#|/\*\*?|\*)\s*(This (function|method|class|module|component|hook|helper|utility|service) (does|is|will|handles|provides|ensures|represents|manages|creates|returns|takes|accepts|processes|performs))'
```

Also flag:
- Docstrings that restate the function name verbatim
- `@param` tags that just restate the parameter name ("@param email - The email")
- Module-level docstrings that are pure filler ("provides comprehensive X functionality for the application")
- Comments that narrate the next line (`// Set the value` above `setValue(x)`)

### Copy-Paste Artifacts

Look for 3+ added blocks sharing >80% token similarity within the diff. Common signals:
- Repeated function bodies with minor name/variable differences
- Identical error handling blocks across multiple functions
- Same structural pattern (comment + throw/return) repeated 3+ times

### Commented-Out Code

```bash
grep -nE '^\+\s*(//|#)\s*(if |for |while |return |const |let |var |def |class |import |export |function |async )'
```

Flag blocks of commented real code, NOT explanatory comments. A comment is "commented-out code" if uncommenting it would produce valid syntax.

## Tier 2: LLM Review Patterns (judgment required)

Check EVERY pattern below. Do not skip any. Use the **Calibration** notes to adjust severity based on context.

### Scope Divergence
- New files/functions unrelated to the stated task
- Refactoring code that wasn't asked about
- Adding features beyond what was requested
- "Improving" unrelated utilities or formatting
- Compare the diff against the original task description — flag anything not directly required

**Calibration:**
- HIGH in all contexts — scope divergence is always a problem
- Exception: if the task explicitly says "and clean up related code," adjacent changes are acceptable
- Tiny formatting fixes in lines already being edited (e.g., trailing whitespace) → LOW

### Semantic Duplication
- Two+ functions with different names doing the same thing
- Same logic implemented with different variable names or slight regex differences
- One function imported, the duplicate unused (dead code on arrival)

**Calibration:**
- HIGH when duplicates are in the same file or module — no excuse
- MED when across packages/services — may reflect intentional decoupling
- If code is duplicated across test files, downgrade to LOW — test readability sometimes justifies repetition

### Over-Engineering
- Abstractions with only one caller
- Config/options for one-time operations
- Factory patterns for single implementations
- Interface with 10+ props when 3 are used
- Separate module for what could be an inline helper

**Calibration:**
- HIGH in scripts, CLIs, one-off tools, glue code — simplicity is the point
- MED in application code — some abstraction is normal but question single-caller abstractions
- LOW in libraries/frameworks/SDKs — extensibility is expected; factory patterns and broad interfaces may be intentional
- If the abstraction matches a pattern used elsewhere in the same codebase, downgrade one level

### Abandoned Branches
- Error handling for cases that can't happen given the data flow
- if/else paths that are unreachable from any call site
- `loading` or `disabled` states that no caller ever triggers
- Catch blocks that handle exceptions the try block can't throw

**Calibration:**
- HIGH when the dead branch adds significant complexity (10+ lines of unreachable logic)
- MED for small dead branches (simple else clause, short catch block)
- LOW at system boundaries (API handlers, CLI entry points) — defensive error handling is reasonable even if current callers can't trigger it

### Premature Generalization
- Function parameters nobody passes
- Options/config nobody uses
- Multiple variants (5 sizes, 5 colors) when 1 is used
- "Just in case" flexibility with zero current consumers

**Calibration:**
- HIGH in application code — YAGNI applies; unused params and options are noise
- MED in shared libraries — some forward-thinking is acceptable if the generalization is small (one extra param)
- LOW in public APIs/SDKs — broader interfaces are conventional to avoid breaking changes later
- If the generalization adds more than 20% code volume for zero current use, escalate one level regardless of context

### Missing Guardrails
- No null/undefined checks on external input
- No input validation on user-facing functions
- Non-null assertions (`!`) hiding missing runtime checks
- No error handling on async operations
- Happy-path-only code that breaks on edge cases

**Calibration:**
- HIGH at trust boundaries: user input, API request handlers, file I/O, third-party data — always validate
- MED for internal functions called with data that was already validated upstream — check that upstream validation actually exists
- LOW for pure internal helpers where callers are fully controlled and visible in the same module
- Never flag missing validation on internal code where the type system already guarantees correctness

### Over-Specification
- Rigid narrow solution when a general approach would be simpler
- Hardcoded values that should be parameters (locale strings, magic numbers)
- Single-use-case implementations that resist any reuse

**Calibration:**
- HIGH when hardcoded values will obviously need to change (locale strings, environment-specific URLs, credentials)
- MED for magic numbers that are domain constants — could go either way
- LOW in test code — hardcoded expected values in tests are normal and desirable
- LOW in prototypes/spikes explicitly marked as throwaway

### Wrong-Context Patterns
- Python syntax in TypeScript files (`pass`, `NotImplementedError`)
- Class with only static methods in a functional codebase
- Patterns from a different framework (React patterns in Vue, etc.)
- Technically valid but non-idiomatic for the local codebase style

**Calibration:**
- HIGH for cross-language syntax errors (Python in JS, etc.) — this is always wrong
- HIGH for wrong-framework patterns (React hooks in Vue) — actively confusing
- MED for style mismatches (classes in a functional codebase) — valid but smells
- LOW if the "wrong" pattern is used elsewhere in the codebase — may be an intentional local convention
