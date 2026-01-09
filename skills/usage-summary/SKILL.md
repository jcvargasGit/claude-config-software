---
name: usage-summary
description: Show a summary of agents and skills used today or for a specific date.
---

# Usage Summary Skill

Display a summary of Claude Code agent and skill usage.

## Instructions

1. Run the usage summary script to get the daily log:

```bash
~/.claude/scripts/usage-summary.sh
```

2. For a specific date, pass YYYY-MM-DD as argument:

```bash
~/.claude/scripts/usage-summary.sh 2025-01-08
```

3. Display the output to the user in a clean, readable format.

## Output Format

The summary shows:
- Total count of agents used
- Total count of skills used
- Breakdown by agent type
- Breakdown by skill name
- Full log with timestamps
