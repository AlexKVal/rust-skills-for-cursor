# Changelog

## [1.0.0] - 2026-01-31

### Fork Creation

This is the initial release of **rust-skills-for-cursor**, a cleaned and optimized fork of [ZhangHanDong/rust-skills](https://github.com/ZhangHanDong/rust-skills) specifically for Cursor IDE.

### What Changed

#### Removed (not needed for Cursor)

- `.claude/` - Claude Code hooks and settings
- `.claude-plugin/` - Claude Code plugin system
- `.codex/` - Codex platform support
- `.opencode/` - OpenCode platform support
- `agents/` - Claude Code background agents
- `commands/` - Claude Code slash commands
- `hooks/` - Claude Code hook triggers
- `scripts/` - Claude Code utility scripts
- `templates/` - Code templates
- `tests/` - Test files
- `docs/` - Documentation (mostly Chinese)
- `index/` - Index files
- 22 unreferenced skills

#### Optimized

Skills were cleaned to remove redundant information:

| Skill | Before | After | Reduction |
|-------|--------|-------|-----------|
| m01-ownership | 135 lines | 52 lines | 61% |
| m06-error-handling | 167 lines | 52 lines | 69% |
| m07-concurrency | 223 lines | 68 lines | 70% |
| unsafe-checker | 87 lines | 63 lines | 28% |
| domain-fintech | 147 lines | 68 lines | 54% |
| domain-web | 157 lines | 78 lines | 50% |
| domain-cli | 162 lines | 72 lines | 56% |
| domain-embedded | 179 lines | 96 lines | 46% |

**Removed from skills:**
- Standard Rust knowledge the AI already has
- Redundant error code tables
- Verbose decision flowcharts
- Orphan sub-files (patterns/, examples/)

**Preserved:**
- Meta-cognition framework (trace layers)
- "Don't just say X, ask Y instead" prompts
- Cross-references for layer tracing
- Domain-specific constraints and patterns

#### Verified

- Cross-references between skills intact
- Layer tracing mechanism works
- Original REVIEW.md concerns addressed
