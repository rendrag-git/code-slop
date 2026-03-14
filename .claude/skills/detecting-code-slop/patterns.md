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

Check EVERY pattern below. Do not skip any.

### Scope Divergence
- New files/functions unrelated to the stated task
- Refactoring code that wasn't asked about
- Adding features beyond what was requested
- "Improving" unrelated utilities or formatting
- Compare the diff against the original task description — flag anything not directly required

### Semantic Duplication
- Two+ functions with different names doing the same thing
- Same logic implemented with different variable names or slight regex differences
- One function imported, the duplicate unused (dead code on arrival)

### Over-Engineering
- Abstractions with only one caller
- Config/options for one-time operations
- Factory patterns for single implementations
- Interface with 10+ props when 3 are used
- Separate module for what could be an inline helper

### Abandoned Branches
- Error handling for cases that can't happen given the data flow
- if/else paths that are unreachable from any call site
- `loading` or `disabled` states that no caller ever triggers
- Catch blocks that handle exceptions the try block can't throw

### Premature Generalization
- Function parameters nobody passes
- Options/config nobody uses
- Multiple variants (5 sizes, 5 colors) when 1 is used
- "Just in case" flexibility with zero current consumers

### Missing Guardrails
- No null/undefined checks on external input
- No input validation on user-facing functions
- Non-null assertions (`!`) hiding missing runtime checks
- No error handling on async operations
- Happy-path-only code that breaks on edge cases

### Over-Specification
- Rigid narrow solution when a general approach would be simpler
- Hardcoded values that should be parameters (locale strings, magic numbers)
- Single-use-case implementations that resist any reuse

### Wrong-Context Patterns
- Python syntax in TypeScript files (`pass`, `NotImplementedError`)
- Class with only static methods in a functional codebase
- Patterns from a different framework (React patterns in Vue, etc.)
- Technically valid but non-idiomatic for the local codebase style
