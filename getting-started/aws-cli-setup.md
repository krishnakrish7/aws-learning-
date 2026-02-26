# AWS CLI Setup Guide

The AWS Command Line Interface (CLI) is a unified tool to manage AWS services from the command line.

## Installation

### macOS

```bash
# Using Homebrew
brew install awscli

# Or download the installer
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

### Linux

```bash
# Download and install
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Windows

1. Download the installer: [AWS CLI MSI installer](https://awscli.amazonaws.com/AWSCLIV2.msi)
2. Run the installer and follow the prompts

## Verify Installation

```bash
aws --version
# Expected output: aws-cli/2.x.x Python/3.x.x ...
```

## Configuration

### 1. Create Access Keys

Before configuring CLI, you need access keys:

1. Sign in to AWS Console
2. Navigate to IAM → Users → Your IAM User
3. Click "Security credentials" tab
4. Click "Create access key"
5. Choose "Command Line Interface (CLI)"
6. Download and save the credentials securely

⚠️ **Security Warning**: Never share or commit access keys to version control!

### 2. Configure AWS CLI

```bash
aws configure
```

You'll be prompted for:

```
AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]: json
```

**Recommended regions for learning**:
- `us-east-1` (N. Virginia) - Most services available
- `us-west-2` (Oregon) - Good for West Coast
- `eu-west-1` (Ireland) - Good for Europe

**Output formats**:
- `json` - Default, most versatile
- `table` - Human-readable tables
- `text` - Tab-separated values
- `yaml` - YAML format

### 3. Verify Configuration

```bash
# Check configured credentials
aws sts get-caller-identity

# Expected output:
# {
#     "UserId": "AIDAXXXXXXXXXXXXXXXX",
#     "Account": "123456789012",
#     "Arn": "arn:aws:iam::123456789012:user/your-username"
# }
```

## Multiple Profiles

You can configure multiple AWS profiles for different accounts or roles:

```bash
# Configure a new profile
aws configure --profile project1

# Use a specific profile
aws s3 ls --profile project1

# Set default profile via environment variable
export AWS_PROFILE=project1
```

### Profile Configuration File

Profiles are stored in `~/.aws/credentials`:

```ini
[default]
aws_access_key_id = YOUR_DEFAULT_KEY
aws_secret_access_key = YOUR_DEFAULT_SECRET

[project1]
aws_access_key_id = YOUR_PROJECT1_KEY
aws_secret_access_key = YOUR_PROJECT1_SECRET
```

And `~/.aws/config`:

```ini
[default]
region = us-east-1
output = json

[profile project1]
region = us-west-2
output = table
```

## Common AWS CLI Commands

### S3 Operations
```bash
# List buckets
aws s3 ls

# Create bucket
aws s3 mb s3://my-bucket-name

# Upload file
aws s3 cp file.txt s3://my-bucket-name/

# Download file
aws s3 cp s3://my-bucket-name/file.txt ./

# Sync directory
aws s3 sync ./local-dir s3://my-bucket-name/
```

### EC2 Operations
```bash
# List instances
aws ec2 describe-instances

# List instances (formatted)
aws ec2 describe-instances \
    --query 'Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]' \
    --output table

# Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

### IAM Operations
```bash
# List users
aws iam list-users

# List policies for user
aws iam list-attached-user-policies --user-name your-username

# Create user
aws iam create-user --user-name new-user
```

## AWS CLI Best Practices

1. **Use IAM Roles for EC2**
   - Attach roles to EC2 instances instead of storing credentials

2. **Use Named Profiles**
   - Separate credentials for different projects/environments

3. **Enable MFA for CLI**
   - Use temporary credentials with MFA

4. **Use AWS CLI v2**
   - Latest version with improved features

5. **Leverage AWS CLI Aliases**
   - Create shortcuts for frequently used commands

## Advanced Configuration

### Using MFA with CLI

```bash
# Get session token with MFA
aws sts get-session-token \
    --serial-number arn:aws:iam::123456789012:mfa/your-username \
    --token-code 123456

# Use temporary credentials
export AWS_ACCESS_KEY_ID=TEMP_KEY_ID
export AWS_SECRET_ACCESS_KEY=TEMP_SECRET_KEY
export AWS_SESSION_TOKEN=TEMP_SESSION_TOKEN
```

### CLI Aliases

Create `~/.aws/cli/alias` file:

```ini
[toplevel]
whoami = sts get-caller-identity
list-instances = ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value|[0]]' --output table
```

Use aliases:
```bash
aws whoami
aws list-instances
```

## Troubleshooting

### Issue: Command not found
```bash
# Add AWS CLI to PATH
export PATH=$PATH:/usr/local/bin
```

### Issue: Access denied
- Verify IAM permissions
- Check if access keys are correct
- Ensure region is correct

### Issue: SSL certificate errors
```bash
# Disable SSL verification (not recommended for production)
aws s3 ls --no-verify-ssl
```

## Additional Resources

- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [AWS CLI User Guide](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS CLI GitHub Repository](https://github.com/aws/aws-cli)

## Next Steps

- [Learn IAM Basics](./iam-basics.md)
- [Start with EC2](../compute/ec2/)
- [Explore S3](../storage/s3/)
