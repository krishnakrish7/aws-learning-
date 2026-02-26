# AWS Lambda

AWS Lambda is a serverless compute service that runs your code in response to events and automatically manages the computing resources.

## What is Lambda?

Lambda allows you to:
- Run code without provisioning or managing servers
- Pay only for compute time consumed
- Automatically scale based on demand
- Respond to events from various AWS services
- Support multiple programming languages

## Key Concepts

### Function
- Your code and configuration
- Triggered by events
- Executes in isolated environment

### Runtime
- Execution environment for your code
- Supported: Python, Node.js, Java, Go, .NET, Ruby, Custom Runtime

### Handler
- Method in your code that processes events
- Entry point for Lambda execution

### Event Source
- Service/application that triggers Lambda
- Examples: S3, API Gateway, DynamoDB, SNS, SQS, CloudWatch

### Execution Role
- IAM role Lambda assumes to access AWS services
- Defines permissions for your function

### Layers
- Reusable code/libraries
- Share code across multiple functions
- Manage dependencies separately

## Getting Started

### 1. Create Your First Lambda Function

#### Via AWS Console

1. Navigate to Lambda Console
2. Click "Create function"
3. Choose "Author from scratch"
4. Configure:
   - **Function name**: my-first-function
   - **Runtime**: Python 3.12
   - **Architecture**: x86_64
   - **Permissions**: Create new role with basic Lambda permissions
5. Click "Create function"

#### Via AWS CLI

```bash
# Create execution role
aws iam create-role \
    --role-name lambda-execution-role \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }]
    }'

# Attach basic execution policy
aws iam attach-role-policy \
    --role-name lambda-execution-role \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Create function code
cat > lambda_function.py <<EOF
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello from Lambda!'
    }
EOF

# Package code
zip function.zip lambda_function.py

# Create function
aws lambda create-function \
    --function-name my-first-function \
    --runtime python3.12 \
    --role arn:aws:iam::123456789012:role/lambda-execution-role \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

### 2. Test Your Function

#### Via Console
1. Click "Test" button
2. Create test event
3. Click "Test"
4. View execution results

#### Via CLI

```bash
# Invoke function
aws lambda invoke \
    --function-name my-first-function \
    --payload '{"key": "value"}' \
    response.json

# View response
cat response.json
```

## Lambda Function Examples

### Python Example

```python
import json
import boto3

def lambda_handler(event, context):
    # Log event
    print(f"Event: {json.dumps(event)}")
    
    # Process data
    name = event.get('name', 'World')
    
    # Return response
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': json.dumps({
            'message': f'Hello, {name}!',
            'input': event
        })
    }
```

### Node.js Example

```javascript
exports.handler = async (event) => {
    console.log('Event:', JSON.stringify(event, null, 2));
    
    const name = event.name || 'World';
    
    return {
        statusCode: 200,
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            message: `Hello, ${name}!`,
            input: event
        })
    };
};
```

### Java Example

```java
package example;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;

public class Handler implements RequestHandler<Map<String, String>, String> {
    @Override
    public String handleRequest(Map<String, String> event, Context context) {
        context.getLogger().log("Event: " + event);
        String name = event.getOrDefault("name", "World");
        return "Hello, " + name + "!";
    }
}
```

## Common Use Cases

### 1. S3 Event Processing

```python
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get S3 event details
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    print(f"Processing file: {key} from bucket: {bucket}")
    
    # Process file (example: get metadata)
    response = s3.head_object(Bucket=bucket, Key=key)
    
    return {
        'statusCode': 200,
        'body': json.dumps(f'Processed {key}')
    }
```

### 2. API Gateway Integration

```python
import json

def lambda_handler(event, context):
    # Parse request
    http_method = event['httpMethod']
    path = event['path']
    
    if http_method == 'GET':
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps({'message': 'GET request received'})
        }
    elif http_method == 'POST':
        body = json.loads(event['body'])
        return {
            'statusCode': 201,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps({'message': 'POST request received', 'data': body})
        }
    else:
        return {
            'statusCode': 405,
            'body': json.dumps({'error': 'Method not allowed'})
        }
```

### 3. DynamoDB Stream Processing

```python
import json

def lambda_handler(event, context):
    for record in event['Records']:
        if record['eventName'] == 'INSERT':
            new_image = record['dynamodb']['NewImage']
            print(f"New item: {new_image}")
        elif record['eventName'] == 'MODIFY':
            old_image = record['dynamodb']['OldImage']
            new_image = record['dynamodb']['NewImage']
            print(f"Modified: {old_image} -> {new_image}")
        elif record['eventName'] == 'REMOVE':
            old_image = record['dynamodb']['OldImage']
            print(f"Deleted: {old_image}")
    
    return {'statusCode': 200}
```

### 4. Scheduled Tasks (CloudWatch Events)

```python
import boto3
import datetime

ec2 = boto3.client('ec2')

def lambda_handler(event, context):
    # Stop all dev instances at night
    instances = ec2.describe_instances(
        Filters=[{'Name': 'tag:Environment', 'Values': ['dev']}]
    )
    
    instance_ids = []
    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            if instance['State']['Name'] == 'running':
                instance_ids.append(instance['InstanceId'])
    
    if instance_ids:
        ec2.stop_instances(InstanceIds=instance_ids)
        print(f"Stopped instances: {instance_ids}")
    
    return {'statusCode': 200, 'stopped': instance_ids}
```

## Environment Variables

```bash
# Set environment variables
aws lambda update-function-configuration \
    --function-name my-function \
    --environment Variables={
        DB_HOST=database.example.com,
        DB_NAME=mydb,
        API_KEY=secret123
    }
```

Access in Python:
```python
import os

db_host = os.environ['DB_HOST']
db_name = os.environ['DB_NAME']
```

## Lambda Layers

Create and use layers:

```bash
# Create layer structure
mkdir -p layer/python/lib/python3.12/site-packages
pip install requests -t layer/python/lib/python3.12/site-packages

# Package layer
cd layer
zip -r ../layer.zip .
cd ..

# Publish layer
aws lambda publish-layer-version \
    --layer-name my-dependencies \
    --description "Common dependencies" \
    --zip-file fileb://layer.zip \
    --compatible-runtimes python3.12

# Add layer to function
aws lambda update-function-configuration \
    --function-name my-function \
    --layers arn:aws:lambda:us-east-1:123456789012:layer:my-dependencies:1
```

## Function Configuration

### Memory and Timeout

```bash
# Update memory (128 MB to 10,240 MB)
aws lambda update-function-configuration \
    --function-name my-function \
    --memory-size 512

# Update timeout (max 900 seconds / 15 minutes)
aws lambda update-function-configuration \
    --function-name my-function \
    --timeout 300
```

### Concurrency

```bash
# Set reserved concurrency
aws lambda put-function-concurrency \
    --function-name my-function \
    --reserved-concurrent-executions 10

# Set provisioned concurrency
aws lambda put-provisioned-concurrency-config \
    --function-name my-function \
    --provisioned-concurrent-executions 5 \
    --qualifier $LATEST
```

## Monitoring and Logging

### CloudWatch Logs

```python
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info('Processing event')
    logger.error('An error occurred')
    return {'statusCode': 200}
```

### View Logs

```bash
# Get log groups
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/

# Get recent logs
aws logs tail /aws/lambda/my-function --follow
```

### CloudWatch Metrics

```bash
# Get invocation count
aws cloudwatch get-metric-statistics \
    --namespace AWS/Lambda \
    --metric-name Invocations \
    --dimensions Name=FunctionName,Value=my-function \
    --start-time 2024-01-01T00:00:00Z \
    --end-time 2024-01-02T00:00:00Z \
    --period 3600 \
    --statistics Sum
```

## Lambda Triggers

### S3 Trigger

```bash
# Add S3 permission
aws lambda add-permission \
    --function-name my-function \
    --statement-id s3-trigger \
    --action lambda:InvokeFunction \
    --principal s3.amazonaws.com \
    --source-arn arn:aws:s3:::my-bucket

# Configure S3 notification
aws s3api put-bucket-notification-configuration \
    --bucket my-bucket \
    --notification-configuration '{
        "LambdaFunctionConfigurations": [{
            "LambdaFunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-function",
            "Events": ["s3:ObjectCreated:*"]
        }]
    }'
```

### API Gateway Trigger

```bash
# Create REST API
aws apigateway create-rest-api --name my-api

# Create Lambda integration
# (detailed steps in API Gateway documentation)
```

### EventBridge (CloudWatch Events) Trigger

```bash
# Create rule
aws events put-rule \
    --name daily-task \
    --schedule-expression "rate(1 day)"

# Add Lambda target
aws events put-targets \
    --rule daily-task \
    --targets "Id=1,Arn=arn:aws:lambda:us-east-1:123456789012:function:my-function"

# Add permission
aws lambda add-permission \
    --function-name my-function \
    --statement-id eventbridge-trigger \
    --action lambda:InvokeFunction \
    --principal events.amazonaws.com \
    --source-arn arn:aws:events:us-east-1:123456789012:rule/daily-task
```

## Best Practices

1. **Minimize Package Size**: Include only necessary dependencies
2. **Use Environment Variables**: For configuration
3. **Implement Idempotency**: Handle duplicate events
4. **Optimize Cold Starts**: Keep functions warm if needed
5. **Use Layers**: Share common code/dependencies
6. **Monitor Performance**: Use CloudWatch and X-Ray
7. **Set Appropriate Timeouts**: Don't use maximum unless needed
8. **Handle Errors Gracefully**: Implement retry logic
9. **Use IAM Roles**: Least privilege principle
10. **Version Functions**: Use aliases for production

## Cost Optimization

- **Right-size memory**: Affects both performance and cost
- **Optimize execution time**: Reduce billable duration
- **Use ARM architecture**: Graviton2 processors (20% cost savings)
- **Delete unused functions**: Clean up regularly
- **Use provisioned concurrency sparingly**: Only when needed
- **Monitor invocations**: Detect inefficiencies

## Debugging

### Local Testing with SAM

```bash
# Install SAM CLI
pip install aws-sam-cli

# Create template
sam init

# Test locally
sam local invoke MyFunction -e event.json

# Start API locally
sam local start-api
```

### AWS X-Ray

```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

patch_all()

@xray_recorder.capture('my_function')
def lambda_handler(event, context):
    # Your code here
    pass
```

## Troubleshooting

### Common Errors

1. **Timeout**: Increase timeout or optimize code
2. **Out of Memory**: Increase memory allocation
3. **Cold Start**: Use provisioned concurrency or optimize initialization
4. **Permission Denied**: Check IAM role permissions
5. **Package Too Large**: Use layers or optimize dependencies

## Example Projects

1. **Image Thumbnail Generator**: Process S3 uploads
2. **REST API**: Serverless API with API Gateway
3. **Data Pipeline**: Process streams from DynamoDB/Kinesis
4. **Chatbot**: Integrate with Slack/Teams
5. **Scheduled Reports**: Generate and email reports

## Additional Resources

- [Lambda Developer Guide](https://docs.aws.amazon.com/lambda/)
- [Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
- [Serverless Application Model (SAM)](https://aws.amazon.com/serverless/sam/)

## Next Steps

- [Learn about API Gateway](../../networking/api-gateway/)
- [Explore DynamoDB](../../database/dynamodb/)
- [Understand S3](../../storage/s3/)
