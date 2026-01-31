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
git clone https://github.com/AlexKVal/rust-skills-for-cursor.git ~/.cursor/skills/rust-skills
```

> ⚠️ **Important**: Clone directly to `~/.cursor/skills/`. Symlinks are not supported—Cursor doesn't follow symlinks when discovering skills.

### Step 2: Restart Cursor

Skills are discovered when Cursor starts. After cloning, restart the IDE.

### Step 3: Verify Installation

1. Open **Cursor Settings** (Cmd+Shift+J on Mac, Ctrl+Shift+J on Windows/Linux)
2. Navigate to **Rules**
3. Check the **Skills** section — `rust-skills` should appear there


## Verify It Works

Open any Rust project in Cursor and try this test prompt:

```
My PagedTable needs to borrow the Pager, but I also need to pass it to BufferPool. Getting borrow conflicts.
```

**If rust-skills is working**, the response should:
- Ask "Who should own this data?" (not just "use clone")
- Analyze the **design roles** (Pager as I/O, BufferPool as cache, etc.)
- Suggest an **ownership hierarchy** based on domain context
- Reference **layers** or mention tracing through them

**Example of a working response:**
> "The key insight from the ownership skill: 'Who should own this data?' — The Pager is an I/O resource. BufferPool is the caching layer. PagedTable is the record-level abstraction. Clean ownership hierarchy, no shared references needed."

If you just get "use `Arc`" or "add `.clone()`" without design reasoning, the skill may not be loaded.

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

### Layer 2 Skills (Design & Quality)

| Skill | Purpose |
|-------|---------|
| `code-review` | Code smell detection, refactoring guidance |

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
    ├── code-review/SKILL.md    # Code smell analysis
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
