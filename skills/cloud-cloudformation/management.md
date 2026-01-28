# CloudFormation Stack Management

## Nested Stacks

```yaml
NetworkStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: !Sub https://${TemplateBucket}.s3.amazonaws.com/network.yaml
    Parameters:
      Environment: !Ref Environment
      VpcCidr: !Ref VpcCidr
```

## Cross-Stack References

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

## Change Sets

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

## Deletion Policies

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

## Parameter Constraints

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

## Resource Naming

```yaml
# Use stack name as prefix for uniqueness
BucketName: !Sub ${AWS::StackName}-artifacts

# Include account/region for global uniqueness
BucketName: !Sub ${AWS::StackName}-${AWS::AccountId}-${AWS::Region}
```
