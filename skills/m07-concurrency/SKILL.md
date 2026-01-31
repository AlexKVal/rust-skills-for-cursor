---
name: m07-concurrency
description: "Use for concurrency/async. Triggers: E0277 Send Sync, cannot be sent between threads, thread, spawn, Mutex, async, await, Future, tokio, deadlock"
user-invocable: false
---

# Concurrency

> **Layer 1: Language Mechanics**

## Core Question

**Is this CPU-bound or I/O-bound, and what's the sharing model?**

## Error → Design Question

| Error | Don't Just Say | Ask Instead |
|-------|----------------|-------------|
| E0277 Send | "Add Send bound" | Should this type cross threads? |
| E0277 Sync | "Wrap in Mutex" | Is shared access really needed? |
| Future not Send | "Use spawn_local" | Is async the right choice? |

## Trace Up ↑ (MANDATORY for Send/Sync errors)

**Detect domain first:**

| Context Keywords | Load | Key Constraint |
|-----------------|------|----------------|
| Web API, axum, handler | **domain-web** | Handlers run on any thread |
| trading, payment | **domain-fintech** | Audit + thread safety |
| CLI, terminal | **domain-cli** | Usually single-thread OK |

**Example:**
```
"Rc cannot be sent between threads" in Web API
    ↑ DETECT: Web API → Load domain-web
    ↑ FIND: "Shared state must be thread-safe"
    ↑ FIND: "Rc in state" is Common Mistake
    ↓ DESIGN: Arc<T> with State extractor
```

## Critical Async Pattern

```rust
// Bad: guard held across await
let guard = mutex.lock().await;
do_async().await;  // guard still held!

// Good: scope the lock
{
    let guard = mutex.lock().await;
}  // guard dropped
do_async().await;
```

## Thinking Prompt

1. **Workload?** CPU-bound → threads/rayon, I/O-bound → async
2. **Sharing model?** None → channels, Immutable → Arc, Mutable → Arc<Mutex<T>>
3. **Send/Sync?** Cross-thread ownership → Send, Cross-thread refs → Sync
