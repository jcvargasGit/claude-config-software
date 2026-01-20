---
name: cloud-sam
description: AWS SAM expert for serverless. Covers template.yaml, sam build/deploy/local, Lambda functions, and API Gateway. Use for any SAM question.
model: opus
---

# AWS SAM Skill

Apply these AWS SAM patterns and practices when building serverless applications.

## Additional Resources

- [SAM Resource Types](./resources.md) - Functions, APIs, DynamoDB, SQS, Step Functions
- [SAM CLI Commands](./cli.md) - Build, deploy, local development, sync
- [Examples & Templates](./examples.md) - Project structure, Makefile, samconfig.toml

## SAM Template Structure

### Basic Template
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: SAM template for serverless application

Globals:
  Function:
    Timeout: 30
    MemorySize: 256
    Runtime: provided.al2023
    Architectures:
      - arm64
    Tracing: Active
    Environment:
      Variables:
        LOG_LEVEL: INFO

Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]
    Default: dev

Resources:
  # Resources here

Outputs:
  ApiEndpoint:
    Description: API Gateway endpoint URL
    Value: !Sub https://${ServerlessApi}.execute-api.${AWS::Region}.amazonaws.com/${Environment}
```

## Globals Configuration

```yaml
Globals:
  Function:
    Timeout: 30
    MemorySize: 256
    Runtime: provided.al2023
    Architectures:
      - arm64
    Tracing: Active
    LoggingConfig:
      LogFormat: JSON
      ApplicationLogLevel: INFO
      SystemLogLevel: WARN
    Environment:
      Variables:
        ENVIRONMENT: !Ref Environment
        LOG_LEVEL: !If [IsProd, WARN, DEBUG]

  HttpApi:
    Auth:
      EnableIamAuthorizer: true
    CorsConfiguration:
      AllowOrigins:
        - '*'
      AllowMethods:
        - GET
        - POST
        - PUT
        - DELETE
```

## Quality Checklist

When building SAM applications, verify:
- [ ] Globals configured to reduce repetition
- [ ] SAM policy templates used instead of inline policies
- [ ] Local testing works with `sam local invoke`
- [ ] Events folder has sample events for testing
- [ ] samconfig.toml has environment-specific configs
- [ ] Dead letter queues configured for async functions
- [ ] Tracing enabled for debugging
- [ ] Proper error handling and logging
- [ ] Function timeouts appropriate for workload
- [ ] Memory sized appropriately (affects CPU allocation)
