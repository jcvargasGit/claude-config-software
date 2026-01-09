---
name: commit
description: Generate semantic commit message and commit staged changes
---

# Commit Command

Generate a commit message following conventional commits and commit the staged changes.

## Instructions

1. Run `git status` and `git diff --staged` to see what's being committed
2. Generate a commit message in this format:
3. The commit message should contain the current branch name

```
type(scope): short description

- bullet points for details (if needed)
```

**Types:**
- `feat` - new feature
- `fix` - bug fix
- `test` - adding or updating tests

**Rules:**
- Scope is optional, use folder/module name if relevant
- Description: lowercase, no period, imperative mood ("add" not "added")
- Keep it under 72 characters
- Only add bullet points if multiple distinct changes

3. Run the commit with the generated message (no footer lines)
4. Show the commit hash when done

**Examples:**
```
feat(skills): BRANCH_NAME add Python and Node.js language skills
fix(api): BRANCH_NAME handle null response from user service
refactor(handlers): BRANCH_NAME extract validation logic
docs: BRANCH_NAME update README with new agent structure
chore(deps): BRANCH_NAME upgrade aws-sdk to v3
```
