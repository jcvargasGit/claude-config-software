---
name: cloud-cloudformation
description: AWS CloudFormation expert. Covers CFN templates, intrinsic functions (!Ref, !GetAtt), stacks, nested stacks, and drift detection. Use for any CloudFormation question.
model: opus
---

# CloudFormation Skill

Apply these CloudFormation patterns and practices when working with AWS infrastructure as code.

## Additional Resources

- [Intrinsic Functions](./functions.md) - !Ref, !GetAtt, !Sub, conditions
- [Resource Examples](./resources.md) - Lambda, DynamoDB, API Gateway, S3
- [Stack Management](./management.md) - Nested stacks, cross-stack refs, best practices

## Template Structure

### Standard Layout
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Brief description of what this stack creates

Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]
    Default: dev

Mappings:
  EnvironmentConfig:
    dev:
      InstanceType: t3.micro
    prod:
      InstanceType: t3.small

Conditions:
  IsProd: !Equals [!Ref Environment, prod]

Resources:
  # Resources defined here

Outputs:
  ResourceArn:
    Description: ARN of the created resource
    Value: !GetAtt Resource.Arn
    Export:
      Name: !Sub ${AWS::StackName}-ResourceArn
```

## CLI Commands

```bash
# Validate template
aws cloudformation validate-template --template-body file://template.yaml

# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack
```

## Quality Checklist

When writing CloudFormation templates, verify:
- [ ] All resources have appropriate deletion policies
- [ ] Sensitive data uses SSM Parameter Store or Secrets Manager
- [ ] IAM roles follow least privilege
- [ ] Resources are tagged appropriately
- [ ] Outputs export values for cross-stack references
- [ ] Parameters have constraints and descriptions
- [ ] Template validates: `aws cloudformation validate-template`
- [ ] No hardcoded account IDs or regions
