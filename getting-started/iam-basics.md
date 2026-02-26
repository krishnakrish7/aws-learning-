# IAM (Identity and Access Management) Basics

AWS Identity and Access Management (IAM) enables you to manage access to AWS services and resources securely.

## Core Concepts

### Users
- Represents a person or service that interacts with AWS
- Has permanent credentials (username/password or access keys)
- Can be a member of groups

### Groups
- Collection of IAM users
- Makes permission management easier
- Users inherit group permissions

### Roles
- Similar to users but intended to be assumed by anyone who needs it
- No permanent credentials
- Used by AWS services, applications, or federated users

### Policies
- JSON documents that define permissions
- Can be attached to users, groups, or roles
- Defines what actions are allowed or denied

## IAM Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### Policy Elements

- **Version**: Policy language version (always use "2012-10-17")
- **Statement**: Main policy element containing:
  - **Effect**: "Allow" or "Deny"
  - **Action**: List of actions (e.g., "s3:GetObject")
  - **Resource**: ARN of resources affected
  - **Condition** (optional): When policy is in effect

## Creating IAM Users

### Via AWS Console

1. Navigate to IAM Console
2. Click "Users" → "Add user"
3. Enter username
4. Select access type:
   - Programmatic access (access key)
   - AWS Management Console access (password)
5. Set permissions
6. Add tags (optional)
7. Review and create

### Via AWS CLI

```bash
# Create user
aws iam create-user --user-name john-doe

# Create access key
aws iam create-access-key --user-name john-doe

# Attach policy
aws iam attach-user-policy \
    --user-name john-doe \
    --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

## Working with Groups

### Create a Group

```bash
# Create group
aws iam create-group --group-name Developers

# Attach policy to group
aws iam attach-group-policy \
    --group-name Developers \
    --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Add user to group
aws iam add-user-to-group \
    --user-name john-doe \
    --group-name Developers
```

## IAM Roles

### When to Use Roles

- EC2 instances accessing AWS services
- Lambda functions
- Cross-account access
- Federation with external identity providers

### Create a Role for EC2

```bash
# Create assume role policy document
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create role
aws iam create-role \
    --role-name EC2-S3-Access-Role \
    --assume-role-policy-document file://trust-policy.json

# Attach policy
aws iam attach-role-policy \
    --role-name EC2-S3-Access-Role \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
    --instance-profile-name EC2-S3-Access-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
    --instance-profile-name EC2-S3-Access-Profile \
    --role-name EC2-S3-Access-Role
```

## AWS Managed Policies

Commonly used AWS managed policies:

- **AdministratorAccess**: Full access to all AWS services
- **PowerUserAccess**: Admin access except IAM/Organizations
- **ReadOnlyAccess**: Read-only access to all AWS services
- **AmazonS3FullAccess**: Full access to S3
- **AmazonEC2FullAccess**: Full access to EC2
- **IAMUserChangePassword**: Allow users to change their password

## Custom IAM Policies

### Example: S3 Bucket Specific Access

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::my-app-bucket"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::my-app-bucket/*"
    }
  ]
}
```

### Create Custom Policy

```bash
# Save policy to file
cat > custom-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:Describe*",
      "Resource": "*"
    }
  ]
}
EOF

# Create policy
aws iam create-policy \
    --policy-name EC2DescribeOnly \
    --policy-document file://custom-policy.json
```

## Multi-Factor Authentication (MFA)

### Enable MFA for User

1. **Via Console**:
   - IAM → Users → Security credentials
   - Manage MFA device
   - Follow activation wizard

2. **Via CLI**:
```bash
# Enable virtual MFA device
aws iam enable-mfa-device \
    --user-name john-doe \
    --serial-number arn:aws:iam::123456789012:mfa/john-doe \
    --authentication-code-1 123456 \
    --authentication-code-2 789012
```

### Require MFA for Sensitive Operations

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:TerminateInstances",
      "Resource": "*",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```

## IAM Best Practices

### 1. Least Privilege Principle
- Grant minimum permissions needed
- Start with minimal permissions, add as needed

### 2. Use Groups
- Assign permissions to groups, not individual users
- Add users to appropriate groups

### 3. Enable MFA
- Especially for privileged accounts
- Both root and IAM users

### 4. Rotate Credentials
- Regularly rotate access keys
- Use temporary credentials when possible

### 5. Use Roles for Applications
- Don't embed credentials in code
- Use IAM roles for EC2, Lambda, etc.

### 6. Monitor Activity
- Enable CloudTrail
- Review IAM credentials report
- Use Access Advisor to review permissions

### 7. Remove Unnecessary Credentials
- Delete unused users
- Deactivate old access keys
- Remove unnecessary permissions

## Useful IAM Commands

```bash
# List users
aws iam list-users

# List groups
aws iam list-groups

# List roles
aws iam list-roles

# Get user details
aws iam get-user --user-name john-doe

# List attached user policies
aws iam list-attached-user-policies --user-name john-doe

# List access keys for user
aws iam list-access-keys --user-name john-doe

# Generate credentials report
aws iam generate-credential-report
aws iam get-credential-report --output text | base64 --decode > credential-report.csv

# Policy simulator (check if action is allowed)
aws iam simulate-principal-policy \
    --policy-source-arn arn:aws:iam::123456789012:user/john-doe \
    --action-names s3:GetObject \
    --resource-arns arn:aws:s3:::my-bucket/file.txt
```

## IAM Access Analyzer

Helps identify resources shared with external entities:

```bash
# Create analyzer
aws accessanalyzer create-analyzer \
    --analyzer-name my-analyzer \
    --type ACCOUNT

# List findings
aws accessanalyzer list-findings \
    --analyzer-arn arn:aws:access-analyzer:us-east-1:123456789012:analyzer/my-analyzer
```

## Common IAM Scenarios

### Scenario 1: Developer Access
Create a group for developers with:
- EC2, S3, Lambda access
- No permission to modify IAM or billing

### Scenario 2: Read-Only Auditor
Create user with:
- ViewOnlyAccess policy
- Access to CloudTrail logs
- No write permissions

### Scenario 3: Cross-Account Access
Allow users from Account A to assume role in Account B

## Troubleshooting

### Access Denied Errors
1. Check IAM policy attached to user/role
2. Check resource-based policies
3. Check service control policies (SCPs)
4. Use policy simulator to test

### Common Mistakes
- Forgetting to attach policy to user/group
- Incorrect resource ARN in policy
- Policy effect set to "Deny" instead of "Allow"
- Missing required actions in policy

## Security Resources

- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- [AWS Security Blog](https://aws.amazon.com/blogs/security/)

## Next Steps

- [Explore EC2](../compute/ec2/)
- [Learn about S3](../storage/s3/)
- [Understand VPC](../networking/vpc/)
