---
name: domain-cli
description: "Use when building CLI tools. Keywords: CLI, command line, terminal, clap, argument parsing, subcommand, progress bar"
user-invocable: false
---

# CLI Domain

> **Layer 3: Domain Constraints**

## Domain Constraints → Rust Implications

| Domain Rule | Why | Rust |
|-------------|-----|------|
| Errors to stderr | Pipeable output | eprintln! for errors |
| Config precedence | CLI > env > file > defaults | Layered config |
| Exit codes | Script integration | Non-zero on error |

## Critical Rules

- **Errors to stderr, data to stdout** → `eprintln!` for errors
- **Return non-zero on error** → `main() -> Result<(), Error>`
- **Show progress for long ops** → `indicatif`

## Clap Pattern

```rust
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "myapp", about = "My CLI tool")]
struct Cli {
    #[arg(short, long)]
    verbose: bool,

    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    Init { name: String },
    Run {
        #[arg(short, long)]
        port: Option<u16>,
    },
}

fn main() -> anyhow::Result<()> {
    let cli = Cli::parse();
    match cli.command {
        Commands::Init { name } => init_project(&name)?,
        Commands::Run { port } => run_server(port.unwrap_or(8080))?,
    }
    Ok(())
}
```

## Key Crates

| Purpose | Crate |
|---------|-------|
| Argument parsing | clap |
| Interactive prompts | dialoguer |
| Progress bars | indicatif |
| Colored output | colored |
| Terminal UI | ratatui |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Errors to stdout | eprintln! |
| Panic on error | Result + proper handling |
| No progress for long ops | indicatif |
