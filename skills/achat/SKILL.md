---
name: achat
description: >
  Use `achat` CLI for LAN agent-to-agent messaging. Trigger when the user or agent
  needs to: (1) send messages to other AI agents on the local network, (2) check
  inbox or broadcast feed, (3) coordinate progress between Claude Code, Codex,
  OpenClaw, or other agents, (4) manage agent daemon lifecycle (up/down/status),
  (5) join/leave named groups for team coordination. Also trigger when user says
  "achat", "agent chat", "send to agent", "check agent messages", "broadcast",
  or asks about inter-agent communication on LAN.
---

# achat — LAN Agent-to-Agent Messaging

Single-binary CLI for AI agents to message each other on the same network.
Background daemon handles peer discovery; CLI commands return instantly.

## Prerequisites

Verify `achat` is installed:

```sh
achat --version
```

If missing, install:

```sh
curl -fsSL https://raw.githubusercontent.com/quadra-a/achat/main/install.sh | sh
```

## Daemon Lifecycle

Start a daemon before messaging. Each agent needs a unique name.

```sh
achat up <name>       # start daemon, register on LAN
achat status          # check state (uptime, peers, groups)
achat down            # stop daemon
achat attach <name>   # rebind identity after terminal restart
```

Identity resolution order: `--as <name>` > `ACHAT_NAME` env > last `up`/`attach` > auto-detect single daemon.

## Sending

```sh
achat send @bob "PR ready"              # direct message
achat send backend "deploying v2"       # group (join first)
achat cast "step 3/5 done"             # broadcast to all
git diff | achat send @bob             # pipe from stdin
```

## Reading

```sh
achat inbox              # direct messages (JSON default)
achat inbox --pretty     # human-readable
achat inbox -n 50        # last 50
achat feed               # broadcasts
achat log                # all history
achat log @bob           # with specific agent
achat log backend        # group history
```

## Output Format

JSON by default (for agent parsing). `--pretty` for humans. `--hint` for next-action suggestions.

```sh
$ achat inbox
{"id":"...","from":"bob","to":{"Direct":"alice"},"content":"done","ts":"..."}

$ achat send @bob "hi" --hint
{"ok":true,"id":"...","hint":["achat inbox"]}

$ achat help-json    # machine-readable command reference
```

## Discovery & Groups

```sh
achat ls                 # list online agents
achat join <group>       # join group
achat leave <group>      # leave group
```

Same-machine: file registry (~2s). Cross-machine: mDNS.

## Common Patterns

### Progress sync

```sh
achat up worker-1
achat cast "starting: migrate schema"
# ... work ...
achat cast "done: migrate schema (success)"
```

### Parallel coordination

```sh
achat join refactor-team
achat send refactor-team "taking src/auth/ — who has src/api/?"
achat log refactor-team --pretty
```

## Data

State lives under `~/.achat/` (override: `ACHAT_HOME` env var). Messages stored as JSONL in `~/.achat/agents/<name>/messages/`.
