# Claw Code Usage

This guide covers the current Rust workspace under `rust/` and the `claw` CLI binary. If you are brand new, make the doctor health check your first run: start `claw`, then run `/doctor`.

## Quick-start health check

Run this before prompts, sessions, or automation:

```bash
cd rust
cargo build --workspace
./target/debug/claw
# first command inside the REPL
/doctor
```

`/doctor` is the built-in setup and preflight diagnostic. Once you have a saved session, you can rerun it with `./target/debug/claw --resume latest /doctor`.

## Prerequisites

- Rust toolchain with `cargo`
- **No API key required for standalone mode** – install [Ollama](https://ollama.com) locally and pull a model:
  ```bash
  ollama pull llama3.2
  ```
  The CLI auto-detects Ollama when no cloud credentials are configured.
- For cloud models, one of:
  - `ANTHROPIC_API_KEY` for direct Anthropic API access
  - `claw login` for OAuth-based auth
  - `XAI_API_KEY` for xAI / Grok models
  - `OPENAI_API_KEY` for OpenAI-compatible models
- Optional: `ANTHROPIC_BASE_URL` / `OLLAMA_HOST` to override the default endpoint

## Install / build the workspace

```bash
cd rust
cargo build --workspace
```

The CLI binary is available at `rust/target/debug/claw` after a debug build. Make the doctor check above your first post-build step.

## Quick start

### Standalone (no API key) with Ollama

Start [Ollama](https://ollama.com) and pull a model once:

```bash
ollama serve          # starts the local server (or use the desktop app)
ollama pull llama3.2  # or any other model
```

Then simply run `claw` — it automatically uses Ollama when no cloud credentials are found:

```bash
cd rust
./target/debug/claw prompt "summarize this repository"
# or use a specific Ollama model
./target/debug/claw --model llama3.2 prompt "review this diff"
./target/debug/claw --model mistral  "explain src/main.rs"
```

Supported Ollama model aliases (passed through to Ollama as-is):
- `llama3`, `mistral`, `phi`, `gemma`, `qwen`
- Any raw Ollama tag like `llama3.2`, `mistral:7b`, `codellama`, etc.

Use `OLLAMA_HOST` to override the default endpoint (`http://localhost:11434/v1`):
```bash
export OLLAMA_HOST="http://remote-host:11434/v1"
```

### First-run doctor check

```bash
cd rust
./target/debug/claw
/doctor
```

### Interactive REPL

```bash
cd rust
./target/debug/claw
```

### One-shot prompt

```bash
cd rust
./target/debug/claw prompt "summarize this repository"
```

### Shorthand prompt mode

```bash
cd rust
./target/debug/claw "explain rust/crates/runtime/src/lib.rs"
```

### JSON output for scripting

```bash
cd rust
./target/debug/claw --output-format json prompt "status"
```

## Model and permission controls

```bash
cd rust
./target/debug/claw --model sonnet prompt "review this diff"
./target/debug/claw --model llama3.2 prompt "review this diff"   # Ollama
./target/debug/claw --permission-mode read-only prompt "summarize Cargo.toml"
./target/debug/claw --permission-mode workspace-write prompt "update README.md"
./target/debug/claw --allowedTools read,glob "inspect the runtime crate"
```

Supported permission modes:

- `read-only`
- `workspace-write`
- `danger-full-access`

Model aliases currently supported by the CLI:

| Alias | Resolves to | Provider |
|-------|-------------|----------|
| `opus` | `claude-opus-4-6` | Anthropic |
| `sonnet` | `claude-sonnet-4-6` | Anthropic |
| `haiku` | `claude-haiku-4-5-20251213` | Anthropic |
| `grok` / `grok-3` | `grok-3` | xAI |
| `grok-mini` / `grok-3-mini` | `grok-3-mini` | xAI |
| `llama3` | `llama3` | Ollama (local) |
| `mistral` | `mistral` | Ollama (local) |
| `phi` | `phi` | Ollama (local) |
| `gemma` | `gemma` | Ollama (local) |
| `qwen` | `qwen` | Ollama (local) |
| any other string | passed through | Ollama (local) when no cloud creds found |

## Authentication

### Standalone (no API key)

No setup required. Install [Ollama](https://ollama.com) and run:

```bash
ollama pull llama3.2
claw prompt "hello"
```

### Anthropic API key

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

### OAuth

```bash
cd rust
./target/debug/claw login
./target/debug/claw logout
```

### xAI / Grok

```bash
export XAI_API_KEY="xai-..."
./target/debug/claw --model grok prompt "hello"
```

### OpenAI-compatible

```bash
export OPENAI_API_KEY="sk-..."
./target/debug/claw --model gpt-4o prompt "hello"
```

## Common operational commands

```bash
cd rust
./target/debug/claw status
./target/debug/claw sandbox
./target/debug/claw agents
./target/debug/claw mcp
./target/debug/claw skills
./target/debug/claw system-prompt --cwd .. --date 2026-04-04
```

## Session management

REPL turns are persisted under `.claw/sessions/` in the current workspace.

```bash
cd rust
./target/debug/claw --resume latest
./target/debug/claw --resume latest /status /diff
```

Useful interactive commands include `/help`, `/status`, `/cost`, `/config`, `/session`, `/model`, `/permissions`, and `/export`.

## Config file resolution order

Runtime config is loaded in this order, with later entries overriding earlier ones:

1. `~/.claw.json`
2. `~/.config/claw/settings.json`
3. `<repo>/.claw.json`
4. `<repo>/.claw/settings.json`
5. `<repo>/.claw/settings.local.json`

## Mock parity harness

The workspace includes a deterministic Anthropic-compatible mock service and parity harness.

```bash
cd rust
./scripts/run_mock_parity_harness.sh
```

Manual mock service startup:

```bash
cd rust
cargo run -p mock-anthropic-service -- --bind 127.0.0.1:0
```

## Verification

```bash
cd rust
cargo test --workspace
```

## Workspace overview

Current Rust crates:

- `api`
- `commands`
- `compat-harness`
- `mock-anthropic-service`
- `plugins`
- `runtime`
- `rusty-claude-cli`
- `telemetry`
- `tools`
