# CloudFormation Resource Examples

## Lambda Function

```yaml
LambdaFunction:
  Type: AWS::Lambda::Function
  Properties:
    FunctionName: !Sub ${AWS::StackName}-handler
    Runtime: provided.al2023
    Handler: bootstrap
    Code:
      S3Bucket: !Ref ArtifactBucket
      S3Key: !Ref ArtifactKey
    MemorySize: 256
    Timeout: 30
    Role: !GetAtt LambdaRole.Arn
    Environment:
      Variables:
        TABLE_NAME: !Ref DynamoTable
    TracingConfig:
      Mode: Active

LambdaRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
    Policies:
      - PolicyName: DynamoAccess
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - dynamodb:GetItem
                - dynamodb:PutItem
                - dynamodb:Query
              Resource: !GetAtt DynamoTable.Arn
```

## DynamoDB Table

```yaml
DynamoTable:
  Type: AWS::DynamoDB::Table
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain
  Properties:
    TableName: !Sub ${AWS::StackName}-table
    BillingMode: PAY_PER_REQUEST
    AttributeDefinitions:
      - AttributeName: pk
        AttributeType: S
      - AttributeName: sk
        AttributeType: S
      - AttributeName: gsi1pk
        AttributeType: S
    KeySchema:
      - AttributeName: pk
        KeyType: HASH
      - AttributeName: sk
        KeyType: RANGE
    GlobalSecondaryIndexes:
      - IndexName: gsi1
        KeySchema:
          - AttributeName: gsi1pk
            KeyType: HASH
          - AttributeName: sk
            KeyType: RANGE
        Projection:
          ProjectionType: ALL
    PointInTimeRecoverySpecification:
      PointInTimeRecoveryEnabled: true
    SSESpecification:
      SSEEnabled: true
```

## API Gateway (HTTP API)

```yaml
HttpApi:
  Type: AWS::ApiGatewayV2::Api
  Properties:
    Name: !Sub ${AWS::StackName}-api
    ProtocolType: HTTP
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

ApiStage:
  Type: AWS::ApiGatewayV2::Stage
  Properties:
    ApiId: !Ref HttpApi
    StageName: $default
    AutoDeploy: true
    AccessLogSettings:
      DestinationArn: !GetAtt ApiLogGroup.Arn
      Format: '{"requestId":"$context.requestId","ip":"$context.identity.sourceIp","method":"$context.httpMethod","path":"$context.path","status":"$context.status"}'
```

## S3 Bucket

```yaml
Bucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub ${AWS::StackName}-${AWS::AccountId}-${AWS::Region}
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: AES256
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
    VersioningConfiguration:
      Status: Enabled
```
