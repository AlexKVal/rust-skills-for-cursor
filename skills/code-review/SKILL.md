---
name: rust-code-review
description: Code smell analysis for Rust. Triggers: code smell, code review, refactor, cleanup, technical debt.
---

# Rust Code Review

## CRITICAL: Execute This Process Automatically

**Do NOT wait for user prompts between steps. Execute the full process:**

### Step 1: Initialize (Do Immediately)
```
1. Create docs/code-review-findings.md with template below
2. Glob for all *.rs files in crates/
3. Create TodoWrite with all files as pending
4. Tell user: "Starting review of N files. Findings will be saved to docs/code-review-findings.md"
```

### Step 2: Review Loop (Automatic)
```
For each file in todos:
  1. Read the file
  2. Check against smell checklist below
  3. If findings: append to docs/code-review-findings.md immediately (don't accumulate)
  4. Mark todo complete
  5. Every 5 files: update Summary section in findings file with progress
```

### Step 3: Finalize
```
1. Update findings file: Status = Complete, final counts
2. Present summary to user
3. Ask: "Want me to fix any of these? (specify by number or 'all High')"
```

### If Session Resumes
```
1. Read docs/code-review-findings.md first
2. Check which files already reviewed
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

---

## Findings File Template

Create this at `docs/code-review-findings.md`:

```markdown
# Code Review Findings

Generated: [DATE]
Status: In Progress

## Summary
- Files reviewed: 0/N
- Findings: High: 0, Medium: 0, Low: 0

## Findings

| # | File | Line | Smell | Severity | Suggested Fix |
|---|------|------|-------|----------|---------------|

## Notes

[Any cross-cutting observations]
```

---

## Severity Guide

- **High**: Will cause bugs OR makes bugs hard to find OR blocks future changes
- **Medium**: Makes code harder to change safely
- **Low**: Makes code harder to understand
