# Amazon VPC (Virtual Private Cloud)

Amazon VPC lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define.

## What is VPC?

VPC allows you to:
- Create isolated network environments
- Control IP address ranges
- Configure route tables and network gateways
- Define security using security groups and network ACLs
- Connect to on-premises networks via VPN or Direct Connect

## Key Concepts

### VPC (Virtual Private Cloud)
- Isolated virtual network in AWS
- Regional resource (spans all AZs in a region)
- CIDR block defines IP address range
- Default: 172.31.0.0/16

### Subnets
- Subdivision of VPC
- Availability Zone specific
- Public: Has route to Internet Gateway
- Private: No direct internet access

### Internet Gateway (IGW)
- Allows communication between VPC and internet
- Highly available and scalable
- One per VPC

### NAT Gateway/Instance
- Allows private subnet instances to access internet
- Prevents inbound internet access
- NAT Gateway: AWS managed (recommended)
- NAT Instance: EC2 instance (legacy)

### Route Tables
- Rules for routing network traffic
- Each subnet must have route table
- Main route table: Default for VPC

### Security Groups
- Virtual firewall at instance level
- Stateful (return traffic auto-allowed)
- Only allow rules (no deny)
- Can reference other security groups

### Network ACLs (Access Control Lists)
- Firewall at subnet level
- Stateless (must configure return traffic)
- Allow and deny rules
- Numbered rules (processed in order)

### VPC Peering
- Connect two VPCs
- Private IP communication
- No transitive peering

### VPN & Direct Connect
- VPN: Encrypted connection over internet
- Direct Connect: Dedicated network connection

## Getting Started

### 1. Create a VPC

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Enable DNS hostnames
aws ec2 modify-vpc-attribute \
    --vpc-id vpc-12345678 \
    --enable-dns-hostnames

# Tag VPC
aws ec2 create-tags \
    --resources vpc-12345678 \
    --tags Key=Name,Value=MyVPC
```

### 2. Create Subnets

```bash
# Public subnet
aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-east-1a

# Private subnet
aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.2.0/24 \
    --availability-zone us-east-1a

# Tag subnets
aws ec2 create-tags \
    --resources subnet-12345678 \
    --tags Key=Name,Value=PublicSubnet
```

### 3. Create and Attach Internet Gateway

```bash
# Create IGW
aws ec2 create-internet-gateway

# Attach to VPC
aws ec2 attach-internet-gateway \
    --vpc-id vpc-12345678 \
    --internet-gateway-id igw-12345678
```

### 4. Configure Route Tables

```bash
# Create route table for public subnet
aws ec2 create-route-table --vpc-id vpc-12345678

# Add route to internet
aws ec2 create-route \
    --route-table-id rtb-12345678 \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-12345678

# Associate with subnet
aws ec2 associate-route-table \
    --subnet-id subnet-12345678 \
    --route-table-id rtb-12345678
```

### 5. Create NAT Gateway

```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Create NAT Gateway in public subnet
aws ec2 create-nat-gateway \
    --subnet-id subnet-12345678 \
    --allocation-id eipalloc-12345678

# Add route in private subnet route table
aws ec2 create-route \
    --route-table-id rtb-private \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id nat-12345678
```

## Complete VPC Architecture Example

```bash
#!/bin/bash
# Create complete VPC with public and private subnets

VPC_CIDR="10.0.0.0/16"
PUBLIC_SUBNET_CIDR="10.0.1.0/24"
PRIVATE_SUBNET_CIDR="10.0.2.0/24"
REGION="us-east-1"
AZ="${REGION}a"

# Create VPC
VPC_ID=$(aws ec2 create-vpc \
    --cidr-block $VPC_CIDR \
    --query 'Vpc.VpcId' \
    --output text)

echo "VPC created: $VPC_ID"

# Enable DNS
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames

# Create public subnet
PUBLIC_SUBNET_ID=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block $PUBLIC_SUBNET_CIDR \
    --availability-zone $AZ \
    --query 'Subnet.SubnetId' \
    --output text)

echo "Public subnet created: $PUBLIC_SUBNET_ID"

# Create private subnet
PRIVATE_SUBNET_ID=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block $PRIVATE_SUBNET_CIDR \
    --availability-zone $AZ \
    --query 'Subnet.SubnetId' \
    --output text)

echo "Private subnet created: $PRIVATE_SUBNET_ID"

# Create and attach Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway \
    --query 'InternetGateway.InternetGatewayId' \
    --output text)

aws ec2 attach-internet-gateway \
    --vpc-id $VPC_ID \
    --internet-gateway-id $IGW_ID

echo "Internet Gateway created and attached: $IGW_ID"

# Create public route table
PUBLIC_RT_ID=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --query 'RouteTable.RouteTableId' \
    --output text)

# Add route to internet
aws ec2 create-route \
    --route-table-id $PUBLIC_RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id $IGW_ID

# Associate with public subnet
aws ec2 associate-route-table \
    --subnet-id $PUBLIC_SUBNET_ID \
    --route-table-id $PUBLIC_RT_ID

echo "Public route table created and configured: $PUBLIC_RT_ID"

# Allocate Elastic IP for NAT Gateway
ALLOCATION_ID=$(aws ec2 allocate-address \
    --domain vpc \
    --query 'AllocationId' \
    --output text)

echo "Elastic IP allocated: $ALLOCATION_ID"

# Create NAT Gateway
NAT_GW_ID=$(aws ec2 create-nat-gateway \
    --subnet-id $PUBLIC_SUBNET_ID \
    --allocation-id $ALLOCATION_ID \
    --query 'NatGateway.NatGatewayId' \
    --output text)

echo "NAT Gateway created: $NAT_GW_ID"

# Wait for NAT Gateway to be available
echo "Waiting for NAT Gateway to be available..."
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GW_ID

# Create private route table
PRIVATE_RT_ID=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --query 'RouteTable.RouteTableId' \
    --output text)

# Add route to NAT Gateway
aws ec2 create-route \
    --route-table-id $PRIVATE_RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id $NAT_GW_ID

# Associate with private subnet
aws ec2 associate-route-table \
    --subnet-id $PRIVATE_SUBNET_ID \
    --route-table-id $PRIVATE_RT_ID

echo "Private route table created and configured: $PRIVATE_RT_ID"
echo "VPC setup complete!"
```

## Security Groups

```bash
# Create security group
aws ec2 create-security-group \
    --group-name web-sg \
    --description "Web server security group" \
    --vpc-id vpc-12345678

# Allow HTTP
aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# Allow HTTPS
aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 22 \
    --cidr 203.0.113.0/24
```

## Network ACLs

```bash
# Create NACL
aws ec2 create-network-acl --vpc-id vpc-12345678

# Add inbound rule (allow HTTP)
aws ec2 create-network-acl-entry \
    --network-acl-id acl-12345678 \
    --rule-number 100 \
    --protocol tcp \
    --port-range From=80,To=80 \
    --cidr-block 0.0.0.0/0 \
    --egress false \
    --rule-action allow

# Add outbound rule (allow all)
aws ec2 create-network-acl-entry \
    --network-acl-id acl-12345678 \
    --rule-number 100 \
    --protocol -1 \
    --cidr-block 0.0.0.0/0 \
    --egress true \
    --rule-action allow
```

## VPC Peering

```bash
# Create peering connection
aws ec2 create-vpc-peering-connection \
    --vpc-id vpc-12345678 \
    --peer-vpc-id vpc-87654321

# Accept peering connection
aws ec2 accept-vpc-peering-connection \
    --vpc-peering-connection-id pcx-12345678

# Add route in VPC A
aws ec2 create-route \
    --route-table-id rtb-vpc-a \
    --destination-cidr-block 10.1.0.0/16 \
    --vpc-peering-connection-id pcx-12345678

# Add route in VPC B
aws ec2 create-route \
    --route-table-id rtb-vpc-b \
    --destination-cidr-block 10.0.0.0/16 \
    --vpc-peering-connection-id pcx-12345678
```

## VPC Flow Logs

```bash
# Create flow log
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-12345678 \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs \
    --log-group-name /aws/vpc/flowlogs \
    --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flowlogsRole

# Query flow logs (using CloudWatch Insights)
# Filter: type = 'REJECT'
```

## Best Practices

1. **Plan CIDR blocks**: Avoid overlapping with other networks
2. **Use multiple AZs**: For high availability
3. **Separate public/private subnets**: Security best practice
4. **Use NAT Gateway**: Instead of NAT instances
5. **Implement least privilege**: Security groups and NACLs
6. **Enable VPC Flow Logs**: For troubleshooting and security
7. **Use VPC endpoints**: For AWS service access without internet
8. **Tag resources**: For organization and cost tracking
9. **Document network design**: Keep architecture diagrams
10. **Regular security audits**: Review security group rules

## Common Patterns

### 1. Basic Web Application
- Public subnet: Web servers (with ALB)
- Private subnet: Application servers
- Private subnet: Database servers

### 2. Three-Tier Architecture
- Public subnet: Load balancers
- Private subnet: Application tier
- Private subnet (isolated): Database tier

### 3. Multi-Region VPC
- VPC in each region
- VPC peering or Transit Gateway
- Route 53 for DNS failover

## Troubleshooting

### Cannot connect to instance
1. Check security group rules
2. Verify NACL rules
3. Check route tables
4. Verify instance is in public subnet (for internet access)
5. Check if Elastic IP or public IP assigned

### No internet access from private subnet
1. Verify NAT Gateway is running
2. Check route table has route to NAT Gateway
3. Verify NAT Gateway is in public subnet
4. Check security groups and NACLs

### VPC peering not working
1. Verify peering connection accepted
2. Check route tables in both VPCs
3. Verify CIDR blocks don't overlap
4. Check security groups allow traffic

## Cost Optimization

- **NAT Gateway**: Consider NAT instance for low traffic
- **VPC endpoints**: Save on data transfer costs
- **Delete unused resources**: Elastic IPs, NAT Gateways
- **Monitor data transfer**: Between AZs and regions
- **Use VPC sharing**: Share VPCs across accounts

## Additional Resources

- [VPC User Guide](https://docs.aws.amazon.com/vpc/)
- [VPC Scenarios](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenarios.html)
- [VPC Pricing](https://aws.amazon.com/vpc/pricing/)
- [Network ACLs vs Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html)

## Next Steps

- [Learn about Route 53](../route53/)
- [Explore EC2](../../compute/ec2/)
- [Understand Load Balancing](../load-balancing/)
