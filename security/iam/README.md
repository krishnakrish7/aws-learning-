# AWS IAM (Identity and Access Management)

Comprehensive IAM learning resources and best practices.

## Quick Links

- [IAM Basics Guide](../../getting-started/iam-basics.md)
- [Official IAM Documentation](https://docs.aws.amazon.com/iam/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

## What You'll Learn

1. **Core Concepts**
   - Users, Groups, Roles, and Policies
   - Authentication vs Authorization
   - Permissions and privileges

2. **Access Management**
   - Password policies
   - MFA configuration
   - Access keys management
   - Temporary credentials

3. **Policy Management**
   - Managed policies vs inline policies
   - Policy evaluation logic
   - Policy variables and conditions
   - Service control policies (SCPs)

4. **Best Practices**
   - Least privilege principle
   - Regular audits
   - Credential rotation
   - MFA enforcement

## Getting Started

Start with the comprehensive [IAM Basics Guide](../../getting-started/iam-basics.md) which covers:
- Creating users and groups
- Understanding policies
- Working with roles
- Enabling MFA
- Best practices

## Common Use Cases

### 1. Developer Access Setup
```bash
# Create developer group
aws iam create-group --group-name Developers

# Attach policies
aws iam attach-group-policy \
    --group-name Developers \
    --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Create user and add to group
aws iam create-user --user-name john-doe
aws iam add-user-to-group \
    --user-name john-doe \
    --group-name Developers
```

### 2. EC2 Instance Role
```bash
# Create role for EC2
aws iam create-role \
    --role-name EC2-S3-Access \
    --assume-role-policy-document file://trust-policy.json

# Attach policy
aws iam attach-role-policy \
    --role-name EC2-S3-Access \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

### 3. Cross-Account Access
```bash
# Create assume role policy for cross-account
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111111111111:root"},
    "Action": "sts:AssumeRole"
  }]
}
EOF

# Create role
aws iam create-role \
    --role-name CrossAccountRole \
    --assume-role-policy-document file://trust-policy.json
```

## Security Checklist

- [ ] Root account MFA enabled
- [ ] Root account not used for daily operations
- [ ] IAM users created for individuals
- [ ] Groups used for permission management
- [ ] Least privilege principle applied
- [ ] Regular access key rotation
- [ ] Password policy configured
- [ ] CloudTrail logging enabled
- [ ] Regular permission audits
- [ ] Unused credentials removed

## Tools and Utilities

- **IAM Policy Simulator**: Test policy permissions
- **Access Advisor**: Review service access
- **Credential Report**: Audit user credentials
- **IAM Access Analyzer**: Identify resources shared externally

## Example Policies

### Read-Only S3 Access
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:ListBucket"
    ],
    "Resource": [
      "arn:aws:s3:::my-bucket",
      "arn:aws:s3:::my-bucket/*"
    ]
  }]
}
```

### EC2 Start/Stop with MFA
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ec2:StartInstances",
      "ec2:StopInstances"
    ],
    "Resource": "*",
    "Condition": {
      "Bool": {"aws:MultiFactorAuthPresent": "true"}
    }
  }]
}
```

## Additional Resources

- [Getting Started Guide](../../getting-started/iam-basics.md)
- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [IAM Best Practices Whitepaper](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Policy Examples](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)

## Next Steps

1. Complete [IAM Basics](../../getting-started/iam-basics.md)
2. Set up [MFA for your account](../../getting-started/aws-account-setup.md)
3. Learn about [EC2 roles](../../compute/ec2/)
4. Understand [S3 bucket policies](../../storage/s3/)
