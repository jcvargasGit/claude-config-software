---
name: devops-terraform
description: Terraform expert for *.tf files. Covers HCL, modules, state management, providers, workspaces, and terraform plan/apply. Use for any Terraform question.
model: opus
---

# Terraform Skill

Apply these Terraform patterns and practices when working with infrastructure as code.

## Code Style

- Use `terraform fmt` formatting
- Use snake_case for resource names and variables
- Use meaningful resource names that describe purpose
- Keep resources focused - one logical resource per block
- Use locals for computed values and reducing repetition
- Maximum 2 parameters inline, use variable blocks for complex configurations

## Project Structure

```
infrastructure/
├── modules/
│   ├── lambda/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── api-gateway/
│   └── dynamodb/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── backend.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
└── shared/
    └── state-bucket/
```

## Module Design

```hcl
# variables.tf - Input validation
variable "environment" {
  type        = string
  description = "Deployment environment"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "lambda_config" {
  type = object({
    memory_size = number
    timeout     = number
    runtime     = string
  })
  default = {
    memory_size = 256
    timeout     = 30
    runtime     = "provided.al2023"
  }
}

# main.tf - Resources
resource "aws_lambda_function" "this" {
  function_name = "${var.service_name}-${var.environment}"
  role          = aws_iam_role.lambda.arn
  handler       = "bootstrap"
  runtime       = var.lambda_config.runtime
  memory_size   = var.lambda_config.memory_size
  timeout       = var.lambda_config.timeout

  filename         = var.deployment_package
  source_code_hash = filebase64sha256(var.deployment_package)

  environment {
    variables = var.environment_variables
  }

  tags = local.common_tags
}

# outputs.tf - Expose needed values
output "function_arn" {
  value       = aws_lambda_function.this.arn
  description = "Lambda function ARN"
}

output "function_name" {
  value       = aws_lambda_function.this.function_name
  description = "Lambda function name"
}
```

## State Management

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "services/auth/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# Use workspaces for environment isolation
# terraform workspace new dev
# terraform workspace select prod
```

## Resource Patterns

### Tagging Strategy
```hcl
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "terraform"
    Service     = var.service_name
  }
}

resource "aws_lambda_function" "this" {
  # ... config
  tags = merge(local.common_tags, var.additional_tags)
}
```

### IAM Least Privilege
```hcl
data "aws_iam_policy_document" "lambda_assume" {
  statement {
    actions = ["sts:AssumeRole"]
    principals {
      type        = "Service"
      identifiers = ["lambda.amazonaws.com"]
    }
  }
}

data "aws_iam_policy_document" "lambda_permissions" {
  statement {
    sid    = "DynamoDBAccess"
    effect = "Allow"
    actions = [
      "dynamodb:GetItem",
      "dynamodb:PutItem",
      "dynamodb:Query"
    ]
    resources = [aws_dynamodb_table.this.arn]
  }

  statement {
    sid    = "CloudWatchLogs"
    effect = "Allow"
    actions = [
      "logs:CreateLogStream",
      "logs:PutLogEvents"
    ]
    resources = ["${aws_cloudwatch_log_group.lambda.arn}:*"]
  }
}
```

### Data Sources Over Hardcoding
```hcl
data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

locals {
  account_id = data.aws_caller_identity.current.account_id
  region     = data.aws_region.current.name
}
```

## Environment Configuration

```hcl
# environments/prod/terraform.tfvars
environment = "prod"

lambda_config = {
  memory_size = 512
  timeout     = 30
  runtime     = "provided.al2023"
}

# Use tfvars for environment-specific values
# Use variables with defaults for optional config
# Use locals for computed/derived values
```

## Common Patterns

### Conditional Resources
```hcl
resource "aws_cloudwatch_metric_alarm" "errors" {
  count = var.enable_alarms ? 1 : 0

  alarm_name          = "${var.service_name}-errors"
  comparison_operator = "GreaterThanThreshold"
  # ... config
}
```

### Dynamic Blocks
```hcl
resource "aws_security_group" "this" {
  name   = "${var.service_name}-sg"
  vpc_id = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

### Module Composition
```hcl
module "auth_lambda" {
  source = "../../modules/lambda"

  service_name  = "auth"
  environment   = var.environment
  lambda_config = var.lambda_config

  environment_variables = {
    TABLE_NAME = module.auth_table.table_name
  }
}

module "auth_table" {
  source = "../../modules/dynamodb"

  table_name  = "auth-users"
  environment = var.environment
  hash_key    = "user_id"
}
```

## Quality Checklist

When writing Terraform code, verify:
- [ ] State backend configured with locking
- [ ] Variables have descriptions and validation
- [ ] Resources use consistent tagging
- [ ] IAM follows least privilege principle
- [ ] Sensitive values marked as sensitive
- [ ] Outputs documented and exposed
- [ ] No hardcoded values (use variables/data sources)
- [ ] Resource dependencies explicit when needed
