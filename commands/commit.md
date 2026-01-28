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

### Step 2: Analyze and Commit

Check the number of changed files from Step 1.

#### Standard Commit (≤10 files)

Generate message and commit immediately. Do NOT run `git log` or any other commands.

#### Large Change Detection (>10 files)

When more than 10 files are changed, analyze and offer to split:

1. **Analyze changes by:**
   - Directory grouping (`services/auth`, `services/users`)
   - Change type (`feat`, `fix`, `test`, `refactor`, `docs`)
   - Logical feature boundaries

2. **Suggest commit groups:**
   ```
   Group 1: feat(auth): add login endpoint (5 files)
   Group 2: test(auth): add login integration tests (3 files)
   Group 3: refactor(auth): extract validation logic (4 files)
   ```

3. **Ask user to select groups or commit all together**

4. **For each selected group:**
   - Stage only the relevant files (`git add <files>`)
   - Commit with semantic message
   - Repeat for next group

## Commit Message Format

```
type(scope): BRANCH_NAME short description

- bullet points for details (if needed)
```

**Types:** `feat` | `fix` | `test` | `refactor` | `docs` | `chore` | `ci`

## Commit Boundaries

Each commit should represent one logical change:

| Good Commit | Bad Commit |
|-------------|------------|
| `feat(auth): add login endpoint` | `add login, fix bug, update tests` |
| `test(auth): add login integration tests` | `WIP` |
| `refactor(auth): extract validation logic` | `misc changes` |

**What goes together:**
- Feature + its tests = same commit (one logical unit)
- Refactoring = separate commit (behavior unchanged)
- Bug fix = separate commit (easy to revert)
- Config/infra changes = separate commit (different concern)

**Rules:**
- Scope: use folder/module name if relevant
- Description: lowercase, no period, imperative mood ("add" not "added")
- Keep under 72 characters
- Only add bullet points if multiple distinct changes
- **NEVER add "Co-Authored-By: Claude"**

## Final Steps

3. Run the commit with the generated message (no footer lines)
4. Show the commit hash when done

## Examples
```
feat(skills): feature/TA-123 add Python and Node.js language skills
fix(api): bugfix/API-456 handle null response from user service
refactor(handlers): feature/TA-789 extract validation logic
docs: main update README with new agent structure
chore(deps): main upgrade aws-sdk to v3
ci: feature/CI-101 add integration test workflow
```
