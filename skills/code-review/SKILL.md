---
name: rust-code-review
description: Code smell analysis for Rust. Triggers: code smell, code review, refactor, cleanup, technical debt.
---

# Rust Code Review

## CRITICAL: Fully Autonomous Execution

**Run the ENTIRE process without stopping for user input. No questions, no confirmations.**

The user will see results in:
- Git commit diff (what changed)
- `docs/code-review-findings.md` (details if they want them)

### Step 1: Initialize (Do Immediately)
```
1. Create docs/code-review-findings.md with template below
2. Glob for all *.rs files in crates/ (or src/ if no crates/)
3. Create TodoWrite with all files as pending
4. Brief status: "Reviewing N files..."
```

### Step 2: Review Loop (Automatic)
```
For each file in todos:
  1. Read the file
  2. Check against smell checklist below
  3. Update docs/code-review-findings.md immediately (don't accumulate)
  4. Mark todo complete
  5. Every 5 files: update Summary section in findings file with progress
```

### Step 3: Auto-Fix (No User Prompt)
```
Automatically fix all High and Medium severity issues:
1. For each confirmed High/Medium finding:
   - Apply the fix
   - Update findings file: mark as "Fixed"
2. Skip Low severity (not worth the tokens unless trivial)
3. Exception: Skip if fix requires architectural decisions or user input
```

**What to fix automatically:**
- Getter naming (`get_x` → `x`)
- `&String` → `&str`, `&Vec<T>` → `&[T]`
- Missing `#[must_use]` on pure functions
- Unnecessary `.clone()` when reference works
- `unwrap()` → `expect("reason")` or `?`
- Visibility reduction (`pub` → `pub(crate)` when internal)

**What to skip (even if High/Medium):**
- Structural refactors (Feature Envy, Extract Method) - too invasive
- Error type redesigns - needs user decision
- Anything that changes public API signatures

### Step 4: Verify
```
1. Run: cargo check
2. Run: cargo test
3. Run: cargo clippy (if available)
4. If any fail: revert problematic fix, update findings file
```

### Step 5: Commit & Report
```
1. Update findings file: Status = Complete, final counts
2. Commit all changes with message summarizing fixes
3. Brief final message (2-3 lines max):
   "Code review complete. Fixed N issues (H high, M medium). 
    L low-priority items noted in docs/code-review-findings.md.
    Tests pass."
```

### If Session Resumes
```
1. Read docs/code-review-findings.md first
2. Check which files already reviewed/fixed
3. Continue from where left off
```

---

## Rust-Specific Smells Checklist

### Ownership & Borrowing
| Smell | Example | Fix |
|-------|---------|-----|
| Unnecessary `.clone()` | `foo.clone()` when `&foo` works | Use reference or restructure |
| Clone to satisfy borrow checker | Clone inside loop to avoid borrow | Restructure ownership, use indices |
| `RefCell` for convenience | `RefCell` when redesign would work | Split struct or change API |

### Error Handling
| Smell | Example | Fix |
|-------|---------|-----|
| Bare `unwrap()` in lib code | `file.read().unwrap()` | Use `?` or `expect("reason")` |
| `unwrap()` on user input | `args[1].parse().unwrap()` | Return Result, use `?` |
| Missing error context | `.map_err(Error::Io)?` | Add `.context("what failed")` |

### API Design
| Smell | Example | Fix |
|-------|---------|-----|
| `get_` prefix on getter | `fn get_name(&self)` | `fn name(&self)` |
| Wrong mut suffix | `fn get_name_mut(&mut self)` | `fn name_mut(&mut self)` |
| Broad visibility | `pub fn` that's only used in crate | `pub(crate) fn` or private |
| Missing `#[must_use]` | `fn compute() -> Result<T>` | Add `#[must_use]` |
| `&String` / `&Vec<T>` params | `fn foo(s: &String)` | `fn foo(s: &str)` / `&[T]` |

### Structural
| Smell | Example | Fix |
|-------|---------|-----|
| Feature Envy | Method mostly uses other struct's fields | Move method to that struct |
| Tell Don't Ask | `if x.state() { x.do_a() } else { x.do_b() }` | `x.do_appropriate()` |
| Middle Man | Struct that only delegates | Inline or remove |
| Data Clump | Same 3 fields always together | Extract to struct |

### Duplication
| Smell | Example | Fix |
|-------|---------|-----|
| Copy-paste blocks | Same 5+ lines in two places | Extract function |
| Similar match arms | Match with near-identical arms | Consolidate or extract |
| Repeated error creation | Same `Error::new(...)` pattern | Helper function |

---

## Skip These

- rustfmt style issues
- clippy lints (run `cargo clippy` separately)  
- Intentional patterns with explanatory comments
- Test code duplication (often acceptable for clarity)
- Single-use code that's clear as-is

## Valid Exceptions

Some patterns are NOT smells. Skip these during review:

| Pattern | Valid When | Example |
|---------|------------|---------|
| `fn get_` | Follows std convention | `get_mut()` like `HashMap::get_mut` |
| `fn get_` | Fallible retrieval | `get_or_insert()`, `get_unchecked()` |
| `.clone()` | Required by ownership | Cloning into spawned task |
| `.clone()` | Intentional copy | Implementing `Clone` trait |
| `.unwrap()` | Proven infallible | `"literal".parse::<i32>().unwrap()` |
| `.unwrap()` | Test code | Tests can panic on failure |
| `unsafe` | Has SAFETY comment | Properly documented unsafe |
| `pub fn` | Part of public API | Crate's exported interface |
| `&String` | Rare: need String methods | `.capacity()`, etc. |
| `panic!` | Invariant violation | Logic bug, not user error |

---

## Findings File Template

Create this at `docs/code-review-findings.md`:

```markdown
# Code Review Findings

Generated: [DATE]
Status: In Progress | Complete

## Summary
- Files reviewed: 0/N
- Fixed: High: 0, Medium: 0
- Skipped (Low): 0

## Findings

| # | File | Line | Smell | Severity | Status | Notes |
|---|------|------|-------|----------|--------|-------|
<!-- Status: Fixed | Skipped -->

## Notes

[Any cross-cutting observations]
```

---

## Severity Guide

- **High**: Will cause bugs, makes bugs hard to find, or blocks future work
- **Medium**: Makes code harder to change safely
- **Low**: Makes code harder to understand

---

## AI-Optimized Priorities

This codebase is maintained by AI agents. Prioritize fixes that help future agents:

| Priority | Why It Helps Agents |
|----------|---------------------|
| Consistent naming | Pattern matching, predictable APIs |
| Explicit types | Less inference needed, clearer context |
| Small functions | Fit in context window, easier to modify |
| Good error messages | Faster debugging when things fail |
| `#[must_use]` | Compiler catches forgotten returns |

**Don't optimize for:**
- Human readability concerns that don't affect agents
- Documentation prose (agents read code directly)
- Aesthetic preferences with no functional impact
