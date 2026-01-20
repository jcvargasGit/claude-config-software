---
name: commit
description: Generate semantic commit message and commit staged changes
---

# Commit Command

Generate a commit message following conventional commits and commit the staged changes.

## Instructions

**EFFICIENCY RULE: This is a simple command. Execute in exactly 2 steps - no extra commands.**

### Step 1: Get status and diff (single command)
```bash
git status && git diff --staged
```

### Step 2: Commit immediately
Generate message and commit. Do NOT run `git log` or any other commands.

## Commit Message Format

```
type(scope): BRANCH_NAME short description

- bullet points for details (if needed)
```

**Types:** `feat` | `fix` | `test` | `refactor` | `docs` | `chore`

**Rules:**
- Scope: use folder/module name if relevant
- Description: lowercase, no period, imperative mood ("add" not "added")
- Keep under 72 characters
- Only add bullet points if multiple distinct changes
- **NEVER add "Co-Authored-By: Claude"**

## Examples
```
feat(skills): feature/TA-123 add Python and Node.js language skills
fix(api): bugfix/API-456 handle null response from user service
refactor(handlers): feature/TA-789 extract validation logic
```
