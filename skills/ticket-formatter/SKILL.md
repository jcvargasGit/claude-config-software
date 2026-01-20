---
name: ticket-formatter
description: Format project tickets, tasks, and work items as beautiful dark-themed HTML files for easy reading. Use this skill when creating or updating ticket files (like LN-001.md, TASK-123.md), work items, sprint tasks, or any project tracking documents. Triggers on requests to "create a ticket", "format this task", "make this readable", or when writing files that follow ticket/task patterns with checkboxes, progress tracking, or status fields.
---

# Ticket Formatter

Create visually appealing, dark-themed HTML ticket views that are easy to scan and track.

## When to Use

- Creating new tickets/tasks (e.g., `LN-001`, `TASK-123`)
- Converting markdown task files to readable HTML
- Any work item with checkboxes, progress, or status tracking
- User asks for "readable format" or "nice view" of a ticket

## Output Format

Always output as `.html` files with the template structure from `assets/ticket-template.html`.

## Key Elements

1. **Header**: Ticket ID badge (with emoji), title, description, context box
2. **Progress**: Visual progress bar with "X of Y tasks complete"
3. **Task Groups**: Separate "Pending" and "Completed" sections
4. **Checkboxes**: Visual checkbox styling (empty vs checked with ✓)
5. **Dates**: Show completion dates on done items
6. **Notes**: Highlighted notes section with code formatting
7. **Docs**: Documentation references with file icons

## Styling Rules

- Dark theme: `#0d1117` background, `#161b22` cards
- Accent color: `#f7931a` (Bitcoin orange) for Lightning/crypto, or `#58a6ff` for general
- Success green: `#3fb950` for completed items
- Muted text: `#8b949e` for secondary info
- Border color: `#30363d`

## Badge Colors by Type

| Type | Color | Emoji |
|------|-------|-------|
| Lightning/Bitcoin | `#f7931a` | ⚡ |
| Bug/Fix | `#da3633` | 🐛 |
| Feature | `#238636` | ✨ |
| Documentation | `#58a6ff` | 📚 |
| Refactor | `#a371f7` | 🔧 |

## Template Usage

1. Copy structure from `assets/ticket-template.html`
2. Replace placeholder content with actual ticket data
3. Calculate progress percentage: `(done_count / total_count) * 100`
4. Group tasks into pending vs completed
5. Add completion dates where available

## Example Conversion

**Input (Markdown):**
```markdown
# TASK-001
## Description
Fix login bug

## Progress
- [x] Reproduce bug (completed: 2024-01-15)
- [ ] Write fix
- [ ] Add tests
```

**Output:** HTML file with:
- `TASK-001` badge with 🐛 emoji
- Progress bar at 33%
- "1 of 3 tasks complete"
- Completed task in green section with date
- Pending tasks in main section
