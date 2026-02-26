# AWS CloudFormation

AWS CloudFormation provides infrastructure as code, allowing you to model and provision AWS resources using templates.

## What is CloudFormation?

CloudFormation enables you to:
- Define infrastructure as code (YAML or JSON)
- Version control your infrastructure
- Automate resource provisioning
- Ensure consistent deployments
- Manage dependencies automatically
- Roll back changes if needed

## Key Concepts

### Templates
- JSON or YAML text files
- Describe AWS resources and configurations
- Reusable and version-controlled

### Stacks
- Collection of AWS resources
- Created from a template
- Managed as a single unit

### Change Sets
- Preview changes before execution
- Review impact of updates
- Prevent unwanted changes

### Stack Sets
- Deploy stacks across multiple accounts/regions
- Centralized management
- Consistent configurations

## Template Structure

### YAML Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Description of what this template creates'

Parameters:
  ParameterName:
    Type: String
    Description: Parameter description

Resources:
  ResourceLogicalID:
    Type: AWS::Service::ResourceType
    Properties:
      PropertyName: value

Outputs:
  OutputName:
    Description: Output description
    Value: !Ref ResourceLogicalID
```

### Template Sections

1. **AWSTemplateFormatVersion**: Template version (optional)
2. **Description**: Template description (optional)
3. **Parameters**: Input values (optional)
4. **Mappings**: Static variables (optional)
5. **Conditions**: Control resource creation (optional)
6. **Resources**: AWS resources (required)
7. **Outputs**: Return values (optional)

## Getting Started

### 1. Simple S3 Bucket Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Create an S3 bucket'

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-cloudformation-bucket-12345
      VersioningConfiguration:
        Status: Enabled
      Tags:
        - Key: Environment
          Value: Development

Outputs:
  BucketName:
    Description: Name of the S3 bucket
    Value: !Ref MyBucket
  BucketArn:
    Description: ARN of the S3 bucket
    Value: !GetAtt MyBucket.Arn
```

### 2. Create Stack

```bash
# Create stack from template file
aws cloudformation create-stack \
    --stack-name my-s3-stack \
    --template-body file://s3-bucket.yaml

# Create stack from URL
aws cloudformation create-stack \
    --stack-name my-stack \
    --template-url https://s3.amazonaws.com/bucket/template.yaml

# Wait for stack creation
aws cloudformation wait stack-create-complete \
    --stack-name my-s3-stack

# Describe stack
aws cloudformation describe-stacks \
    --stack-name my-s3-stack
```

## EC2 Instance Example

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Launch EC2 instance with security group'

Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 Key Pair for SSH access
  
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
    Description: EC2 instance type

Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    us-west-2:
      AMI: ami-0d1cd67c26f5fca19

Resources:
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH and HTTP access
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
      SecurityGroupIds:
        - !Ref MySecurityGroup
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "<h1>Hello from CloudFormation!</h1>" > /var/www/html/index.html
      Tags:
        - Key: Name
          Value: MyWebServer

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref MyEC2Instance
  
  PublicIP:
    Description: Public IP address
    Value: !GetAtt MyEC2Instance.PublicIp
  
  WebsiteURL:
    Description: Website URL
    Value: !Sub 'http://${MyEC2Instance.PublicDnsName}'
```

Deploy with parameters:

```bash
aws cloudformation create-stack \
    --stack-name web-server-stack \
    --template-body file://ec2-webserver.yaml \
    --parameters \
        ParameterKey=KeyName,ParameterValue=my-key-pair \
        ParameterKey=InstanceType,ParameterValue=t2.micro
```

## VPC with Public/Private Subnets

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'VPC with public and private subnets'

Parameters:
  VpcCIDR:
    Type: String
    Default: 10.0.0.0/16
    Description: CIDR block for VPC

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-VPC'

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-IGW'

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-Public-Subnet'

  PrivateSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-Private-Subnet'

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-Public-RT'

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VPC-ID'
  
  PublicSubnetId:
    Description: Public Subnet ID
    Value: !Ref PublicSubnet
    Export:
      Name: !Sub '${AWS::StackName}-Public-Subnet-ID'
```

## Stack Operations

### Update Stack

```bash
# Create change set
aws cloudformation create-change-set \
    --stack-name my-stack \
    --change-set-name my-changes \
    --template-body file://updated-template.yaml

# View change set
aws cloudformation describe-change-set \
    --stack-name my-stack \
    --change-set-name my-changes

# Execute change set
aws cloudformation execute-change-set \
    --stack-name my-stack \
    --change-set-name my-changes

# Or update directly
aws cloudformation update-stack \
    --stack-name my-stack \
    --template-body file://updated-template.yaml
```

### Delete Stack

```bash
# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Wait for deletion
aws cloudformation wait stack-delete-complete --stack-name my-stack
```

### List Stacks

```bash
# List all stacks
aws cloudformation list-stacks

# List active stacks
aws cloudformation list-stacks \
    --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

# Describe stack resources
aws cloudformation describe-stack-resources \
    --stack-name my-stack

# Get stack outputs
aws cloudformation describe-stacks \
    --stack-name my-stack \
    --query 'Stacks[0].Outputs'
```

## Intrinsic Functions

### Ref
```yaml
!Ref ResourceLogicalID
# Returns resource ID or parameter value
```

### GetAtt
```yaml
!GetAtt ResourceLogicalID.AttributeName
# Returns resource attribute
```

### Sub
```yaml
!Sub 'String with ${Variable}'
# Substitutes variables
```

### Join
```yaml
!Join ['delimiter', [list, of, values]]
# Joins strings
```

### Select
```yaml
!Select [index, list]
# Selects item from list
```

### Split
```yaml
!Split [delimiter, string]
# Splits string into list
```

### GetAZs
```yaml
!GetAZs region
# Returns availability zones
```

### If
```yaml
!If [ConditionName, ValueIfTrue, ValueIfFalse]
# Conditional value
```

## Conditions

```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, 'production']
  CreateProdResources: !And
    - !Equals [!Ref Environment, 'production']
    - !Equals [!Ref Region, 'us-east-1']

Resources:
  ProdOnlyResource:
    Type: AWS::S3::Bucket
    Condition: IsProduction
    Properties:
      BucketName: production-bucket
```

## Cross-Stack References

### Exporting Values

```yaml
Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VPC-ID'
```

### Importing Values

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      SubnetId: !ImportValue network-stack-Public-Subnet-ID
```

## Nested Stacks

```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/bucket/network.yaml
      Parameters:
        VpcCIDR: 10.0.0.0/16
  
  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: NetworkStack
    Properties:
      TemplateURL: https://s3.amazonaws.com/bucket/application.yaml
      Parameters:
        VPCId: !GetAtt NetworkStack.Outputs.VPCId
```

## Best Practices

1. **Use Parameters**: Make templates reusable
2. **Use Mappings**: Region-specific values
3. **Use Conditions**: Conditional resource creation
4. **Use Outputs**: Share values between stacks
5. **Use Change Sets**: Preview changes
6. **Version Templates**: Store in version control
7. **Use Stack Policies**: Prevent accidental updates
8. **Tag Resources**: For organization and cost tracking
9. **Use Nested Stacks**: Modularize complex templates
10. **Validate Templates**: Before deployment

## Template Validation

```bash
# Validate template
aws cloudformation validate-template \
    --template-body file://template.yaml

# Use cfn-lint for advanced validation
pip install cfn-lint
cfn-lint template.yaml
```

## Drift Detection

```bash
# Detect drift
aws cloudformation detect-stack-drift \
    --stack-name my-stack

# Get drift results
aws cloudformation describe-stack-resource-drifts \
    --stack-name my-stack
```

## Stack Policies

```json
{
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "LogicalResourceId/ProductionDatabase"
    },
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "*"
    }
  ]
}
```

Apply policy:
```bash
aws cloudformation set-stack-policy \
    --stack-name my-stack \
    --stack-policy-body file://policy.json
```

## Troubleshooting

### Stack creation failed
1. Check CloudFormation events
2. Review error messages
3. Verify IAM permissions
4. Check resource limits
5. Validate template syntax

### Stack stuck in UPDATE_ROLLBACK_FAILED
```bash
# Continue rollback
aws cloudformation continue-update-rollback \
    --stack-name my-stack \
    --resources-to-skip Resource1,Resource2
```

### View stack events
```bash
aws cloudformation describe-stack-events \
    --stack-name my-stack
```

## Additional Resources

- [CloudFormation User Guide](https://docs.aws.amazon.com/cloudformation/)
- [CloudFormation Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)
- [CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [AWS Quick Starts](https://aws.amazon.com/quickstart/)
- [CloudFormation Samples](https://github.com/awslabs/aws-cloudformation-templates)

## Next Steps

- [Learn about AWS CDK](../cdk/)
- [Explore Terraform](../terraform/)
- [Understand Lambda](../../compute/lambda/)
