# CloudFormation Intrinsic Functions

## Common Functions

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

## Condition Functions

```yaml
Conditions:
  IsProd: !Equals [!Ref Environment, prod]
  IsNotProd: !Not [!Equals [!Ref Environment, prod]]
  IsProdOrStaging: !Or
    - !Equals [!Ref Environment, prod]
    - !Equals [!Ref Environment, staging]
  HasBucket: !Not [!Equals [!Ref BucketName, '']]
```

## Using Conditions in Resources

```yaml
Resources:
  ProdOnlyResource:
    Type: AWS::S3::Bucket
    Condition: IsProd
    Properties:
      BucketName: !Sub ${AWS::StackName}-prod-bucket

  ConditionalProperty:
    Type: AWS::Lambda::Function
    Properties:
      MemorySize: !If [IsProd, 1024, 256]
      Timeout: !If [IsProd, 30, 10]
```
