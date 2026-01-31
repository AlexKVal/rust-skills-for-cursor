---
name: rust-skills
description: Meta-cognition framework for Rust development. Use for Rust questions involving errors, design, ownership, async, traits, or domain-specific code (fintech, web, CLI, embedded).
---

# Rust Meta-Cognition Framework

## Core Principle

**Trace through cognitive layers before answering.**

```
Layer 3: Domain Constraints (WHY) - Business rules, requirements
Layer 2: Design Choices (WHAT)   - Architecture, patterns
Layer 1: Language Mechanics (HOW) - Ownership, traits, lifetimes
```

## Routing

| Signal | Start | Direction |
|--------|-------|-----------|
| E0xxx / compile error | L1 | UP ↑ to find root cause in design/domain |
| "How to design..." | L2 | Check L3 context, then DOWN ↓ |
| Building [domain] app | L3 | DOWN ↓ through design to mechanics |
| Performance issue | L1 | UP ↑ to L2 design, then back DOWN |

## Error Resolution Strategy

Don't just fix the error mechanically. Ask:
- **L1**: What's the mechanical issue? (moved value, borrow conflict, trait bound)
- **L2**: Does the data structure/API design cause this?
- **L3**: What domain requirement should guide the solution?

Example: E0382 in trading system → Don't suggest `.clone()`. Trace UP: trading requires audit trails → immutable shared data → `Arc<T>` is the domain-appropriate solution.

## Domain Awareness

When domain context is present, load domain constraints:

| Domain | Consider |
|--------|----------|
| Fintech | Immutability for audit, decimal precision, `Arc<T>` over clone |
| Web API | Handler cloning, lifetimes across await, concurrent access |
| CLI | User-facing errors (`anyhow`), input validation |
| Embedded | No heap, `no_std`, stack allocation, const generics |

## Deep Reference

For detailed patterns, see the skills/ directory:

- `skills/m01-ownership/` - ownership patterns and common errors
- `skills/m06-error-handling/` - thiserror vs anyhow, error strategies
- `skills/m07-concurrency/` - async patterns, Send/Sync
- `skills/unsafe-checker/` - unsafe code review
- `skills/code-review/` - code smell analysis and refactoring
- `skills/domain-*/` - domain-specific constraints
