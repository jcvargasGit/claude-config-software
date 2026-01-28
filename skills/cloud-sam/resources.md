# SAM Resource Types

## Serverless Function
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

## HTTP API (API Gateway v2)
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

## REST API (API Gateway v1)
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

## DynamoDB Table
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

## Step Functions
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

## SQS Queue with Lambda Trigger
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

## EventBridge Rule
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
