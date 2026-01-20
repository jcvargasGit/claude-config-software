---
name: load-skills
description: Load all available skills into context for the current session
---

# Load All Skills Command

Load all available skills into context so their patterns and checklists are available for the session.

## Instructions

Invoke each of the following skills in sequence to load their patterns into context:

### Language Skills
1. `/lang-golang` - Go best practices
2. `/lang-typescript` - TypeScript patterns
3. `/lang-javascript` - JavaScript/ES6+ patterns
4. `/lang-python` - Python best practices
5. `/lang-nodejs` - Node.js async patterns

### Cloud Skills
6. `/cloud-aws` - AWS service patterns
7. `/cloud-cloudformation` - CloudFormation IaC
8. `/cloud-sam` - SAM serverless patterns

### Architecture Skills
9. `/arch-cloud` - Cloud architecture principles
10. `/arch-hexagonal` - Hexagonal architecture

### DevOps Skills
11. `/devops-terraform` - Terraform IaC patterns
12. `/devops-github-actions` - GitHub Actions CI/CD

### Specification Skills
13. `/spec-user-stories` - User story format
14. `/spec-acceptance-criteria` - BDD Given/When/Then
15. `/spec-prd` - PRD structure
16. `/spec-api-specs` - OpenAPI/REST conventions

### Quality Skills
17. `/clean-code` - Clean Code principles
18. `/testing-serverless` - Serverless testing pyramid

## After Loading

Report which skills were loaded successfully and confirm they are available in context.

**Note:** This will consume significant context. Only use when you need multiple skills for a complex task.
