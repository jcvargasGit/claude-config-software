# Claude Code Configuration

Personal Claude Code configuration files for skills, agents, and settings.

## Structure

```
├── settings.json
├── CLAUDE.md
├── skills/
│   ├── lang-golang/
│   ├── lang-typescript/
│   ├── lang-javascript/
│   ├── lang-python/
│   ├── lang-nodejs/
│   ├── frontend-html/
│   ├── frontend-css/
│   ├── frontend-react/
│   ├── cloud-aws/
│   ├── cloud-cloudformation/
│   ├── cloud-sam/
│   ├── devops-terraform/
│   ├── devops-github-actions/
│   ├── spec-user-stories/
│   ├── spec-acceptance-criteria/
│   ├── spec-prd/
│   ├── spec-api-specs/
│   ├── arch-cloud/
│   └── clean-code/
├── agents/
│   ├── solution-architect.md
│   ├── backend-developer.md
│   └── frontend-engineer.md
├── commands/
│   └── commit.md
└── setup.sh
```

## Skills

Skills are organized by category prefix:

| Prefix | Category | Skills |
|--------|----------|--------|
| `lang-` | Languages | golang, typescript, javascript, python, nodejs |
| `frontend-` | Frontend | html, css, react |
| `cloud-` | Cloud Providers | aws, cloudformation, sam |
| `devops-` | DevOps | terraform, github-actions |
| `spec-` | Specifications | user-stories, acceptance-criteria, prd, api-specs |
| `arch-` | Architecture | cloud |
| (none) | General | clean-code |

## Agents

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| `solution-architect` | Design, trade-offs, decisions | Architecture design, system reviews, technology selection. Does NOT write code. |
| `backend-developer` | Backend implementation | API development, Lambda functions, Terraform, CI/CD. Writes code. |
| `frontend-engineer` | Frontend implementation | React components, UI features, frontend tests. Writes code. |

### Agent Skills

| Agent | Skills |
|-------|--------|
| `solution-architect` | arch-cloud, spec-acceptance-criteria, spec-api-specs, cloud-aws, clean-code |
| `backend-developer` | clean-code, arch-cloud, lang-golang, lang-python, lang-nodejs, devops-terraform, devops-github-actions, cloud-cloudformation, cloud-sam |
| `frontend-engineer` | clean-code, lang-typescript, lang-javascript, frontend-html, frontend-css, frontend-react, spec-acceptance-criteria |

## Commands

| Command | Description |
|---------|-------------|
| `/commit` | Generate semantic commit message and commit staged changes |

## Setup

Run the setup script to symlink configuration files to `~/.claude`:

```bash
./setup.sh
```

## Verify

After running setup, verify the configuration:

```bash
claude /config
```

## License

MIT
