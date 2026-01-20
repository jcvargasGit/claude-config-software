---
name: review-pr
description: Review PR/branch changes using git diff (no gh CLI). Runs code-reviewer agent in background, presents issues with formatting, asks per-issue actions.
model: opus
---

# PR Review Skill

Comprehensive pull request review using intelligent agent selection based on the code being reviewed.

## CRITICAL RULES (from CLAUDE.md - NON-NEGOTIABLE)

- **NEVER use `gh` CLI** - Use `git diff` instead
- **code-reviewer agent MUST run in background** (`run_in_background: true`)

## Instructions

### Step 1: Sync with Remote

```bash
git fetch origin
git pull origin $(git branch --show-current) --rebase 2>/dev/null || true
```

### Step 2: Get Diff Using Git (NOT gh)

```bash
# Get file list
git diff --name-only dev...HEAD

# Get stats
git diff --stat dev...HEAD

# Get full diff
DIFF=$(git diff dev...HEAD)
echo "$DIFF" | head -100  # Preview
```

**Store DIFF variable. Pass it to agent. DO NOT run git diff again.**

### Step 3: Run Code Review Agent (BACKGROUND)

Launch code-reviewer agent with `run_in_background: true`:

```
Task tool parameters:
- subagent_type: "feature-dev:code-reviewer"
- max_turns: 10
- run_in_background: true  # REQUIRED per CLAUDE.md
- prompt: |
    Before starting, load these skills using the Skill tool: /lang-golang, /clean-code
    Follow the patterns and best practices from these skills throughout your work.

    Review this PR diff. Report issues with confidence >= 80% only.

    ## Changed Files:
    [list from git diff --name-only]

    Output format for each issue:
    - Severity: CRITICAL/IMPORTANT/MINOR
    - File: FULL path from repo root
    - Line: specific line number(s)
    - Issue: description
    - Code: the problematic code snippet
    - Fix: suggested code fix

    DO NOT run git/gh commands. Read the files directly to analyze.
```

**Tell user:** "**code-reviewer agent** running in background. You can continue working."

### Step 4: Wait for Agent Completion

When the agent completes (you'll receive a task notification), proceed to Step 5.

### Step 5: Present Issues (DO NOT re-analyze)

**CRITICAL:** Use ONLY the agent's findings. Do NOT:
- Read source files again
- Run git diff again
- Re-analyze or verify findings

**For EACH issue from agent output:**

1. **Copy file path to clipboard:**
   ```bash
   echo -n "path/to/file.go" | pbcopy
   ```

2. **Display with color formatting:**
   ```bash
   echo -e "
   \033[90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\033[0m
   \033[1;31m🔴 CRITICAL\033[0m  │  Issue 1 of N
   \033[90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\033[0m
   \033[36m📁 path/to/file.go:42-45\033[0m

   \033[1mIssue Title\033[0m

   Description of the issue.

   \033[90m// 📍 file.go:42-45\033[0m
   \033[41;37mproblematic code\033[0m  \033[31m← explanation\033[0m

   \033[32m✨ Fix:\033[0m
   \033[42;30mfixed code\033[0m

   \033[33m💡 Pro tip:\033[0m \"Fun tip here\"
   \033[90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\033[0m
   "
   ```

   **Severity indicators:**
   - `🔴 CRITICAL` - Must fix (bugs, security)
   - `🟡 IMPORTANT` - Should fix (code quality)
   - `🔵 SUGGESTION` - Nice to have (style)

   **Color codes:**
   - `\033[41;37m` = Red background (problematic code)
   - `\033[42;30m` = Green background (fix)
   - `\033[31m` = Red text (bug explanation)
   - `\033[32m` = Green text (fix label)
   - `\033[36m` = Cyan text (file path)

   **Fun tips (rotate):**
   - "Errors are like vegetables - ignoring them doesn't make them go away."
   - "This code trusts external services more than I trust Monday mornings."
   - "Future you will mass DM 'thank you' for catching this."
   - "Every silent failure is a mystery novel you'll debug at 3am."
   - "Your future self called - they want you to fix this now."

3. **Ask per issue:**
   ```
   AskUserQuestion:
   Question: "What would you like to do with this issue?"
   Header: "Issue N/M"
   Options:
   - "Note for fixing" - Track this issue
   - "Skip this one" - Not relevant
   - "Skip all remaining" - Stop reviewing
   ```

4. **If "Skip all remaining"**, stop asking.

### Step 6: Summary

After all issues reviewed, provide a summary:

```
## Review Summary

**Branch:** feature/XXX → dev
**Files Changed:** N
**Issues Found:** M (X critical, Y important, Z suggestions)

### Issues to Address:
1. [file:line] Issue description
2. [file:line] Issue description

### Skipped:
- [file:line] Issue description (user skipped)
```

### Step 7: Save Review (OPTIONAL)

Only if user requests with `--save`:

```bash
mkdir -p .claude/reviews
cat > ".claude/reviews/$(git branch --show-current)-$(date +%Y-%m-%d).md" << 'EOF'
# Code Review: branch-name

## Issues Found
[list]

## Action Items
[list of issues user noted]
EOF
```

## Example Usage

```bash
# Review current branch against dev
/review-pr

# Review specific branch
/review-pr feature/TA-501
```

## What This Skill Does NOT Do

- **No `gh` CLI** - Cannot post PR comments directly
- **No PR number lookup** - Works with branches, not PR numbers
- To post comments to PR, copy the formatted output manually
