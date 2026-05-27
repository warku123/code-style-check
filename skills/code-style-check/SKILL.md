---
name: code-style-check
description: >
  Code style self-review assistant. Checks semantic rules that Checkstyle cannot
  enforce (naming clarity, cross-file reuse, comment quality, test hygiene,
  refactoring hints, security/correctness, project conventions).
  Trigger phrases: "style check", "code style review", "/code-style-check".
---

# code-style-check

## Purpose

This Skill complements the Checkstyle hard gate (`checkStyleStrict.xml`) by reviewing the **semantic** rules that Checkstyle cannot mechanically enforce. Use it before pushing a PR or when reviewer wants a second opinion on style quality.

**What Checkstyle already covers** (do NOT duplicate):
- `catch (Exception/Throwable/RuntimeException)`
- Naming pattern (`TypeName`, `MethodName`, `ConstantName`, etc.)
- Spelling against built-in word list
- Trailing whitespace / newline at EOF
- TODO without owner
- Redundant / unused imports
- `System.out/err.print(ln)` and `.printStackTrace()`
- Line length > 100

**What this Skill covers** (7 categories below).

## Input Detection

| Input | Action |
|---|---|
| `git diff` output pasted by user | Review the diff |
| "check my current branch" / no explicit input | Run `git diff develop..HEAD` locally and review |
| No explicit input + local `develop` missing | Fallback: `git diff HEAD` (working tree vs HEAD) + list untracked files |
| Specific file path(s) | Review those files directly |

## Output Format

Group findings by file. For each:

```
**path/to/File.java:LINE**
[TAG] Problem description.
Suggestion or corrected snippet (if applicable).
```

Tags:

| Tag | Meaning | Blocks merge? |
|-----|---------|--------------|
| `[MUST]` | Violates project convention or introduces real risk — must fix | Yes (by reviewer) |
| `[SHOULD]` | Affects readability, maintainability, or consistency | Recommended |
| `[NIT]` | Minor preference — can be skipped | No |

**Output rules**:
- Be direct: state what, where, and how to fix — no lengthy preamble
- Group minor issues of the same type into a single comment
- Omit files with no issues
- End with one-line summary: `LGTM`, `LGTM with nits`, or `N [MUST] / M [SHOULD] findings`

## Review Checklist

Work through each category. Skip silently if nothing found.

### 1. Semantic Naming

Checkstyle enforces *pattern* (camelCase, UPPER_SNAKE_CASE). This Skill checks *meaning*.

- `[SHOULD]` Names that describe "what" but not "why": `data`, `temp`, `result`, `info`, `obj`, `val`, `num`, `str`, `list`, `map` — unless scope is extremely narrow (single expression)
- `[SHOULD]` Abbreviations ambiguous to non-authors: `addr` (address or adder?), `tx` (ok in TRON), `blk` (prefer `block`), `qty` (quantity or quality?)
- `[SHOULD]` Boolean without predicate prefix: `active` → `isActive`; `enable` → `isEnabled` / `shouldEnable`
- `[NIT]` Single-letter variables except loop indices / math formulas: `a`, `b`, `x` in business logic
- `[SHOULD]` Method name does not match what it actually does: `validate()` that also mutates, `getXxx()` with side effects
- `[SHOULD]` Same concept named differently across files: `blockId` vs `blockID` vs `blockNum` vs `blockNumber` — pick one, flag inconsistency

### 2. Cross-File Check

Checkstyle is single-file. This Skill simulates cross-file awareness using project knowledge.

- `[MUST]` Hardcoded constant already exists in `Constant.java` / `Args.java` / `Parameter.java`: new code should reference existing constant
- `[SHOULD]` Duplicated logic fragment (>5 lines, same semantics) in 2+ files — suggest extracting shared utility
- `[SHOULD]` Method renamed in one file but call sites in other files not updated (if diff shows rename)
- `[SHOULD]` New public method added without any caller in the diff — ask if it's premature abstraction or if callers are in another PR
- `[NIT]` Import from `*.internal.*` package into non-internal code — may indicate layer violation

### 3. Comment Quality

Checkstyle cannot judge comment *content*.

- `[SHOULD]` Comment states the obvious: `// increment i` — delete or replace with *why*
- `[SHOULD]` Comment contradicts code: comment says "thread-safe" but no synchronization visible
- `[MUST]` Comment contains stale TODO/FIXME without owner — already caught by Checkstyle, but verify *content* is still relevant
- `[SHOULD]` Complex algorithm (bit manipulation, Merkle tree logic, fee calculation) with zero comment — add brief "why this formula"
- `[SHOULD]` Javadoc `@param` / `@return` missing on public API methods
- `[NIT]` Comment in Chinese on English codebase, or vice versa — should match codebase primary language

### 4. Test Hygiene

- `[MUST]` Test has no assertions — just executes code and exits
- `[SHOULD]` Test only covers happy path — missing null, empty, boundary, overflow, exception branches
- `[SHOULD]` Flaky patterns: `Thread.sleep`, `System.currentTimeMillis()` comparisons, shared static mutable state between tests
- `[SHOULD]` Mock so deep that real logic is bypassed — verify the mock setup doesn't nullify the test purpose
- `[SHOULD]` Test name does not describe what's being verified: `test1()`, `testMethod()` — should be `shouldReject_whenBalanceInsufficient()`
- `[NIT]` `assertTrue(true)` or equivalent no-op assertions

### 5. Refactoring Suggestions

- `[SHOULD]` Method > 80 lines and does > 3 distinct things — consider extracting helpers
- `[SHOULD]` Nesting depth > 4 (if/try/for/synchronized) — flatten with early returns or extract method
- `[SHOULD]` Same switch/case logic duplicated in 2+ methods — polymorphism or strategy pattern
- `[NIT]` String concatenation in loop — `StringBuilder`
- `[NIT]` Collection pre-sizing missed: `new ArrayList<>()` in a known-size loop — use `new ArrayList<>(size)`

### 6. Security / Correctness

These overlap with `tron-pr-review` but framed as *style* (pattern) rather than *bug* (exploitability).

- `[MUST]` `equals()` not symmetric: compares subclass field without `getClass()` check or `instanceof` guard
- `[SHOULD]` Defensive copy missing: returns internal mutable collection/array directly — caller can mutate internal state
- `[SHOULD]` Lock acquisition without `try/finally`: `lock.lock()` followed by operations that may throw
- `[SHOULD]` Resource not closed in all paths: `InputStream`, `ResultSet`, `Iterator` from DB — use try-with-resources
- `[SHOULD]` Catch-and-ignore: `catch (SomeException e) { /* empty */ }` without comment — even if not `Exception`/`Throwable`, empty catch is suspicious

### 7. TRON Project Conventions

Project-specific patterns learned from codebase history.

- `[SHOULD]` DB key naming inconsistent with existing pattern: `DB_KEY_XXX` vs `KEY_DB_XXX`
- `[SHOULD]` Proto field accessed by index (`getField(3)`) instead of getter (`getFieldName()`) — prefer typed getter
- `[SHOULD]` Energy/fee calculation without `Math.addExact` / `multiplyExact` — TRON uses strict math to prevent overflow
- `[SHOULD]` Actuator pattern violation: `validate()` should be pure check, `execute()` should assume validation passed
- `[NIT]` Logger pattern: prefer `logger.info("{} {} {}", a, b, c)` over `logger.info(a + " " + b + " " + c)`

## Suppression

If author disagrees with a finding, reply with `// SKILL:OFF <category> — reason: <one-line>` on the line above. Reviewer decides whether to accept.
