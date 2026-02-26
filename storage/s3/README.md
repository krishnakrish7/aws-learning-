# Amazon S3 (Simple Storage Service)

Amazon S3 is an object storage service offering industry-leading scalability, data availability, security, and performance.

## What is S3?

S3 allows you to:
- Store and retrieve any amount of data
- Access data from anywhere on the web
- Pay only for storage you use
- Achieve 99.999999999% (11 9's) durability
- Host static websites

## Key Concepts

### Buckets
- Containers for objects
- Globally unique names
- Region-specific
- Up to 100 buckets per account (can request increase)

### Objects
- Files stored in S3
- Consist of data and metadata
- Maximum size: 5 TB
- Identified by key (filename)

### Keys
- Unique identifier for object within bucket
- Can include prefixes (folder-like structure)
- Example: `images/2024/photo.jpg`

### Regions
- Physical location where S3 stores buckets
- Choose based on latency, cost, regulatory requirements

### Storage Classes
- **S3 Standard**: Frequent access
- **S3 Intelligent-Tiering**: Automatic cost optimization
- **S3 Standard-IA**: Infrequent access
- **S3 One Zone-IA**: Infrequent access, single AZ
- **S3 Glacier**: Long-term archive
- **S3 Glacier Deep Archive**: Lowest cost archive

## Getting Started

### 1. Create Your First Bucket

#### Via AWS Console

1. Navigate to S3 Console
2. Click "Create bucket"
3. Configure:
   - **Bucket name**: my-first-bucket-12345 (must be unique)
   - **Region**: us-east-1
   - **Block Public Access**: Keep enabled (recommended)
4. Click "Create bucket"

#### Via AWS CLI

```bash
# Create bucket
aws s3 mb s3://my-first-bucket-12345

# Create bucket in specific region
aws s3 mb s3://my-first-bucket-12345 --region us-west-2

# List buckets
aws s3 ls
```

### 2. Upload Files

#### Via Console
1. Click on bucket name
2. Click "Upload"
3. Add files
4. Click "Upload"

#### Via CLI

```bash
# Upload single file
aws s3 cp file.txt s3://my-first-bucket-12345/

# Upload with prefix (folder)
aws s3 cp photo.jpg s3://my-first-bucket-12345/images/

# Upload directory
aws s3 cp ./local-folder s3://my-first-bucket-12345/folder/ --recursive

# Sync directory
aws s3 sync ./local-folder s3://my-first-bucket-12345/folder/
```

### 3. Download Files

```bash
# Download single file
aws s3 cp s3://my-first-bucket-12345/file.txt ./

# Download directory
aws s3 cp s3://my-first-bucket-12345/folder/ ./local-folder/ --recursive

# Sync from S3
aws s3 sync s3://my-first-bucket-12345/folder/ ./local-folder/
```

### 4. List Objects

```bash
# List all objects
aws s3 ls s3://my-first-bucket-12345/

# List with prefix
aws s3 ls s3://my-first-bucket-12345/images/

# Recursive list
aws s3 ls s3://my-first-bucket-12345/ --recursive

# Human-readable sizes
aws s3 ls s3://my-first-bucket-12345/ --human-readable
```

## Bucket Operations

### Bucket Policies

Grant public read access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

Apply policy:

```bash
# Create policy file
cat > bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
EOF

# Apply policy
aws s3api put-bucket-policy \
    --bucket my-bucket \
    --policy file://bucket-policy.json

# Get bucket policy
aws s3api get-bucket-policy --bucket my-bucket

# Delete bucket policy
aws s3api delete-bucket-policy --bucket my-bucket
```

### Versioning

Enable version control:

```bash
# Enable versioning
aws s3api put-bucket-versioning \
    --bucket my-bucket \
    --versioning-configuration Status=Enabled

# Check versioning status
aws s3api get-bucket-versioning --bucket my-bucket

# List object versions
aws s3api list-object-versions --bucket my-bucket

# Retrieve specific version
aws s3api get-object \
    --bucket my-bucket \
    --key file.txt \
    --version-id VERSION_ID \
    output.txt
```

### Lifecycle Policies

Automate transitions and deletions:

```json
{
  "Rules": [
    {
      "Id": "Archive old files",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ],
      "Expiration": {
        "Days": 365
      }
    }
  ]
}
```

Apply lifecycle:

```bash
# Create lifecycle configuration
cat > lifecycle.json <<EOF
{
  "Rules": [{
    "Id": "MoveToGlacier",
    "Status": "Enabled",
    "Transitions": [{
      "Days": 90,
      "StorageClass": "GLACIER"
    }],
    "Filter": {"Prefix": "archive/"}
  }]
}
EOF

# Apply lifecycle
aws s3api put-bucket-lifecycle-configuration \
    --bucket my-bucket \
    --lifecycle-configuration file://lifecycle.json

# Get lifecycle
aws s3api get-bucket-lifecycle-configuration --bucket my-bucket
```

### Server-Side Encryption

```bash
# Enable default encryption (AES-256)
aws s3api put-bucket-encryption \
    --bucket my-bucket \
    --server-side-encryption-configuration '{
        "Rules": [{
            "ApplyServerSideEncryptionByDefault": {
                "SSEAlgorithm": "AES256"
            }
        }]
    }'

# Upload with encryption
aws s3 cp file.txt s3://my-bucket/ --sse AES256

# Upload with KMS encryption
aws s3 cp file.txt s3://my-bucket/ --sse aws:kms --sse-kms-key-id KEY_ID
```

### Cross-Region Replication

```bash
# Enable versioning (required for replication)
aws s3api put-bucket-versioning \
    --bucket source-bucket \
    --versioning-configuration Status=Enabled

aws s3api put-bucket-versioning \
    --bucket destination-bucket \
    --versioning-configuration Status=Enabled

# Create replication configuration
cat > replication.json <<EOF
{
  "Role": "arn:aws:iam::123456789012:role/s3-replication-role",
  "Rules": [{
    "Status": "Enabled",
    "Priority": 1,
    "Filter": {},
    "Destination": {
      "Bucket": "arn:aws:s3:::destination-bucket"
    }
  }]
}
EOF

# Apply replication
aws s3api put-bucket-replication \
    --bucket source-bucket \
    --replication-configuration file://replication.json
```

## Static Website Hosting

### Enable Website Hosting

```bash
# Enable website hosting
aws s3 website s3://my-bucket/ \
    --index-document index.html \
    --error-document error.html

# Or using API
aws s3api put-bucket-website \
    --bucket my-bucket \
    --website-configuration '{
        "IndexDocument": {"Suffix": "index.html"},
        "ErrorDocument": {"Key": "error.html"}
    }'

# Get website configuration
aws s3api get-bucket-website --bucket my-bucket
```

### Upload Website Files

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>My S3 Website</title>
</head>
<body>
    <h1>Hello from S3!</h1>
    <p>This is a static website hosted on Amazon S3.</p>
</body>
</html>
```

```bash
# Upload website files
aws s3 cp index.html s3://my-bucket/ --content-type text/html
aws s3 cp error.html s3://my-bucket/ --content-type text/html
aws s3 cp styles.css s3://my-bucket/ --content-type text/css

# Sync entire website
aws s3 sync ./website s3://my-bucket/ --delete
```

Website URL: `http://my-bucket.s3-website-us-east-1.amazonaws.com`

## Presigned URLs

Generate temporary URLs for private objects:

```bash
# Generate presigned URL (valid for 1 hour)
aws s3 presign s3://my-bucket/private-file.pdf --expires-in 3600

# Use in scripts
URL=$(aws s3 presign s3://my-bucket/file.txt --expires-in 300)
curl "$URL" -o downloaded-file.txt
```

Python example:

```python
import boto3
from botocore.exceptions import ClientError

s3_client = boto3.client('s3')

def create_presigned_url(bucket_name, object_name, expiration=3600):
    try:
        response = s3_client.generate_presigned_url(
            'get_object',
            Params={'Bucket': bucket_name, 'Key': object_name},
            ExpiresIn=expiration
        )
    except ClientError as e:
        print(e)
        return None
    return response

# Usage
url = create_presigned_url('my-bucket', 'file.txt')
print(url)
```

## S3 Event Notifications

Trigger actions on S3 events:

```bash
# Configure Lambda notification
aws s3api put-bucket-notification-configuration \
    --bucket my-bucket \
    --notification-configuration '{
        "LambdaFunctionConfigurations": [{
            "LambdaFunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:process-upload",
            "Events": ["s3:ObjectCreated:*"],
            "Filter": {
                "Key": {
                    "FilterRules": [
                        {"Name": "prefix", "Value": "uploads/"},
                        {"Name": "suffix", "Value": ".jpg"}
                    ]
                }
            }
        }]
    }'

# Configure SNS notification
aws s3api put-bucket-notification-configuration \
    --bucket my-bucket \
    --notification-configuration '{
        "TopicConfigurations": [{
            "TopicArn": "arn:aws:sns:us-east-1:123456789012:s3-notifications",
            "Events": ["s3:ObjectCreated:*"]
        }]
    }'
```

## Performance Optimization

### Multipart Upload

For files > 100 MB:

```bash
# Automatic with aws s3 cp (threshold 8MB by default)
aws s3 cp large-file.zip s3://my-bucket/

# Manual multipart upload
aws s3api create-multipart-upload --bucket my-bucket --key large-file.zip

# Upload parts
aws s3api upload-part --bucket my-bucket --key large-file.zip \
    --part-number 1 --upload-id UPLOAD_ID --body part1.bin

# Complete upload
aws s3api complete-multipart-upload --bucket my-bucket --key large-file.zip \
    --upload-id UPLOAD_ID --multipart-upload file://parts.json
```

### Transfer Acceleration

```bash
# Enable transfer acceleration
aws s3api put-bucket-accelerate-configuration \
    --bucket my-bucket \
    --accelerate-configuration Status=Enabled

# Upload using acceleration
aws s3 cp file.txt s3://my-bucket/ --endpoint-url https://s3-accelerate.amazonaws.com
```

## Access Control

### ACLs (Access Control Lists)

```bash
# Make object public
aws s3api put-object-acl \
    --bucket my-bucket \
    --key file.txt \
    --acl public-read

# Grant specific user access
aws s3api put-object-acl \
    --bucket my-bucket \
    --key file.txt \
    --grant-read emailaddress=user@example.com
```

### Bucket Ownership Controls

```bash
# Enable bucket owner preferred
aws s3api put-bucket-ownership-controls \
    --bucket my-bucket \
    --ownership-controls Rules=[{ObjectOwnership=BucketOwnerPreferred}]
```

## Best Practices

1. **Enable Versioning**: Protect against accidental deletes
2. **Use Lifecycle Policies**: Automate data management
3. **Enable Encryption**: Encrypt data at rest
4. **Use IAM Policies**: Control access with least privilege
5. **Enable Logging**: Track bucket access
6. **Use CloudFront**: Improve performance for global users
7. **Tag Buckets**: Organize and track costs
8. **Block Public Access**: Unless specifically needed
9. **Monitor Costs**: Use Cost Explorer and budgets
10. **Regular Audits**: Review bucket policies and access

## Cost Optimization

- **Use appropriate storage class**: Standard, IA, Glacier
- **Enable Intelligent-Tiering**: Automatic cost savings
- **Lifecycle policies**: Move or delete old data
- **Delete incomplete uploads**: Clean up failed multipart uploads
- **Compress data**: Before uploading
- **Use CloudFront**: Reduce data transfer costs
- **Monitor requests**: Optimize application access patterns

## Security Best Practices

1. **Block public access** by default
2. **Use bucket policies** and IAM roles
3. **Enable MFA Delete** for versioned buckets
4. **Encrypt sensitive data**
5. **Enable logging** and monitoring
6. **Use VPC endpoints** for private access
7. **Implement least privilege** access
8. **Scan for vulnerabilities** regularly

## Troubleshooting

### Access Denied
1. Check bucket policy
2. Verify IAM permissions
3. Check ACLs
4. Verify bucket exists in correct region

### Slow Upload/Download
1. Use Transfer Acceleration
2. Use multipart upload for large files
3. Check network connection
4. Consider CloudFront

### Bucket Already Exists
- Bucket names are globally unique
- Choose a different name

## Common Commands Cheat Sheet

```bash
# Create bucket
aws s3 mb s3://bucket-name

# Delete bucket (must be empty)
aws s3 rb s3://bucket-name

# Delete bucket with contents
aws s3 rb s3://bucket-name --force

# Copy file
aws s3 cp source.txt s3://bucket/destination.txt

# Move file
aws s3 mv s3://bucket/old.txt s3://bucket/new.txt

# Delete file
aws s3 rm s3://bucket/file.txt

# Delete folder
aws s3 rm s3://bucket/folder/ --recursive

# Get object metadata
aws s3api head-object --bucket bucket-name --key file.txt

# Get bucket location
aws s3api get-bucket-location --bucket bucket-name

# Get bucket size
aws s3 ls s3://bucket-name --recursive --human-readable --summarize
```

## Example Projects

1. **Static Website**: Host portfolio or blog
2. **Backup Solution**: Automated backups with lifecycle
3. **Data Lake**: Store and analyze big data
4. **Content Delivery**: Use with CloudFront
5. **File Sharing**: Presigned URLs for sharing
6. **Log Storage**: Centralized logging

## Additional Resources

- [S3 Developer Guide](https://docs.aws.amazon.com/s3/)
- [S3 Storage Classes](https://aws.amazon.com/s3/storage-classes/)
- [S3 Pricing](https://aws.amazon.com/s3/pricing/)
- [S3 Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/best-practices.html)

## Next Steps

- [Learn about CloudFront](../../networking/cloudfront/)
- [Explore Lambda](../../compute/lambda/)
- [Understand IAM](../../security/iam/)
