---
name: review-pr
description: Comprehensive PR review with intelligent agent selection and summary saving
argument-hint: "[PR number or branch name]"
model: opus
allowed-tools: ["Bash", "Glob", "Grep", "Read", "Write", "Task", "AskUserQuestion"]
---

# PR Review Command

Comprehensive pull request review using intelligent agent selection based on the code being reviewed.

## Instructions

### Step 1: Sync with Remote

First, pull the latest changes to ensure we're reviewing up-to-date code:

```bash
git fetch origin
git pull origin $(git branch --show-current) --rebase 2>/dev/null || true
```

### Step 2: Get PR Information

If an argument is provided ("$ARGUMENTS"), use it as the PR number or branch name.

```bash
# If numeric, it's a PR number
gh pr view $ARGUMENTS

# If branch name, find the PR
gh pr list --head $ARGUMENTS
```

If no argument, check current branch:
```bash
gh pr view
```

### Step 3: Analyze the Diff

Get the PR diff and analyze what types of files changed:

```bash
gh pr diff $PR_NUMBER --name-only
```

Categorize the changes:
- **Backend code**: `.go`, `.py`, `.java`, `.rs`, `.rb`
- **Frontend code**: `.ts`, `.tsx`, `.js`, `.jsx`, `.vue`, `.svelte`, `.css`, `.html`
- **Test files**: `*_test.go`, `*.test.ts`, `*.spec.ts`, `*_test.py`, etc.
- **Infrastructure**: `.tf`, `.yaml`, `.yml` (CloudFormation, k8s), `Dockerfile`
- **Types/Interfaces**: Files with type definitions, interfaces, structs

### Step 4: Select and Run Agents

Based on the file types detected, run the appropriate agents **in parallel** using the Task tool:

| File Types | Agents to Use |
|------------|---------------|
| Backend (.go, .py, etc.) | `backend-developer` (architecture review) |
| Frontend (.ts, .tsx, etc.) | `frontend-engineer` (UI/UX review) |
| Test files | `pr-review-toolkit:pr-test-analyzer` (test coverage) |
| Error handling code | `pr-review-toolkit:silent-failure-hunter` (error handling) |
| New types/interfaces | `pr-review-toolkit:type-design-analyzer` (type design) |
| All PRs | `pr-review-toolkit:code-reviewer` (general quality) |
| Comments/docs added | `pr-review-toolkit:comment-analyzer` (comment accuracy) |

**Always run `code-reviewer` as baseline. Add other agents based on detected file types.**

Launch agents in parallel for efficiency:
```
Use Task tool with multiple parallel calls for independent agents
```

### Step 5: Compile Review Summary

After all agents complete, compile a summary with detailed issue information:

```markdown
# PR Review Summary

**PR:** #[number] - [title]
**Branch:** [branch-name]
**Author:** [author]
**Date:** [date]

## Files Changed
- [list of files with change type: added/modified/deleted]

## Review Results

### Critical Issues (must fix)

#### Issue 1: [Title]
- **File:** `path/to/file.go:line_number`
- **Confidence:** [80-100]%
- **Description:** [What the issue is]
- **Current Code:**
  ```go
  [problematic code snippet]
  ```
- **Suggested Fix:**
  ```go
  [improved code snippet]
  ```

### Important Issues (should fix)
[Same format as critical issues]

### Suggestions (nice to have)
[Same format as critical issues]

### Strengths
- [positive observations]

## Agents Used
- [list of agents that ran]

## Recommendation
[APPROVE / REQUEST_CHANGES / COMMENT]
```

### Step 6: Ask User About PR Comments

For each issue found:

1. **MUST copy file path to clipboard FIRST** before displaying the issue:
   ```bash
   FILE_PATH="path/to/file.go"
   if command -v pbcopy >/dev/null 2>&1; then
     printf "%s" "$FILE_PATH" | pbcopy
   elif command -v xclip >/dev/null 2>&1; then
     printf "%s" "$FILE_PATH" | xclip -selection clipboard
   elif command -v xsel >/dev/null 2>&1; then
     printf "%s" "$FILE_PATH" | xsel --clipboard --input
   else
     echo "No clipboard tool (pbcopy/xclip/xsel) found; file path:"
     echo "$FILE_PATH"
   fi
   ```

2. Then display the issue information:

```markdown
---
### Issue [N]: [Title]
**File:** `path/to/file.go:line_number` (copied to clipboard)
**PR Files Link:** https://github.com/[org]/[repo]/pull/[pr_number]/files
**Severity:** Critical | Important | Suggestion
**Confidence:** [80-100]%

**Reason:** [Why this is an issue]

**Current Code:**
```go
[the problematic code snippet]
```

**Suggested Improvement:**
```go
[the improved code snippet]
```
---
```

**Link format**: `https://github.com/[org]/[repo]/pull/[pr_number]/files`
- Get PR number from `gh pr list --head [branch] --json number`
- Or from `gh pr view --json number`
- Get org/repo from: `git remote get-url origin | sed 's/.*github.com[:/]\(.*\)\.git/\1/'`

3. Use AskUserQuestion to ask for EACH issue individually:
```
Add this issue as a PR comment?
- Yes, add this comment
- No, skip this one
```

For each issue, record the user's choice (e.g., mark the issue as "selected" when they choose "Yes"). After going through all issues, if the user selected any to add, aggregate the selected issues into a single Markdown-formatted list string (for example, `$SELECTED_ISSUES_MARKDOWN`) that includes file:line and the suggested improvement for each issue.
```bash
# For a single general comment with all selected issues
gh pr comment $PR_NUMBER --body "## Code Review Comments

$SELECTED_ISSUES_MARKDOWN
"

# Or, alternatively, for line-specific comments (if PR is accessible via gh),
# iterate over the selected issues and post one comment per issue:
# gh pr review $PR_NUMBER --comment --body "$ISSUE_BODY" --file "$FILE" --line "$LINE"
```

### Step 7: Save Review Summary (Auto-Save)

**Automatically save the review summary** - no user approval needed:

```bash
mkdir -p .claude/reviews
```

Save as `.claude/reviews/PR-[number]-[date].md`

**Run the file write operation in the background** using the Task tool with `run_in_background: true` so the user can continue working while the summary is being saved. Do NOT ask for approval - just save it.

## Agent Selection Logic

```
IF has backend files (.go, .py, .java, .rs):
  → Run backend-developer agent

IF has frontend files (.ts, .tsx, .js, .jsx, .vue):
  → Run frontend-engineer agent

IF has test files (*_test.*, *.test.*, *.spec.*):
  → Run pr-test-analyzer agent

IF has error handling (try/catch, if err !=, .catch):
  → Run silent-failure-hunter agent

IF has new type definitions (type, interface, struct):
  → Run type-design-analyzer agent

IF has new/modified comments or docs:
  → Run comment-analyzer agent

ALWAYS:
  → Run code-reviewer agent (baseline)
```

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

- Agents run in parallel for faster reviews
- Only high-confidence issues (≥80) are reported
- Summary is auto-saved to `.claude/reviews/` (no approval needed)
- Summary is saved in the background (non-blocking)
- File path is copied to clipboard for each issue (for easy navigation in PR)
- PR comments are optional and user-controlled
- Uses extended thinking (opus model) for thorough analysis
