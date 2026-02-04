# Background Agent Registry & Auto-Tracking

**Implemented:** 2026-02-04

## Overview

Automatic tracking of background agents with 30-second polling, registry persistence across sessions, and auto-recovery capabilities.

## Problem Solved

- Agents lost during context compaction
- No visibility into agent status between prompts
- Manual status checking required

## Architecture

```
Agent Launch
     │
     ▼
PostToolUse Hook ──► Register in JSON ──► Start Watcher
     │                                          │
     │                                          ▼
     │                              Poll every 30 seconds
     │                                          │
     │                                          ▼
     │                              Check output for "stop_reason"
     │                                          │
     │                                          ▼
     └─────────────────────────────► Update registry to "completed"
                                                │
                                                ▼
                                    Auto-exit when no agents
```

## Files

### Scripts

| File | Purpose |
|------|---------|
| `~/.claude/scripts/agent-watcher.sh` | Background daemon polling every 30 seconds |
| `~/.claude/scripts/log-agent-start.sh` | SubagentStart hook handler |
| `~/.claude/scripts/log-agent-stop.sh` | SubagentStop hook handler |
| `~/.claude/scripts/log-agent-usage.sh` | Registers agents + starts watcher |
| `~/.claude/scripts/status-line.sh` | Completion detection on each prompt |

### Data

| File | Purpose |
|------|---------|
| `~/.claude/running-agents.json` | Registry persistence |
| `~/.claude/agent-watcher.log` | Watcher activity log |

### Configuration

Added to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/scripts/log-agent-start.sh",
            "async": true
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/scripts/log-agent-stop.sh",
            "async": true
          }
        ]
      }
    ]
  }
}
```

## Registry Schema

```json
{
  "agents": [
    {
      "task_id": "abc123",
      "agent_type": "Explore",
      "description": "Find auth handler",
      "started_at": "2026-02-04T03:00:00Z",
      "output_file": "/private/tmp/claude/.../abc123.output",
      "status": "running|completed",
      "working_directory": "/path/to/project",
      "completed_at": "2026-02-04T03:01:00Z"
    }
  ],
  "last_updated": "2026-02-04T03:01:00Z"
}
```

## How It Works

1. **Agent Launch** → PostToolUse hook registers in JSON + starts watcher daemon
2. **Background Polling** → Watcher polls every 30 seconds, checks output files for `"stop_reason"`
3. **Completion Detection** → Updates `status` to `"completed"` with timestamp
4. **Auto-Exit** → Watcher exits when no running agents remain
5. **Session Recovery** → `/agents-status` skill reads registry for lost agents

## Usage

```bash
# View registry
cat ~/.claude/running-agents.json

# Check watcher
ps aux | grep agent-watcher

# Manual recovery
/agents-status
```

## Documentation Findings

From official Claude Code hooks documentation:

- `SubagentStart`/`SubagentStop` are official events but don't fire for Task-spawned agents
- `tool_response` is correct field (not `tool_result`)
- Field names use camelCase: `agentId`, `outputFile`
