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

### Step 1.5: Mechanical Pattern Scan (MANDATORY - Before Subjective Review)

Run these grep patterns on ALL .rs files FIRST. Every match is a CANDIDATE finding.
Only clear a candidate after EXPLICIT verification that it's intentional.

**Universal Checks (always run):**
```bash
# API naming
rg 'fn get_[a-z]'           # Getter prefix - should be fn name() not fn get_name()
rg 'fn set_[a-z]'           # Setter prefix - consider builder pattern or direct field

# Error handling
rg '\.unwrap\(\)'           # Potential panic - verify intentional or use ? / expect
rg '\.expect\("'            # Check message is descriptive
rg 'panic!\('               # Verify this is truly unrecoverable

# Cloning
rg '\.clone\(\)'            # Verify necessary - often a design smell

# Visibility
rg 'pub fn'                 # Check if truly needs to be public

# Suboptimal types
rg 'fn.*&String'            # Should be &str
rg 'fn.*&Vec<'              # Should be &[T]
rg '&Box<'                  # Usually just &T or &dyn Trait

# Unsafe
rg 'unsafe \{'              # Verify SAFETY comment exists above
rg 'unsafe fn'              # Verify /// # Safety doc exists

# Deprecated patterns  
rg 'mem::uninitialized'     # Use MaybeUninit
rg 'mem::zeroed'            # Usually want MaybeUninit
rg 'static mut'             # Use atomics or Mutex instead
```

**Domain-Specific Checks (detect and run if applicable):**

If web/axum/actix detected:
```bash
rg 'Rc<'                    # Should be Arc for thread safety
rg '\.lock\(\).*\.await'    # Guard held across await point
```

If fintech/money/trading detected:
```bash
rg ': f64'                  # Never use float for money - use rust_decimal
rg ': f32'                  # Never use float for money - use rust_decimal  
rg 'as f64'                 # Float conversion in financial context
```

If embedded/no_std detected:
```bash
rg 'Vec<'                   # Should use heapless::Vec
rg 'String'                 # Should use heapless::String
rg 'Box<'                   # No heap allocation in no_std
```

If CLI detected:
```bash
rg 'println!\(.*[Ee]rror'   # Errors should go to stderr (eprintln!)
rg 'println!\(.*[Ff]ailed'  # Errors should go to stderr (eprintln!)
```

**Recording candidates:**
For each grep match, add to findings file with status "Candidate" before review.
During Step 2, change status to either "Confirmed" (it's a smell) or "Cleared" (intentional/false positive with reason).

### Step 2: Review Loop (Automatic)
```
For each file in todos:
  1. Read the file
  2. Review candidates from Step 1.5 that match this file - confirm or clear each
  3. Check against smell checklist below for additional issues
  4. Update docs/code-review-findings.md immediately (don't accumulate)
  5. Mark todo complete
  6. Every 5 files: update Summary section in findings file with progress
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

## Valid Exceptions (Clear These Candidates)

Some grep matches are NOT smells. Document reason when clearing:

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
Status: In Progress

## Summary
- Files reviewed: 0/N
- Candidates from pattern scan: N
- Confirmed findings: High: 0, Medium: 0, Low: 0
- Cleared (false positives): 0

## Pattern Scan Candidates

| # | File | Line | Pattern | Status | Notes |
|---|------|------|---------|--------|-------|
<!-- Status: Candidate → Confirmed/Cleared -->

## Confirmed Findings

| # | File | Line | Smell | Severity | Suggested Fix |
|---|------|------|-------|----------|---------------|

## Cleared Candidates

| # | File | Line | Pattern | Reason |
|---|------|------|---------|--------|
<!-- Document WHY each was cleared to prevent re-flagging -->

## Notes

[Any cross-cutting observations]
```

---

## Severity Guide

- **High**: Will cause bugs OR makes bugs hard to find OR blocks future changes
- **Medium**: Makes code harder to change safely
- **Low**: Makes code harder to understand
