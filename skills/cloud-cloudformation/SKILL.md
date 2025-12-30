---
name: cloud-cloudformation
description: Apply AWS CloudFormation best practices for infrastructure as code, including template structure, intrinsic functions, and stack management.
---

# CloudFormation Skill

Apply these CloudFormation patterns and practices when working with AWS infrastructure as code.

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

## Intrinsic Functions

### Common Functions
```yaml
# Reference parameter or resource
!Ref MyParameter
!Ref MyResource

# Get attribute from resource
!GetAtt MyBucket.Arn
!GetAtt MyBucket.DomainName

# String substitution
!Sub 'arn:aws:s3:::${BucketName}/*'
!Sub
  - 'https://${Domain}/api'
  - Domain: !Ref ApiDomain

# Join strings
!Join ['-', [!Ref Environment, app, bucket]]

# Select from list
!Select [0, !GetAZs '']

# Split string
!Split [',', !Ref SubnetIds]

# Conditionals
!If [IsProd, t3.large, t3.micro]

# Import from other stack
!ImportValue OtherStack-OutputName

# Base64 encode
!Base64 |
  #!/bin/bash
  echo "Hello World"
```

### Condition Functions
```yaml
Conditions:
  IsProd: !Equals [!Ref Environment, prod]
  IsNotProd: !Not [!Equals [!Ref Environment, prod]]
  IsProdOrStaging: !Or
    - !Equals [!Ref Environment, prod]
    - !Equals [!Ref Environment, staging]
  HasBucket: !Not [!Equals [!Ref BucketName, '']]
```

## Resource Patterns

### Lambda Function
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

### DynamoDB Table
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

### API Gateway
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

### S3 Bucket
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

## Stack Management

### Nested Stacks
```yaml
NetworkStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: !Sub https://${TemplateBucket}.s3.amazonaws.com/network.yaml
    Parameters:
      Environment: !Ref Environment
      VpcCidr: !Ref VpcCidr
```

### Cross-Stack References
```yaml
# Exporting stack
Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: !Sub ${AWS::StackName}-VpcId

# Importing stack
Resources:
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !ImportValue NetworkStack-VpcId
```

### Change Sets
```bash
# Create change set
aws cloudformation create-change-set \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --change-set-name my-changes

# Review changes
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-changes

# Execute change set
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-changes
```

## Best Practices

### Deletion Policies
```yaml
# Retain data resources on stack deletion
DynamoTable:
  Type: AWS::DynamoDB::Table
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain

# Snapshot before deletion
RDSInstance:
  Type: AWS::RDS::DBInstance
  DeletionPolicy: Snapshot
```

### Parameter Constraints
```yaml
Parameters:
  InstanceType:
    Type: String
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
    ConstraintDescription: Must be a valid T3 instance type

  Email:
    Type: String
    AllowedPattern: '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    ConstraintDescription: Must be a valid email address

  CidrBlock:
    Type: String
    Default: 10.0.0.0/16
    AllowedPattern: '^(\d{1,3}\.){3}\d{1,3}/\d{1,2}$'
```

### Resource Naming
```yaml
# Use stack name as prefix for uniqueness
BucketName: !Sub ${AWS::StackName}-artifacts

# Include account/region for global uniqueness
BucketName: !Sub ${AWS::StackName}-${AWS::AccountId}-${AWS::Region}
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
