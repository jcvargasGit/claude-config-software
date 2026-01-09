# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code configuration repository containing reusable skills, agents, and commands that get symlinked to `~/.claude` for use across projects. The configurations define specialized AI behaviors for software development workflows.

## Setup

```bash
./setup.sh  # Symlinks configs to ~/.claude
claude /config  # Verify configuration
```

## Repository Structure

```
skills/           # Specialized knowledge modules (SKILL.md files)
  lang-*          # Language skills: golang, typescript, javascript, python, nodejs
  frontend-*      # Frontend: html, css, react
  cloud-*         # Cloud: aws, cloudformation, sam
  devops-*        # DevOps: terraform, github-actions
  spec-*          # Specs: user-stories, acceptance-criteria, prd, api-specs
  arch-*          # Architecture: cloud, hexagonal

agents/           # Agent definitions with skill compositions
  solution-architect.md   # Design/advisory only, NO code
  backend-developer.md    # Backend implementation
  frontend-engineer.md    # Frontend implementation
  test-engineer.md        # Testing

commands/         # Custom slash commands
  commit.md       # /commit - semantic commit messages
```

## File Format

### Skills (`skills/*/SKILL.md`)
```yaml
---
name: skill-name
description: When to apply this skill
---
# Content with patterns, examples, checklists
```

### Agents (`agents/*.md`)
```yaml
---
name: agent-name
description: When to use this agent
model: opus
skills:
  - skill-1
  - skill-2
---
# Agent instructions with USE/DO NOT USE guidance
```

### Commands (`commands/*.md`)
```yaml
---
name: command-name
description: What the command does
---
# Command instructions
```

## Naming Conventions

- Skills: `{category}-{name}` (e.g., `lang-golang`, `arch-hexagonal`)
- Agents: Role-based names (e.g., `backend-developer`, `solution-architect`)
- Commands: Action-based names (e.g., `commit`)

## Commit Messages

Use semantic format: `type(scope): description`

Types: `feat`, `fix`, `refactor`, `docs`, `chore`, `test`

Example: `feat(skills): add lang-rust skill for Rust development`
