# codex-inter-agent-chat

![codex-inter-chat-demo](./assets/demo/codex-inter-chat-demo-3x.gif)

[Watch full demo video](https://github.com/user-attachments/assets/5b977592-0d6f-4715-b512-bc8dd5bd4e7b)

A Codex-native port of [tessron/claude-code-skills/inter-agent-chat](https://github.com/tessron/claude-code-skills/tree/main/inter-agent-chat).

---

## Overview

- Disabled by default; only Codex sessions launched explicitly via `codex-team` will enable inter-agent chat.
- Codex sessions started with the same `--team` share one team-local registry and can message each other.
- Different `--team` values are fully isolated and do not interfere with each other.
- `codex-team` is a thin wrapper around the original `codex`: it fully inherits the normal Codex config, MCPs, and skills, and only appends the team-specific environment.
- On macOS, run it inside **tmux** when possible. The transport now prefers `tmux send-keys`; `TIOCSTI` is only a fallback.
- This is not for Codex `spawn_agent` / `send_input` subagents; it is for multiple independent Codex terminal sessions.

---

## Quick Start

### 1) Install

```bash
PROJECT_ROOT=~/projects/codex-inter-agent-chat
$PROJECT_ROOT/tools/install-skill.sh
```

The installer will:

1. install the skill into `~/.codex/skills/inter-agent-chat-codex`
2. install a real `codex-team` command into the same user bin area used by your current `codex`
3. update `~/.zshrc` if needed so that bin directory is on PATH

If your current shell has not refreshed yet, run:

```bash
source ~/.zshrc
```

### 2) Start two Codex sessions in the same team

Pane A:

```bash
codex-team --team reverse --agent-name agent-a
```

Pane B:

```bash
codex-team --team reverse --agent-name agent-b
```

### 3) Verify registrations

```bash
codex-team list --team reverse
```

### 4) Dry-run a message

```bash
codex-team send \
  --to agent-b \
  --message "status?" \
  --dry-run
```

### 5) Send a real message

```bash
codex-team send \
  --to agent-b \
  --message "Keep running tests and send me the result"
```

### 6) Show resolved identity

```bash
codex-team whoami
```

---

## Design Model

### Normal `codex`

```bash
codex
```

- inter-agent chat remains disabled
- no behavior changes

### `codex-team`

```bash
codex-team --team reverse --agent-name agent-a
```

- enables inter-agent chat only for the current process
- automatically sets a team-local registry
- fully inherits the original `codex` config, MCPs, and skills
- only appends these team-specific environment variables:
  - `CODEX_INTER_AGENT_CHAT_ENABLE=1`
  - `CODEX_INTER_AGENT_CHAT_TEAM=<team>`
  - `CODEX_INTER_AGENT_CHAT_REGISTRY_DIR=<registry-dir>`
  - `AGENT_NAME=<agent-name>`

---

## Multi-Team Isolation

Example:

```bash
codex-team --team reverse --agent-name agent-a
codex-team --team reverse --agent-name agent-b
codex-team --team qa --agent-name qa-a
codex-team --team qa --agent-name qa-b
```

Result:

- `reverse` can only see members inside `reverse`
- `qa` can only see members inside `qa`
- the two groups cannot message each other

---

## Usage

### Launch a team session

```bash
codex-team --team TEAM_NAME --agent-name AGENT_NAME
```

### Preview launcher environment

```bash
codex-team --team reverse --agent-name agent-a --print-env
```

### Use a custom registry dir

```bash
codex-team \
  --team reverse \
  --agent-name agent-a \
  --registry-dir /tmp/my-custom-registry
```

### Pass arguments through to Codex

```bash
codex-team \
  --team reverse \
  --agent-name agent-a \
  -- -m gpt-5.4
```

---

## Common Commands

### List registered agents

```bash
codex-team list --team reverse
```

### Check transport capability

```bash
codex-team capability
```

### Send a dry-run message

```bash
codex-team send --team reverse \
  --to agent-b \
  --message "status?" \
  --dry-run
```

### Send a real message

```bash
codex-team send --team reverse \
  --to agent-b \
  --message "reply via inter-agent-chat-codex."
```

### Broadcast to everyone in the same team

```bash
codex-team send --team reverse \
  --to all \
  --message "checkpoint reached"
```

### Remove a stale registration

```bash
codex-team unregister --team reverse agent-b
```

---

## Python CLI Usage

From source:

```bash
PYTHONPATH=~/projects/codex-inter-agent-chat/src \
python3 -m codex_inter_agent_chat.cli capability
```

Editable install:

```bash
cd ~/projects/codex-inter-agent-chat
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -e .
codex-inter-agent-chat capability
```

On macOS / Homebrew Python, prefer a local `.venv` because system `pip install -e .` is often blocked by PEP 668.

---

## Non-Interactive Demo

```bash
~/projects/codex-inter-agent-chat/tools/demo-two-agents.sh
```

This script exercises registration, listing, capability reporting, and dry-run dispatch.

---

## Compatibility and Limitations

- registry handling works on Linux and macOS
- transport is based on `TIOCSTI`
- newer Linux kernels may require `sudo` or `sysctl dev.tty.legacy_tiocsti=1`
- on macOS, `TIOCSTI` may be blocked by OS policy
- if capability looks fine but real sending fails, check terminal / OS policy first

---

## Wire Format

Default format:

```text
[@target] (from sender) message\r
```

Avoid `--raw` unless you intentionally want full manual control.

---

## Project Layout

```text
codex-inter-agent-chat/
├── src/codex_inter_agent_chat/        # Python implementation
├── skills/inter-agent-chat-codex/     # Codex skill package
├── tests/                             # unittest suite
├── tools/install-skill.sh             # install skill + codex-team launcher
├── tools/uninstall-skill.sh           # remove installed skill + launcher
├── tools/demo-two-agents.sh           # dry-run demo flow
├── tools/codex-team.sh                # launcher source
└── docs/inter-agent-chat/plan/        # working notes and plan
```

---

## Tests

```bash
cd ~/projects/codex-inter-agent-chat
PYTHONPATH=src python3 -m unittest discover -s tests -v
```

---

## Uninstall

Remove the installed skill and launcher:

```bash
~/projects/codex-inter-agent-chat/tools/uninstall-skill.sh
```

---

## Credit

- [tessron/claude-code-skills](https://github.com/tessron/claude-code-skills/)

---

## License

MIT. This port is derived from the original MIT-licensed work by tessron.
