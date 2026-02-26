# AWS Account Setup

This guide walks you through creating and configuring your AWS account.

## Creating an AWS Account

1. **Visit AWS Website**
   - Go to [https://aws.amazon.com/](https://aws.amazon.com/)
   - Click on "Create an AWS Account"

2. **Provide Account Information**
   - Enter your email address
   - Choose an AWS account name
   - Create a strong password

3. **Add Contact Information**
   - Choose account type (Personal or Professional)
   - Enter your contact details
   - Read and accept AWS Customer Agreement

4. **Payment Information**
   - Add a valid credit/debit card
   - Note: AWS may charge $1 for verification (will be refunded)

5. **Verify Identity**
   - Choose verification method (SMS or Voice call)
   - Enter the verification code received

6. **Select Support Plan**
   - For learning, select the "Basic Support - Free" plan
   - You can upgrade later if needed

## Post-Setup Configuration

### 1. Enable Billing Alerts

```bash
# Enable billing alerts through AWS CLI
aws ce put-subscription \
    --subscription-name "BillingAlert" \
    --threshold 10 \
    --subscribers Type=EMAIL,Address=your-email@example.com
```

Or use the AWS Console:
- Navigate to Billing Dashboard
- Click on "Billing Preferences"
- Enable "Receive Billing Alerts"
- Set up CloudWatch billing alarm

### 2. Enable MFA on Root Account

1. Sign in to AWS Console with root account
2. Click on account name (top right) → Security Credentials
3. Expand "Multi-factor authentication (MFA)"
4. Click "Activate MFA"
5. Choose MFA device type:
   - Virtual MFA device (recommended for learning)
   - Hardware MFA device
   - Security key
6. Follow the setup wizard

### 3. Create IAM Admin User

⚠️ **Best Practice**: Never use root account for daily operations

1. Go to IAM Console
2. Click "Users" → "Add user"
3. Enter username (e.g., "admin")
4. Select "AWS Management Console access"
5. Choose password options
6. Attach "AdministratorAccess" policy
7. Review and create user
8. Save the sign-in URL and credentials securely

## Understanding AWS Free Tier

AWS Free Tier includes:

### Always Free
- Lambda: 1 million requests per month
- DynamoDB: 25 GB storage
- SNS: 1 million publishes

### 12 Months Free
- EC2: 750 hours/month of t2.micro or t3.micro
- S3: 5 GB standard storage
- RDS: 750 hours/month of db.t2.micro

### Trials
- Various services offer short-term trials

**Important**: Monitor usage to avoid unexpected charges!

## Cost Management Tips

1. **Set Billing Alerts**
   - Create CloudWatch alarms for different threshold amounts
   - Set up budget alerts in AWS Budgets

2. **Use Cost Explorer**
   - Analyze spending patterns
   - Identify cost-saving opportunities

3. **Tag Resources**
   - Apply tags to track costs by project/department
   - Use tags for resource organization

4. **Terminate Unused Resources**
   - Stop or terminate EC2 instances when not in use
   - Delete unused S3 buckets
   - Remove old snapshots and volumes

## Useful Console Links

- [AWS Console Home](https://console.aws.amazon.com/)
- [Billing Dashboard](https://console.aws.amazon.com/billing/)
- [IAM Console](https://console.aws.amazon.com/iam/)
- [Cost Explorer](https://console.aws.amazon.com/cost-management/home)

## Next Steps

- [Set up AWS CLI](./aws-cli-setup.md)
- [Learn IAM basics](./iam-basics.md)

## Troubleshooting

### Issue: Account verification pending
- Wait 24 hours for account activation
- Check email for verification messages
- Contact AWS Support if issue persists

### Issue: Payment verification failed
- Verify card details are correct
- Ensure card is enabled for international transactions
- Try a different payment method

### Issue: Cannot access certain AWS services
- Some services are region-specific
- Check if service is available in your selected region
- Verify IAM permissions
