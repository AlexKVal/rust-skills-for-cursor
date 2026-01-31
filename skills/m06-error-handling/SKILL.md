---
name: m06-error-handling
description: "Use for error handling. Triggers: Result, Option, Error, unwrap, expect, panic, anyhow, thiserror, when to panic vs return Result"
user-invocable: false
---

# Error Handling

> **Layer 1: Language Mechanics**

## Core Question

**Is this failure expected or a bug?**

## Error → Design Question

| Pattern | Don't Just Say | Ask Instead |
|---------|----------------|-------------|
| unwrap panics | "Use ?" | Is None/Err actually possible here? |
| Type mismatch on ? | "Use anyhow" | Are error types designed correctly? |
| Lost error context | "Add .context()" | What does the caller need to know? |

## Thinking Prompt

1. **What kind of failure?**
   - Expected → Result<T, E>
   - Absence normal → Option<T>
   - Bug/invariant → panic!

2. **Who handles this?**
   - Caller → propagate with ?
   - User → friendly message
   - Programmer → panic with message

## Library vs Application

| Context | Error Crate | Why |
|---------|-------------|-----|
| Library | `thiserror` | Typed errors for consumers |
| Application | `anyhow` | Ergonomic error handling |
| Mixed | Both | thiserror at boundaries, anyhow internally |

## Trace Up ↑

- Too many unwraps → m09-domain (data model right?)
- Error context design → m13-domain-error (recovery needs?)
- Library vs app errors → m11-ecosystem (who are consumers?)
