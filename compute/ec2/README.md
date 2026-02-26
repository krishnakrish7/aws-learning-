# Amazon EC2 (Elastic Compute Cloud)

Amazon EC2 provides scalable computing capacity in the AWS cloud.

## What is EC2?

EC2 allows you to:
- Launch virtual servers (instances) in minutes
- Scale capacity up or down based on demand
- Pay only for what you use
- Choose from various instance types optimized for different workloads

## Key Concepts

### Instance Types
- **General Purpose** (t2, t3, m5): Balanced compute, memory, and networking
- **Compute Optimized** (c5, c6): High-performance processors
- **Memory Optimized** (r5, x1): Large memory workloads
- **Storage Optimized** (i3, d2): High sequential read/write access
- **Accelerated Computing** (p3, g4): GPU instances

### AMI (Amazon Machine Image)
- Template for EC2 instances
- Contains OS, applications, and configurations
- Can be public, private, or from AWS Marketplace

### Key Pairs
- Public-key cryptography for secure login
- Required for SSH access to Linux instances
- Required for RDP access to Windows instances

### Security Groups
- Virtual firewalls for instances
- Control inbound and outbound traffic
- Stateful (return traffic automatically allowed)

### Instance States
- **Pending**: Instance is launching
- **Running**: Instance is running
- **Stopping**: Instance is shutting down
- **Stopped**: Instance is stopped (not charged for compute)
- **Terminated**: Instance is permanently deleted

## Getting Started

### 1. Launch Your First EC2 Instance

#### Via AWS Console

1. Navigate to EC2 Console
2. Click "Launch Instance"
3. Configure:
   - **Name**: my-first-instance
   - **AMI**: Amazon Linux 2023
   - **Instance Type**: t2.micro (Free Tier)
   - **Key Pair**: Create new or select existing
   - **Network**: Default VPC
   - **Security Group**: Allow SSH (port 22)
4. Review and launch

#### Via AWS CLI

```bash
# Create key pair
aws ec2 create-key-pair \
    --key-name my-key-pair \
    --query 'KeyMaterial' \
    --output text > my-key-pair.pem

# Set permissions
chmod 400 my-key-pair.pem

# Create security group
aws ec2 create-security-group \
    --group-name my-sg \
    --description "My security group"

# Add SSH rule
aws ec2 authorize-security-group-ingress \
    --group-name my-sg \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

# Launch instance
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --count 1 \
    --instance-type t2.micro \
    --key-name my-key-pair \
    --security-groups my-sg \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=my-first-instance}]'
```

### 2. Connect to Your Instance

#### Linux/Mac

```bash
# Get instance public IP
aws ec2 describe-instances \
    --instance-ids i-1234567890abcdef0 \
    --query 'Reservations[0].Instances[0].PublicIpAddress'

# Connect via SSH
ssh -i my-key-pair.pem ec2-user@<PUBLIC_IP>
```

#### Windows (using PuTTY)

1. Convert .pem to .ppk using PuTTYgen
2. Use PuTTY with:
   - Host: ec2-user@<PUBLIC_IP>
   - Auth: Browse to .ppk file

### 3. Install Software on Instance

```bash
# Update packages
sudo yum update -y

# Install Apache web server
sudo yum install httpd -y

# Start Apache
sudo systemctl start httpd
sudo systemctl enable httpd

# Create a simple webpage
echo "<h1>Hello from EC2!</h1>" | sudo tee /var/www/html/index.html

# Allow HTTP traffic in security group
aws ec2 authorize-security-group-ingress \
    --group-name my-sg \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

Visit `http://<PUBLIC_IP>` in your browser!

## Common Operations

### List Instances

```bash
# List all instances
aws ec2 describe-instances

# List with custom output
aws ec2 describe-instances \
    --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
    --output table
```

### Start/Stop/Terminate Instances

```bash
# Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Reboot instance
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0
```

### Monitor Instance

```bash
# Get instance status
aws ec2 describe-instance-status --instance-ids i-1234567890abcdef0

# Get console output
aws ec2 get-console-output --instance-id i-1234567890abcdef0

# Get instance metrics (requires CloudWatch)
aws cloudwatch get-metric-statistics \
    --namespace AWS/EC2 \
    --metric-name CPUUtilization \
    --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
    --start-time 2024-01-01T00:00:00Z \
    --end-time 2024-01-02T00:00:00Z \
    --period 3600 \
    --statistics Average
```

## User Data (Bootstrap Scripts)

Run commands when instance launches:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```

Launch with user data:

```bash
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t2.micro \
    --key-name my-key-pair \
    --security-groups my-sg \
    --user-data file://user-data.sh
```

## Elastic IP Addresses

Static public IP addresses:

```bash
# Allocate Elastic IP
aws ec2 allocate-address

# Associate with instance
aws ec2 associate-address \
    --instance-id i-1234567890abcdef0 \
    --allocation-id eipalloc-12345678

# Release Elastic IP
aws ec2 release-address --allocation-id eipalloc-12345678
```

## EBS (Elastic Block Store) Volumes

### Create and Attach Volume

```bash
# Create volume
aws ec2 create-volume \
    --size 10 \
    --availability-zone us-east-1a \
    --volume-type gp3

# Attach volume
aws ec2 attach-volume \
    --volume-id vol-1234567890abcdef0 \
    --instance-id i-1234567890abcdef0 \
    --device /dev/sdf

# Format and mount (on instance)
sudo mkfs -t ext4 /dev/xvdf
sudo mkdir /data
sudo mount /dev/xvdf /data
```

### Snapshots

```bash
# Create snapshot
aws ec2 create-snapshot \
    --volume-id vol-1234567890abcdef0 \
    --description "My snapshot"

# List snapshots
aws ec2 describe-snapshots --owner-ids self

# Create volume from snapshot
aws ec2 create-volume \
    --snapshot-id snap-1234567890abcdef0 \
    --availability-zone us-east-1a
```

## Auto Scaling

### Create Launch Template

```bash
aws ec2 create-launch-template \
    --launch-template-name my-template \
    --version-description "Version 1" \
    --launch-template-data '{
        "ImageId": "ami-0c55b159cbfafe1f0",
        "InstanceType": "t2.micro",
        "KeyName": "my-key-pair",
        "SecurityGroupIds": ["sg-12345678"]
    }'
```

### Create Auto Scaling Group

```bash
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name my-asg \
    --launch-template LaunchTemplateName=my-template \
    --min-size 1 \
    --max-size 3 \
    --desired-capacity 2 \
    --availability-zones us-east-1a us-east-1b
```

## Best Practices

1. **Use IAM Roles**: Attach roles to instances instead of embedding credentials
2. **Tag Resources**: Organize and track costs with meaningful tags
3. **Regular Backups**: Create EBS snapshots regularly
4. **Right-Size Instances**: Use appropriate instance types
5. **Use Reserved Instances**: For predictable, long-term workloads
6. **Monitor and Alert**: Set up CloudWatch alarms
7. **Security Groups**: Principle of least privilege
8. **Keep AMIs Updated**: Regularly update and patch
9. **Use Auto Scaling**: For high availability and cost optimization
10. **Stop Unused Instances**: Don't leave development instances running

## Cost Optimization

- **Stop instances** when not in use (still charged for EBS)
- **Use Spot Instances** for flexible, fault-tolerant workloads
- **Right-size** your instances using CloudWatch metrics
- **Use Reserved Instances** or Savings Plans for consistent usage
- **Delete unused resources**: EBS volumes, snapshots, Elastic IPs
- **Use Auto Scaling** to match capacity with demand

## Troubleshooting

### Cannot connect to instance
1. Check security group allows SSH/RDP
2. Verify correct key pair
3. Check instance is running
4. Verify public IP/DNS
5. Check network ACLs

### Instance fails to launch
1. Check service limits
2. Verify AMI availability in region
3. Check subnet has available IPs
4. Review error message in console

### Performance issues
1. Check instance metrics in CloudWatch
2. Verify instance type is appropriate
3. Check disk I/O (IOPS)
4. Review application logs

## Example Projects

1. **Web Server**: Deploy Apache/Nginx
2. **WordPress**: Install WordPress on EC2
3. **Load Balanced App**: Use ALB with Auto Scaling
4. **Bastion Host**: Secure access to private instances
5. **CI/CD Agent**: Jenkins or GitLab runner

## Additional Resources

- [EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
- [EC2 Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-best-practices.html)

## Next Steps

- [Learn about Lambda](../lambda/)
- [Explore S3](../../storage/s3/)
- [Understand VPC](../../networking/vpc/)
