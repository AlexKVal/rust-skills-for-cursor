---
name: m01-ownership
description: "Use for ownership/borrow/lifetime issues. Triggers: E0382, E0597, E0506, E0507, value moved, borrowed value does not live long enough, ownership, borrow, lifetime"
user-invocable: false
---

# Ownership & Lifetimes

> **Layer 1: Language Mechanics**

## Core Question

**Who should own this data, and for how long?**

## Error → Design Question

| Error | Don't Just Say | Ask Instead |
|-------|----------------|-------------|
| E0382 | "Clone it" | Who should own this data? |
| E0597 | "Extend lifetime" | Is the scope boundary correct? |
| E0506 | "End borrow first" | Should mutation happen elsewhere? |
| E0507 | "Clone before move" | Why are we moving from a reference? |

## Thinking Prompt

1. **What is this data's domain role?**
   - Entity (unique identity) → owned
   - Value Object (interchangeable) → clone/copy OK
   - Temporary → maybe restructure

2. **Fix symptom or redesign?**
   - If 3rd attempt at same error → escalate to Layer 2

## Trace Up ↑

When errors persist:
- E0382 repeated → m02-resource (Arc/Rc for sharing?)
- E0597 repeated → m09-domain (scope boundary at right place?)
- E0506/E0507 → m03-mutability (interior mutability?)

## Anti-Patterns

| Anti-Pattern | Better |
|--------------|--------|
| `.clone()` everywhere | Design ownership properly |
| `'static` for everything | Use appropriate lifetimes |
| `Box::leak` | Proper lifetime design |
