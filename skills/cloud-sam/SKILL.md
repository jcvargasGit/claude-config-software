---
name: cloud-sam
description: Apply AWS SAM (Serverless Application Model) best practices for building, testing, and deploying serverless applications.
---

# AWS SAM Skill

Apply these AWS SAM patterns and practices when building serverless applications.

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

## SAM Resource Types

### Serverless Function
```yaml
GetUserFunction:
  Type: AWS::Serverless::Function
  Properties:
    FunctionName: !Sub ${AWS::StackName}-get-user
    CodeUri: ./cmd/get-user/
    Handler: bootstrap
    Description: Retrieves user by ID
    Events:
      GetUser:
        Type: HttpApi
        Properties:
          Path: /users/{id}
          Method: GET
          ApiId: !Ref HttpApi
    Policies:
      - DynamoDBReadPolicy:
          TableName: !Ref UsersTable
    Environment:
      Variables:
        TABLE_NAME: !Ref UsersTable

Metadata:
  BuildMethod: go1.x
```

### HTTP API (API Gateway v2)
```yaml
HttpApi:
  Type: AWS::Serverless::HttpApi
  Properties:
    StageName: !Ref Environment
    CorsConfiguration:
      AllowOrigins:
        - '*'
      AllowMethods:
        - GET
        - POST
        - PUT
        - DELETE
      AllowHeaders:
        - Content-Type
        - Authorization
    AccessLogSettings:
      DestinationArn: !GetAtt ApiLogGroup.Arn
      Format: '{"requestId":"$context.requestId","ip":"$context.identity.sourceIp","method":"$context.httpMethod","path":"$context.path","status":"$context.status","latency":"$context.integrationLatency"}'
```

### REST API (API Gateway v1)
```yaml
RestApi:
  Type: AWS::Serverless::Api
  Properties:
    StageName: !Ref Environment
    TracingEnabled: true
    Auth:
      DefaultAuthorizer: CognitoAuthorizer
      Authorizers:
        CognitoAuthorizer:
          UserPoolArn: !GetAtt UserPool.Arn
```

### DynamoDB Table
```yaml
UsersTable:
  Type: AWS::Serverless::SimpleTable
  Properties:
    TableName: !Sub ${AWS::StackName}-users
    PrimaryKey:
      Name: pk
      Type: String
    ProvisionedThroughput:
      ReadCapacityUnits: 5
      WriteCapacityUnits: 5

# For more complex tables, use AWS::DynamoDB::Table
OrdersTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: !Sub ${AWS::StackName}-orders
    BillingMode: PAY_PER_REQUEST
    AttributeDefinitions:
      - AttributeName: pk
        AttributeType: S
      - AttributeName: sk
        AttributeType: S
    KeySchema:
      - AttributeName: pk
        KeyType: HASH
      - AttributeName: sk
        KeyType: RANGE
```

### Step Functions
```yaml
OrderWorkflow:
  Type: AWS::Serverless::StateMachine
  Properties:
    DefinitionUri: statemachine/order-workflow.asl.json
    DefinitionSubstitutions:
      ValidateOrderFunctionArn: !GetAtt ValidateOrderFunction.Arn
      ProcessPaymentFunctionArn: !GetAtt ProcessPaymentFunction.Arn
    Policies:
      - LambdaInvokePolicy:
          FunctionName: !Ref ValidateOrderFunction
      - LambdaInvokePolicy:
          FunctionName: !Ref ProcessPaymentFunction
```

### SQS Queue with Lambda Trigger
```yaml
ProcessQueue:
  Type: AWS::SQS::Queue
  Properties:
    QueueName: !Sub ${AWS::StackName}-process-queue
    VisibilityTimeout: 180
    RedrivePolicy:
      deadLetterTargetArn: !GetAtt DeadLetterQueue.Arn
      maxReceiveCount: 3

ProcessQueueFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: ./cmd/process-queue/
    Handler: bootstrap
    Events:
      SQSEvent:
        Type: SQS
        Properties:
          Queue: !GetAtt ProcessQueue.Arn
          BatchSize: 10
          FunctionResponseTypes:
            - ReportBatchItemFailures
```

### EventBridge Rule
```yaml
ScheduledFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: ./cmd/scheduled/
    Handler: bootstrap
    Events:
      ScheduleEvent:
        Type: Schedule
        Properties:
          Schedule: rate(1 hour)
          Description: Runs every hour
          Enabled: true
```

## SAM Policy Templates

### Common Policies
```yaml
# DynamoDB
Policies:
  - DynamoDBCrudPolicy:
      TableName: !Ref MyTable
  - DynamoDBReadPolicy:
      TableName: !Ref MyTable
  - DynamoDBWritePolicy:
      TableName: !Ref MyTable

# S3
Policies:
  - S3ReadPolicy:
      BucketName: !Ref MyBucket
  - S3WritePolicy:
      BucketName: !Ref MyBucket
  - S3CrudPolicy:
      BucketName: !Ref MyBucket

# SQS
Policies:
  - SQSSendMessagePolicy:
      QueueName: !GetAtt MyQueue.QueueName
  - SQSPollerPolicy:
      QueueName: !GetAtt MyQueue.QueueName

# SNS
Policies:
  - SNSPublishMessagePolicy:
      TopicName: !GetAtt MyTopic.TopicName

# Secrets Manager
Policies:
  - AWSSecretsManagerGetSecretValuePolicy:
      SecretArn: !Ref MySecret

# Step Functions
Policies:
  - StepFunctionsExecutionPolicy:
      StateMachineName: !GetAtt MyStateMachine.Name
```

## SAM CLI Commands

### Build and Deploy
```bash
# Build the application
sam build

# Build with specific template
sam build --template-file template.yaml

# Build for container-based deployment
sam build --use-container

# Deploy with guided prompts (first time)
sam deploy --guided

# Deploy with saved config
sam deploy --config-file samconfig.toml

# Deploy to specific environment
sam deploy --config-env prod
```

### Local Development
```bash
# Start local API
sam local start-api --port 3000

# Invoke function locally
sam local invoke GetUserFunction --event events/get-user.json

# Generate sample event
sam local generate-event apigateway http-api-proxy > events/api-event.json

# Start Lambda locally for debugging
sam local start-lambda

# Run with environment variables
sam local invoke --env-vars env.json
```

### Logs and Debugging
```bash
# Tail function logs
sam logs --name GetUserFunction --stack-name my-stack --tail

# Fetch logs for time range
sam logs --name GetUserFunction --start-time '5min ago'

# Trace requests
sam traces --stack-name my-stack
```

### Sync for Development
```bash
# Start sync for rapid iteration
sam sync --stack-name my-stack --watch

# Sync code only (skip infra changes)
sam sync --code --stack-name my-stack
```

## samconfig.toml

```toml
version = 0.1

[default]
[default.global.parameters]
stack_name = "my-app"

[default.build.parameters]
cached = true
parallel = true

[default.deploy.parameters]
capabilities = "CAPABILITY_IAM CAPABILITY_AUTO_EXPAND"
confirm_changeset = true
resolve_s3 = true

[default.sync.parameters]
watch = true

[dev]
[dev.deploy.parameters]
stack_name = "my-app-dev"
parameter_overrides = "Environment=dev"
confirm_changeset = false

[prod]
[prod.deploy.parameters]
stack_name = "my-app-prod"
parameter_overrides = "Environment=prod"
confirm_changeset = true
```

## Project Structure

```
project/
├── cmd/
│   ├── get-user/
│   │   └── main.go
│   ├── create-user/
│   │   └── main.go
│   └── process-queue/
│       └── main.go
├── internal/
│   ├── domain/
│   ├── handler/
│   └── repository/
├── events/
│   └── get-user.json
├── statemachine/
│   └── workflow.asl.json
├── template.yaml
├── samconfig.toml
├── Makefile
└── README.md
```

## Makefile for SAM

```makefile
.PHONY: build deploy test local clean

build:
	sam build

deploy: build
	sam deploy

deploy-prod: build
	sam deploy --config-env prod

local:
	sam local start-api --port 3000

invoke:
	sam local invoke $(FUNCTION) --event events/$(EVENT).json

logs:
	sam logs --name $(FUNCTION) --stack-name $(STACK) --tail

sync:
	sam sync --stack-name $(STACK) --watch

clean:
	rm -rf .aws-sam

test:
	go test ./...
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
