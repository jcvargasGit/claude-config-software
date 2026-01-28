---
name: review-pr
description: Comprehensive PR review with intelligent agent selection
argument-hint: "[PR number or branch] [--save]"
model: sonnet
allowed-tools: ["Bash", "Task", "AskUserQuestion"]
---

# PR Review Command

Comprehensive pull request review using intelligent agent selection based on the code being reviewed.

## ⚠️ IMPORTANT: Speed Rules

**After agents complete, DO NOT:**
- Read source files directly (agents already did this)
- Re-run git diff (agents already have the diff)
- Re-analyze or verify agent findings
- Do ANY additional investigation

**Your ONLY job after agents complete:**
1. Extract issues from agent outputs
2. Present each issue with nice formatting
3. Ask per-issue question
4. Post selected issues to PR

**If diff is large:** That's fine. Agents handle it. Do NOT try to read files yourself.

## Instructions

### Step 1: Sync with Remote

First, pull the latest changes to ensure we're reviewing up-to-date code:

```bash
git fetch origin
git pull origin $(git branch --show-current) --rebase 2>/dev/null || true
```

### Step 2: Get PR Information (ONE command only)

Get PR number and diff in a single command:

```bash
# Get PR number (use argument if provided, else current branch)
PR_NUM=$(gh pr view ${ARGUMENTS:-} --json number -q '.number' 2>/dev/null || echo "")
if [ -z "$PR_NUM" ]; then
  echo "No PR found. Create a PR first or provide PR number."
  exit 1
fi
echo "Reviewing PR #$PR_NUM"
```

**Store PR_NUM and reuse it. DO NOT run gh pr view again.**

### Step 3: Get Diff (ONE command, store result)

```bash
# Get file list and full diff in one call
DIFF=$(gh pr diff $PR_NUM)
echo "$DIFF" | head -50  # Preview first 50 lines
```

**Store DIFF variable. Pass it to agents. DO NOT run gh pr diff again.**

### Step 4: Run Code Review Agent

Launch ONE agent with the diff (simpler = faster):

```
Task tool parameters:
- subagent_type: "feature-dev:code-reviewer"
- max_turns: 10
- run_in_background: false  # Wait for result
- prompt: |
    Review this PR diff. Report issues with confidence >= 80% only.

    ## Diff:
    $DIFF

    Output format for each issue:
    - Severity: CRITICAL/IMPORTANT/MINOR
    - File: FULL path from repo root (e.g., internal/handlers/main.go:42-45)
    - Issue: description
    - Code: the problematic code snippet with line numbers
    - Fix: suggested code fix

    IMPORTANT: Always use FULL file paths (internal/..., cmd/..., pkg/...)
    DO NOT run git/gh commands. Analyze the diff above only.
```

**ONE agent is enough for most PRs. Skip parallel agents unless PR is huge.**

### Step 5: SKIP - DO NOT READ FILES - GO TO STEP 6

**CRITICAL RULES:**
- ❌ Do NOT read any source files
- ❌ Do NOT run git diff again
- ❌ Do NOT re-analyze the code
- ❌ Do NOT verify agent findings
- ✅ ONLY use issues from agent outputs
- ✅ Go IMMEDIATELY to Step 6

The agents already did the work. Your ONLY job now is to present their findings.

### Step 6: Ask About Each Issue (Per-Issue Questions)

**DO NOT run any git/gh commands here.** Use PR_NUMBER from Step 2.

**For EACH issue from agent outputs:**

1. **Copy file path to clipboard FIRST:**
   ```bash
   echo -n "path/to/file.go" | pbcopy
   ```

2. **Display the issue with color highlighting:**

   Use this Bash command to print with colors:
   ```bash
   echo -e "
   \033[90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\033[0m
   \033[1;31m🔴 CRITICAL\033[0m  │  Issue 1 of 5
   \033[90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\033[0m
   \033[36m📁 internal/handlers/confirmreception/main.go:64-68\033[0m

   \033[1mMissing error handling in user lookup\033[0m

   When GetUserDataLogin returns (nil, nil), the handler masks the underlying
   issue. A service misconfiguration would be reported as \"user not found\".

   \033[90m// 📍 internal/handlers/confirmreception/main.go:64-68\033[0m
   user, err := h.userService.GetUserDataLogin(ctx, req.UserID)
   if err != nil {
       return nil, err
   }
   if user == nil {
       return nil, \033[41;37mErrorCodeUserNotFound\033[0m  \033[31m← BUG: no logging before returning\033[0m
   }

   \033[32m✨ Fix:\033[0m
   if user == nil {
       \033[42;30mh.logger.Warn(\"user service returned nil without error\", \"userID\", req.UserID)\033[0m
       return nil, ErrorCodeUserNotFound
   }

   \033[33m💡 Pro tip:\033[0m \"Errors are like vegetables - ignoring them doesn't make them go away.\"
   \033[90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\033[0m
   "
   ```

   **Color codes used:**
   - `\033[41;37m` = Red background, white text (ONLY the problematic part)
   - `\033[42;30m` = Green background, black text (ONLY the fix/new code)
   - `\033[31m` = Red text (bug explanation after ←)
   - `\033[32m` = Green text (fix label)
   - `\033[36m` = Cyan text (file path)
   - `\033[33m` = Yellow text (tips)
   - `\033[1m` = Bold
   - `\033[0m` = Reset

   **Format rules:**
   - Highlight ONLY the specific problematic token/value (e.g., `""`, `nil`, wrong variable)
   - Highlight ONLY the new/fixed code in green
   - Keep surrounding code normal so the issue stands out
   - Arrow `←` explains what's wrong

   **Severity indicators:**
   - `🔴 CRITICAL` - Must fix (bugs, security)
   - `🟡 IMPORTANT` - Should fix (code quality)
   - `🔵 SUGGESTION` - Nice to have (style)

   **Fun tips to rotate (pick random):**
   - "Errors are like vegetables - ignoring them doesn't make them go away."
   - "This code trusts external services more than I trust Monday mornings."
   - "Future you will mass DM 'thank you' for catching this."
   - "Every silent failure is a mystery novel you'll debug at 3am."
   - "This is the kind of bug that waits until prod to say hello."
   - "Schrödinger's error: it both exists and doesn't until you check."
   - "Your future self called - they want you to fix this now."
   - "This bug has 'urgent Slack message' written all over it."

3. **Ask for THIS issue:**
   ```
   AskUserQuestion:
   Question: "Add this issue as a PR comment?"
   Header: "PR Comment"
   Options:
   - "Yes, add this comment"
   - "No, skip this one"
   - "Skip all remaining issues"
   ```

4. **If "Skip all remaining"**, stop asking and go to posting.

**After all issues**, post comment using PR_NUM from Step 2:
```bash
gh pr comment $PR_NUM --body "## Code Review

[Selected issues here]

---
*Review by Claude Code*"
```

**Use stored PR_NUM. DO NOT run gh pr view or git remote again.**

### Step 7: Save Review Summary (OPTIONAL - Bash only)

**Only save if user explicitly requests with `--save` flag.**

If saving, use Bash heredoc (no Write tool, no permissions needed):

```bash
mkdir -p .claude/reviews
cat > ".claude/reviews/PR-${PR_NUM}-$(date +%Y-%m-%d).md" << 'EOF'
# PR Review: #$PR_NUM

## Issues Found
[list from agents]

## Posted as Comments
[list of issues user selected]
EOF
```

**DO NOT use Write tool or Task agent for saving - causes permission delays.**

## Speed Summary

**Commands to run (total: 3):**
1. `git fetch && git pull` - sync
2. `gh pr view --json number` - get PR number
3. `gh pr diff $PR_NUM` - get diff

**Then:** Pass diff to ONE agent, present findings, post comment.

**DO NOT:** Run multiple gh commands, detect repo URL, run git remote, etc.

## Example Usage

```bash
# Review PR by number
/review-pr 82

# Review PR for current branch
/review-pr

# Review PR by branch name
/review-pr feature/TA-81-migrate-st-create-expiration
```

## Notes

- **Speed optimization:** NO summary step - go straight from agents to per-issue questions
- **Per-issue flow:**
  1. Copy file path to clipboard
  2. Show: file, line, issue description, current code, suggested fix
  3. Ask: "Add as PR comment?" (Yes / No / Skip all)
- **Quality preserved:** Sonnet model, rich display, code snippets
- Only high-confidence issues (≥80%) are reported
- PR comments batch-posted at the end
