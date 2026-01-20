---
name: ticket
description: Track ticket progress with done/pending items across sessions
---

# Ticket Tracking Command

Track progress on Jira tickets with persistent done/pending checklists.

## Usage

```
/ticket <TICKET-ID> [action]
```

**Actions:**
- (none) - Show current status or initialize new ticket
- `done <item-number>` - Mark item as done
- `pending <item-number>` - Mark item as pending
- `add` - Add new items to the checklist
- `note` - Add a note to the ticket

## Storage Location

**Global storage with per-repository organization:**

```
~/.claude/tickets/<repo-name>/<TICKET-ID>.md
```

**Symlink in each project:**
```
<project>/docs/tickets -> ~/.claude/tickets/<repo-name>/
```

### Setup (first time per repository)

When running `/ticket` in a new repository:

1. Detect repository name from git remote or directory name:
   ```bash
   # Try git remote first
   REPO_NAME=$(basename -s .git $(git config --get remote.origin.url) 2>/dev/null)
   # Fallback to current directory name
   [ -z "$REPO_NAME" ] && REPO_NAME=$(basename $(pwd))
   ```

2. Create the global tickets directory:
   ```bash
   mkdir -p ~/.claude/tickets/$REPO_NAME
   ```

3. Create symlink in project's docs folder:
   ```bash
   mkdir -p docs
   ln -sfn ~/.claude/tickets/$REPO_NAME docs/tickets
   ```

4. Add `docs/tickets` to `.gitignore` if not already present (symlinks shouldn't be committed)

### Path Resolution

When looking for a ticket:
1. First, determine `REPO_NAME` from git or directory
2. Look in `~/.claude/tickets/$REPO_NAME/<TICKET-ID>.md`
3. The symlink at `docs/tickets` provides convenient local access

## Display Format

When showing ticket status, use this colored format:

```
╔══════════════════════════════════════════════════════════════╗
║  🎫 CCM-2461: Databricks Integration                        ║
╚══════════════════════════════════════════════════════════════╝

📋 Description
   Add Databricks client for commitment queries...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⏳ PENDING (3)
   1. ○ Create PR for databricks-aws-infra
   2. ○ Apply terraform after PR merge
   3. ○ Get Lambda ARN from output

✅ DONE (5)
   ● Create DatabricksClient                    (Jan 12)
   ● Add SQL injection prevention               (Jan 12)
   ● Update terraform configs                   (Jan 12)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Progress: ████████████░░░░ 62% (5/8)

💡 Next: Create PR for databricks-aws-infra changes
```

## Color Scheme (for terminal output in responses)

Use these text indicators for clarity:
- **Title**: Bold with ticket ID
- **Pending items**: Show number prefix for easy reference (1, 2, 3...)
- **Done items**: Show completion date
- **Progress bar**: Visual representation of completion
- **Next step**: Highlight the immediate next action

## File Format

```markdown
# <TICKET-ID>

## Description
<user-provided description>

## Progress

### Pending
- [ ] Item 1
- [ ] Item 2

### Done
- [x] Completed item (completed: YYYY-MM-DD)

## Notes
<any additional context or updates>
```

## Instructions

### First Time Setup (per repository)

Before any ticket operations, ensure the repository is set up:

```bash
# Detect repo name
REPO_NAME=$(basename -s .git $(git config --get remote.origin.url) 2>/dev/null)
[ -z "$REPO_NAME" ] && REPO_NAME=$(basename $(pwd))

# Create global directory
mkdir -p ~/.claude/tickets/$REPO_NAME

# Create symlink (if not exists)
mkdir -p docs
[ ! -L docs/tickets ] && ln -sfn ~/.claude/tickets/$REPO_NAME docs/tickets

# Add to gitignore if needed
grep -q "^docs/tickets$" .gitignore 2>/dev/null || echo "docs/tickets" >> .gitignore
```

### New Ticket
1. Run setup (above) if not already done
2. Ask user for description and checklist items
3. Create file at `~/.claude/tickets/$REPO_NAME/<TICKET-ID>.md`
4. Display initial status

### Existing Ticket
1. Determine `REPO_NAME` and read the ticket file
2. Display formatted status with:
   - Header with ticket ID and title (first line of description)
   - Pending items numbered for easy reference
   - Done items with completion dates
   - Progress bar showing % complete
   - "Next step" suggestion (first pending item)
3. Ask what user wants to do

### Marking Items Done / Adding Items / Adding Notes
**IMPORTANT: Updates must be fast and silent!**

1. Use Bash with sed to update files - this is faster and auto-approved:
```bash
# Get repo name first
REPO_NAME=$(basename -s .git $(git config --get remote.origin.url) 2>/dev/null)
[ -z "$REPO_NAME" ] && REPO_NAME=$(basename $(pwd))

# Mark item done
sed -i '' 's/- \[ \] Item text/- [x] Item text (DATE)/' ~/.claude/tickets/$REPO_NAME/TICKET.md
```

2. Do NOT use Edit tool for ticket updates (requires approval, slow)
3. Do NOT show file contents after updates
4. Only show a brief confirmation: "Marked item X as done"
5. Skip showing the full formatted status unless user asks for it

**Quick update pattern:**
```bash
REPO_NAME=$(basename -s .git $(git config --get remote.origin.url) 2>/dev/null)
[ -z "$REPO_NAME" ] && REPO_NAME=$(basename $(pwd))

# Add note silently
echo "- Note text (DATE)" >> ~/.claude/tickets/$REPO_NAME/TICKET.md && echo "Note added"
```

### Progress Calculation
- Total items = Pending + Done
- Percentage = (Done / Total) * 100
- Bar: █ for complete, ░ for incomplete (16 chars total)

## Smart Suggestions

At the end of status display, include:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 Actions:
   [1] Mark item done    [2] Add item    [3] Add note    [4] Exit
```
