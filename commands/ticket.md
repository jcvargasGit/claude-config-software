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

**Primary location (tickets-tracker repo):**

```
/Users/jonathandiaz/projects/tickets-tracker/<org>/<TICKET-ID>/ticket.md
```

**Organization mapping:**

| Working Directory | Org Folder | Ticket Prefix |
|-------------------|------------|---------------|
| `/Users/jonathandiaz/flexera/*` | `flexera/` | CCM-XXXX |
| `/Users/jonathandiaz/transnetwork/*` | `transnetwork/` | TN-XXXX |
| `/Users/jonathandiaz/playvox/*` | `playvox/` | PV-XXXX |
| `/Users/jonathandiaz/digitalp2p/*` | `digitalp2p/` | LN-XXXX |

**Session logs:**

```
/Users/jonathandiaz/projects/tickets-tracker/<org>/<TICKET-ID>/sessions/YYYY-MM-DD-summary.md
```

### Organization Detection

Detect organization from working directory:

```bash
# Detect org from path
case "$(pwd)" in
  /Users/jonathandiaz/flexera/*) ORG="flexera" ;;
  /Users/jonathandiaz/transnetwork/*) ORG="transnetwork" ;;
  /Users/jonathandiaz/playvox/*) ORG="playvox" ;;
  /Users/jonathandiaz/digitalp2p/*) ORG="digitalp2p" ;;
  *) ORG="flexera" ;;  # Default
esac
```

### Path Resolution

When looking for a ticket:
1. Detect organization from current working directory
2. Look in `/Users/jonathandiaz/projects/tickets-tracker/$ORG/<TICKET-ID>/ticket.md`
3. Session logs in `/Users/jonathandiaz/projects/tickets-tracker/$ORG/<TICKET-ID>/sessions/`

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

### Path Constants

```bash
TICKETS_REPO="/Users/jonathandiaz/projects/tickets-tracker"
```

### ALWAYS Push After Changes (NON-NEGOTIABLE)

**Every time the ticket file is created or updated, ALWAYS commit and push to the tracker repo.**

```bash
TICKETS_REPO="/Users/jonathandiaz/projects/tickets-tracker"
cd $TICKETS_REPO && git add $ORG/$TICKET_ID/ && git commit -m "Update $TICKET_ID: <short description>" && git push origin master
```

**Rules:**
- Push after creating a new ticket
- Push after marking items done/pending
- Push after adding notes
- Push after adding session logs
- Push after ANY modification to ticket files
- The tracker repo is on GitHub (`jonthdiaz/tickets-tracker`) - changes must be visible to the team

### New Ticket
1. Detect organization from working directory
2. Ask user for description and checklist items
3. Create directory: `$TICKETS_REPO/$ORG/<TICKET-ID>/`
4. Create file: `$TICKETS_REPO/$ORG/<TICKET-ID>/ticket.md`
5. Create sessions dir: `$TICKETS_REPO/$ORG/<TICKET-ID>/sessions/`
6. Commit and push: `cd $TICKETS_REPO && git add $ORG/$TICKET_ID/ && git commit -m "Add $TICKET_ID: <title>" && git push origin master`
7. Display initial status

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
TICKETS_REPO="/Users/jonathandiaz/projects/tickets-tracker"
# Detect org
case "$(pwd)" in
  /Users/jonathandiaz/flexera/*) ORG="flexera" ;;
  /Users/jonathandiaz/transnetwork/*) ORG="transnetwork" ;;
  /Users/jonathandiaz/playvox/*) ORG="playvox" ;;
  /Users/jonathandiaz/digitalp2p/*) ORG="digitalp2p" ;;
  *) ORG="flexera" ;;
esac

# Mark item done
sed -i '' 's/- \[ \] Item text/- [x] Item text (DATE)/' $TICKETS_REPO/$ORG/TICKET-ID/ticket.md
```

2. Do NOT use Edit tool for ticket updates (requires approval, slow)
3. Do NOT show file contents after updates
4. Only show a brief confirmation: "Marked item X as done"
5. Skip showing the full formatted status unless user asks for it

**Quick update pattern:**
```bash
TICKETS_REPO="/Users/jonathandiaz/projects/tickets-tracker"
ORG="flexera"  # or detect from pwd

# Add note silently
echo "- Note text (DATE)" >> $TICKETS_REPO/$ORG/TICKET-ID/ticket.md && echo "Note added"
```

### Progress Calculation
- Total items = Pending + Done
- Percentage = (Done / Total) * 100
- Bar: █ for complete, ░ for incomplete (16 chars total)

## Smart Suggestions

At the end of status display, include:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Actions:
   [1] Mark item done    [2] Add item    [3] Add note    [4] Exit
```

---

## TICKET CONTEXT BANNER (REQUIRED)

When `/ticket` is invoked, show this banner at the START of EVERY response:

```
┌─────────────────────────────────────────────────────────────┐
│  TICKET: <TICKET-ID> | Status: <STATUS>                    │
│  Blockers: <N> | Progress: <done>/<total> (<percent>%)     │
└─────────────────────────────────────────────────────────────┘
```

**Rules:**
1. Show at TOP of every response (before any other content)
2. Update counts as items complete or blockers resolve
3. Keeps both user and Claude aware of context
4. Parse ticket file to get current counts

---

## AUTO-TRACKING MODE (CRITICAL)

When `/ticket` is invoked, enable **auto-tracking** for the rest of the conversation:

### What to Track Automatically

| Event | Action |
|-------|--------|
| Checklist item completed | Mark `[x]` in ticket file |
| Blocker resolved | Update status from BLOCKING to RESOLVED |
| Key decision made | Add to Session Log |
| Architecture finding | Add to Architecture section |
| Code written/modified | Note in Session Log |
| Open question answered | Update the Blockers table |

### Session Log Format

Append to ticket at end of work session or before context clear:

```markdown
## Session Log

### Session: YYYY-MM-DD HH:MM

**Completed:**
- [x] What was done

**Findings:**
- Discovery 1
- Discovery 2

**Decisions:**
- Decision and rationale

**Next Steps:**
- What to do next

---
```

### Why Auto-Track?

1. **Context survives reset** - Ticket file preserves all progress
2. **Resume anywhere** - Next session starts with full context
3. **Documentation** - Automatic record of work done
4. **Team visibility** - Others can see progress

### Implementation

Use Bash with sed/echo for fast updates (no approval needed):

```bash
TICKETS_REPO="/Users/jonathandiaz/projects/tickets-tracker"
ORG="flexera"  # or detect from pwd
TICKET_ID="CCM-3474"

# Create session log
SESSION_FILE="$TICKETS_REPO/$ORG/$TICKET_ID/sessions/$(date '+%Y-%m-%d')-session.md"
echo "# Session: $(date '+%Y-%m-%d %H:%M')" >> $SESSION_FILE

# Mark item done in ticket
sed -i '' 's/- \[ \] Item text/- [x] Item text/' $TICKETS_REPO/$ORG/$TICKET_ID/ticket.md
```
