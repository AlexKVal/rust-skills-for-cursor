# Rust Skills for Cursor

> A cleaned and optimized fork of [ZhangHanDong/rust-skills](https://github.com/ZhangHanDong/rust-skills) specifically for Cursor IDE.

## What is This?

A **meta-cognition framework** for Rust development in Cursor. Instead of giving surface-level answers, it guides the AI to trace through cognitive layers to provide domain-correct solutions.

### The Problem

```
User: "My trading system reports E0382"
AI: "Use .clone()"  ← Surface fix, ignores domain constraints
```

### The Solution

```
User: "My trading system reports E0382"

AI (with rust-skills):
├── Layer 1: E0382 = ownership error
│       ↑ Trace UP
├── Layer 3: Trading = audit trail requirement → immutable shared data
│       ↓ Trace DOWN  
└── Solution: Use Arc<T>, not clone()
```

## Installation

### Step 1: Clone to Cursor Skills Directory

```bash
git clone https://github.com/YOUR_USERNAME/rust-skills-for-cursor.git ~/.cursor/skills/rust-skills
```

### Step 2: Done

The skill is now available globally for all your Rust projects. Cursor will automatically use it when you work with Rust code.

## What's Included

### Core Framework (`SKILL.md`)

The main skill file contains:
- Meta-cognition framework (trace layers before answering)
- Routing logic (error → L1, design → L2, domain → L3)
- Domain awareness triggers

### Layer 1 Skills (Language Mechanics)

| Skill | Purpose |
|-------|---------|
| `m01-ownership` | Ownership, borrowing, lifetimes |
| `m06-error-handling` | Result, Option, thiserror, anyhow |
| `m07-concurrency` | Async, Send/Sync, threading |
| `unsafe-checker` | Unsafe code review, FFI |

### Layer 3 Skills (Domain Constraints)

| Skill | Purpose |
|-------|---------|
| `domain-fintech` | Audit trails, decimal precision, immutability |
| `domain-web` | Async handlers, state management, axum patterns |
| `domain-cli` | Clap, exit codes, stdout/stderr conventions |
| `domain-embedded` | no_std, heapless, interrupt safety |

## How It Works

1. **You ask a Rust question** (error, design, domain)
2. **AI identifies entry layer** (L1 mechanics, L2 design, L3 domain)
3. **AI traces through layers** to find root cause
4. **AI provides domain-appropriate solution**

## Differences from Original

This fork is specifically optimized for Cursor IDE:

| Original | This Fork |
|----------|-----------|
| 30+ skills | 8 essential skills |
| Claude Code hooks, agents, commands | Cursor skill only |
| Verbose explanations | Concise, agent-optimized |
| Multiple platform support | Cursor-only |

### Cleanup Applied

1. **Removed Claude Code specifics** - hooks, agents, commands, plugins
2. **Kept only referenced skills** - 8 skills actually used
3. **Optimized skill content** - removed redundant info the AI already knows
4. **Deleted orphan sub-files** - patterns/, examples/ not referenced
5. **Verified tracing intact** - cross-references between skills preserved

## File Structure

```
~/.cursor/skills/rust-skills/
├── SKILL.md                    # Entry point - meta-cognition framework
└── skills/
    ├── m01-ownership/SKILL.md  # Ownership & lifetimes
    ├── m06-error-handling/SKILL.md
    ├── m07-concurrency/SKILL.md
    ├── unsafe-checker/SKILL.md
    ├── domain-fintech/SKILL.md
    ├── domain-web/SKILL.md
    ├── domain-cli/SKILL.md
    └── domain-embedded/SKILL.md
```

## Credits

- Original project: [ZhangHanDong/rust-skills](https://github.com/ZhangHanDong/rust-skills)
- Meta-cognition framework concept by Zhang Han Dong

## License

MIT License - see original project for details.
