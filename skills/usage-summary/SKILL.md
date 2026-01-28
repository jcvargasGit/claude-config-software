---
name: usage-summary
description: Show a comprehensive summary of agents and skills used, with time saved metrics, workflow suggestions, and trend analysis.
model: opus
---

# Usage Summary Skill

Display a comprehensive summary of Claude Code agent and skill usage with productivity insights.

## Instructions

1. Run the usage summary script to get the daily log:

```bash
~/.claude/scripts/usage-summary.sh
```

2. For a specific date, pass YYYY-MM-DD as argument:

```bash
~/.claude/scripts/usage-summary.sh 2026-01-08
```

3. For full activity log, add `--full` or `-f`:

```bash
~/.claude/scripts/usage-summary.sh 2026-01-08 --full
```

4. Display the output to the user in a clean, readable format.

## Output Sections

### Core Metrics
- **Overview**: Total agents/skills/interactions count
- **Estimated Time Saved**: Time saved from agents AND skills (with breakdown)
- **Agent Usage Breakdown**: Percentage breakdown with progress bars and time estimates
- **Skill Usage Breakdown**: Percentage breakdown with time estimates per skill
- **Agent + Skill Combinations**: Which skills were loaded before which agents

### Activity Analysis
- **Project Activity**: Which projects used agents
- **Activity by Hour**: Peak productivity hours visualization
- **Task Categories**: Code Review, Development, Architecture, Exploration breakdown

### Trend Analysis
- **Weekly Trend (7 days)**: Daily breakdown with time saved
- **Monthly Trend (30 days)**: Weekly summaries with estimated value
- **Quarterly Overview (90 days)**: Long-term metrics with annual projection

### Productivity Insights
- Percentage of workday/workweek saved
- Most used agent
- Peak productivity hour
- Daily/weekly/monthly/annual ROI calculation ($75/hr rate)

### Workflow Improvement Suggestions
Automated analysis that suggests:
- When to use `/feature-dev` based on explore/develop ratio
- When skills should be loaded before agents
- When code review is missing after development
- When test-engineer should be used
- When architecture review is needed

### Missed Opportunities Detection
Identifies patterns where:
- Agents could have run in parallel
- Skills weren't loaded before agents
- Review suite was incomplete for PR work
- Error handling review was skipped

### Bottleneck Analysis
Identifies slow tasks (>5 minutes) to optimize workflow:
- **Bottleneck Detection**: Flags any agent/skill that takes longer than 5 minutes
- **Duration Tracking**: Shows actual execution time for all tasks with timing data
- **Pattern Analysis**: Identifies which agents tend to run slow
- **Statistics**: Average, fastest, and slowest task durations
- **Tips**: Suggestions for reducing bottlenecks (parallelization, task splitting)

## Duration Tracking (PreToolUse/PostToolUse Hooks)

The system uses paired hooks to track actual task duration:

1. **PreToolUse Hook** (`log-tool-start.sh`): Captures start timestamp
2. **PostToolUse Hook** (`log-agent-usage.sh`): Calculates duration and logs

Log format with duration:
```
[2026-01-14 10:30:00] AGENT: backend-developer | Create handler | skills: lang-golang | duration: 180s | project: /path
[2026-01-14 10:30:00] BOTTLENECK: backend-developer | Create handler | 320s | project: /path
```

Tasks exceeding 5 minutes (300s) are flagged as BOTTLENECK entries for easy filtering.

## Skill Time Estimates

Skills now have time estimates (minutes saved per use):
- Language skills (golang, python, typescript): 6-8 min
- Architecture skills (hexagonal, cloud): 12-15 min
- Cloud/DevOps (sam, aws, terraform): 10-12 min
- Testing skills: 10 min
- Spec skills (prd, stories, api): 8-15 min
- Workflow skills (commit, review-pr): 5-20 min

## Agent + Skill Tracking

The log now tracks which skills were loaded before each agent spawn:
```
[2026-01-14 10:30:00] AGENT: backend-developer | Create handler | skills: lang-golang,arch-hexagonal | project: /path
[2026-01-14 10:30:00] COMBO: backend-developer + [lang-golang,arch-hexagonal] | project: /path
```

This enables analysis of which skill combinations are most effective.
