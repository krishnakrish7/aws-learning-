# Amazon RDS (Relational Database Service)

Amazon RDS makes it easy to set up, operate, and scale relational databases in the cloud.

## What is RDS?

RDS provides:
- Managed database service
- Automated backups and patching
- High availability with Multi-AZ
- Read replicas for scaling
- Multiple database engines

## Supported Database Engines

- **Amazon Aurora**: MySQL and PostgreSQL compatible
- **MySQL**
- **PostgreSQL**
- **MariaDB**
- **Oracle**
- **Microsoft SQL Server**

## Key Concepts

### DB Instance
- Basic building block
- Isolated database environment
- Specific engine and version
- Compute and storage resources

### DB Instance Classes
- **Standard**: db.t3, db.m5
- **Memory Optimized**: db.r5, db.x1
- **Burstable**: db.t3 (credits-based)

### Storage Types
- **General Purpose (SSD)**: gp2, gp3
- **Provisioned IOPS (SSD)**: io1, io2
- **Magnetic**: Standard (legacy)

### Multi-AZ Deployment
- Automatic failover
- Synchronous replication
- High availability

### Read Replicas
- Asynchronous replication
- Read scalability
- Up to 5 replicas per instance

## Getting Started

### 1. Create Your First RDS Instance

#### Via AWS Console

1. Navigate to RDS Console
2. Click "Create database"
3. Choose engine (e.g., MySQL)
4. Select template (Free tier for learning)
5. Configure:
   - **DB Instance identifier**: mydb
   - **Master username**: admin
   - **Master password**: [create strong password]
   - **DB instance class**: db.t3.micro
   - **Storage**: 20 GB
6. Configure connectivity:
   - **VPC**: Default
   - **Public access**: Yes (for learning)
   - **Security group**: Create new
7. Click "Create database"

#### Via AWS CLI

```bash
# Create MySQL database
aws rds create-db-instance \
    --db-instance-identifier mydb \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --engine-version 8.0.35 \
    --master-username admin \
    --master-user-password MySecurePassword123! \
    --allocated-storage 20 \
    --storage-type gp2 \
    --vpc-security-group-ids sg-12345678 \
    --backup-retention-period 7 \
    --publicly-accessible

# Wait for instance to be available
aws rds wait db-instance-available --db-instance-identifier mydb
```

### 2. Connect to Database

```bash
# Get endpoint
aws rds describe-db-instances \
    --db-instance-identifier mydb \
    --query 'DBInstances[0].Endpoint.Address' \
    --output text

# Connect using MySQL client
mysql -h mydb.xxxxxx.us-east-1.rds.amazonaws.com \
      -P 3306 \
      -u admin \
      -p

# Or PostgreSQL
psql -h mydb.xxxxxx.us-east-1.rds.amazonaws.com \
     -p 5432 \
     -U admin \
     -d postgres
```

### 3. Create Database and Tables

```sql
-- MySQL
CREATE DATABASE myapp;
USE myapp;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (username, email) 
VALUES ('john_doe', 'john@example.com');

SELECT * FROM users;
```

## Common Operations

### List DB Instances

```bash
# List all instances
aws rds describe-db-instances

# List specific instance
aws rds describe-db-instances \
    --db-instance-identifier mydb \
    --query 'DBInstances[0].[DBInstanceIdentifier,DBInstanceStatus,Endpoint.Address]' \
    --output table
```

### Modify DB Instance

```bash
# Modify instance class
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --db-instance-class db.t3.small \
    --apply-immediately

# Increase storage
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --allocated-storage 50 \
    --apply-immediately

# Enable Multi-AZ
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --multi-az \
    --apply-immediately
```

### Backups and Snapshots

```bash
# Create snapshot
aws rds create-db-snapshot \
    --db-instance-identifier mydb \
    --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)

# List snapshots
aws rds describe-db-snapshots \
    --db-instance-identifier mydb

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
    --db-instance-identifier mydb-restored \
    --db-snapshot-identifier mydb-snapshot-20240101

# Copy snapshot
aws rds copy-db-snapshot \
    --source-db-snapshot-identifier mydb-snapshot-20240101 \
    --target-db-snapshot-identifier mydb-snapshot-backup \
    --source-region us-east-1
```

### Automated Backups

```bash
# Set backup retention (1-35 days)
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --backup-retention-period 7 \
    --preferred-backup-window "03:00-04:00" \
    --apply-immediately

# Restore to point in time
aws rds restore-db-instance-to-point-in-time \
    --source-db-instance-identifier mydb \
    --target-db-instance-identifier mydb-restored \
    --restore-time 2024-01-01T12:00:00Z
```

## Read Replicas

```bash
# Create read replica
aws rds create-db-instance-read-replica \
    --db-instance-identifier mydb-replica \
    --source-db-instance-identifier mydb \
    --db-instance-class db.t3.micro

# Create cross-region replica
aws rds create-db-instance-read-replica \
    --db-instance-identifier mydb-replica-west \
    --source-db-instance-identifier arn:aws:rds:us-east-1:123456789012:db:mydb \
    --db-instance-class db.t3.micro \
    --region us-west-2

# Promote replica to standalone
aws rds promote-read-replica \
    --db-instance-identifier mydb-replica
```

## Multi-AZ Deployment

```bash
# Enable Multi-AZ
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --multi-az \
    --apply-immediately

# Reboot with failover (test failover)
aws rds reboot-db-instance \
    --db-instance-identifier mydb \
    --force-failover
```

## Monitoring and Performance

### CloudWatch Metrics

```bash
# Get CPU utilization
aws cloudwatch get-metric-statistics \
    --namespace AWS/RDS \
    --metric-name CPUUtilization \
    --dimensions Name=DBInstanceIdentifier,Value=mydb \
    --start-time 2024-01-01T00:00:00Z \
    --end-time 2024-01-02T00:00:00Z \
    --period 3600 \
    --statistics Average

# Key RDS metrics:
# - CPUUtilization
# - DatabaseConnections
# - FreeableMemory
# - ReadIOPS, WriteIOPS
# - ReadLatency, WriteLatency
# - FreeStorageSpace
```

### Enhanced Monitoring

```bash
# Enable Enhanced Monitoring
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --monitoring-interval 60 \
    --monitoring-role-arn arn:aws:iam::123456789012:role/rds-monitoring-role \
    --apply-immediately
```

### Performance Insights

```bash
# Enable Performance Insights
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --enable-performance-insights \
    --performance-insights-retention-period 7 \
    --apply-immediately
```

## Security

### Security Groups

```bash
# Create security group
aws ec2 create-security-group \
    --group-name rds-sg \
    --description "RDS security group" \
    --vpc-id vpc-12345678

# Allow MySQL from specific IP
aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 3306 \
    --cidr 203.0.113.0/24

# Allow from another security group (EC2 instances)
aws ec2 authorize-security-group-ingress \
    --group-id sg-rds \
    --protocol tcp \
    --port 3306 \
    --source-group sg-ec2
```

### Encryption

```bash
# Create encrypted instance
aws rds create-db-instance \
    --db-instance-identifier mydb-encrypted \
    --engine mysql \
    --db-instance-class db.t3.micro \
    --master-username admin \
    --master-user-password MySecurePassword123! \
    --allocated-storage 20 \
    --storage-encrypted \
    --kms-key-id arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012

# Enable encryption in transit (SSL/TLS)
# Requires SSL connection from client
mysql -h mydb.xxx.rds.amazonaws.com -u admin -p --ssl-mode=REQUIRED
```

### IAM Database Authentication

```bash
# Enable IAM authentication
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --enable-iam-database-authentication \
    --apply-immediately

# Generate auth token
TOKEN=$(aws rds generate-db-auth-token \
    --hostname mydb.xxx.rds.amazonaws.com \
    --port 3306 \
    --username iam_user)

# Connect using token
mysql -h mydb.xxx.rds.amazonaws.com -u iam_user --password=$TOKEN --ssl-ca=rds-ca.pem
```

## Parameter Groups

```bash
# Create parameter group
aws rds create-db-parameter-group \
    --db-parameter-group-name custom-mysql8 \
    --db-parameter-group-family mysql8.0 \
    --description "Custom MySQL 8.0 parameters"

# Modify parameters
aws rds modify-db-parameter-group \
    --db-parameter-group-name custom-mysql8 \
    --parameters "ParameterName=max_connections,ParameterValue=200,ApplyMethod=immediate"

# Apply to instance
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --db-parameter-group-name custom-mysql8 \
    --apply-immediately

# Reboot to apply some parameters
aws rds reboot-db-instance --db-instance-identifier mydb
```

## Maintenance and Patching

```bash
# Set maintenance window
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --preferred-maintenance-window "sun:03:00-sun:04:00" \
    --apply-immediately

# Enable auto minor version upgrade
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --auto-minor-version-upgrade \
    --apply-immediately
```

## Cost Optimization

```bash
# Stop database (max 7 days)
aws rds stop-db-instance --db-instance-identifier mydb

# Start database
aws rds start-db-instance --db-instance-identifier mydb

# Delete instance (with final snapshot)
aws rds delete-db-instance \
    --db-instance-identifier mydb \
    --final-db-snapshot-identifier mydb-final-snapshot \
    --skip-final-snapshot false

# Delete instance (no snapshot)
aws rds delete-db-instance \
    --db-instance-identifier mydb \
    --skip-final-snapshot
```

## Best Practices

1. **Use Multi-AZ**: For production databases
2. **Enable Automated Backups**: Set retention period
3. **Use Read Replicas**: For read-heavy workloads
4. **Enable Encryption**: For sensitive data
5. **Use Parameter Groups**: Optimize database performance
6. **Monitor Performance**: Use CloudWatch and Performance Insights
7. **Regular Snapshots**: Before major changes
8. **Use VPC**: Isolate database in private subnet
9. **Rotate Credentials**: Regularly update passwords
10. **Tag Resources**: For cost tracking and organization

## Connection from Application

### Python (MySQL)

```python
import pymysql

connection = pymysql.connect(
    host='mydb.xxx.rds.amazonaws.com',
    user='admin',
    password='password',
    database='myapp',
    port=3306
)

with connection.cursor() as cursor:
    cursor.execute("SELECT * FROM users")
    results = cursor.fetchall()
    for row in results:
        print(row)

connection.close()
```

### Node.js (PostgreSQL)

```javascript
const { Client } = require('pg');

const client = new Client({
    host: 'mydb.xxx.rds.amazonaws.com',
    port: 5432,
    user: 'admin',
    password: 'password',
    database: 'myapp',
    ssl: { rejectUnauthorized: false }
});

client.connect();

client.query('SELECT * FROM users', (err, res) => {
    if (err) throw err;
    console.log(res.rows);
    client.end();
});
```

## Troubleshooting

### Cannot connect to database
1. Check security group allows access
2. Verify public accessibility setting
3. Check VPC and subnet configuration
4. Verify credentials
5. Check database status is "available"

### Performance issues
1. Check CloudWatch metrics
2. Use Performance Insights
3. Review slow query logs
4. Consider read replicas
5. Optimize queries and indexes
6. Increase instance size if needed

### Storage full
1. Increase allocated storage
2. Enable storage autoscaling
3. Clean up old data
4. Archive to S3

## Additional Resources

- [RDS User Guide](https://docs.aws.amazon.com/rds/)
- [RDS Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html)
- [RDS Pricing](https://aws.amazon.com/rds/pricing/)
- [Aurora Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/)

## Next Steps

- [Learn about DynamoDB](../dynamodb/)
- [Explore VPC](../../networking/vpc/)
- [Understand Lambda](../../compute/lambda/)
