# AWS Lambda - Complete Practical Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Key Concepts](#key-concepts)
3. [Function Management](#function-management)
4. [Deployment & Code Updates](#deployment--code-updates)
5. [Configuration](#configuration)
6. [Environment Variables & Secrets](#environment-variables--secrets)
7. [IAM Roles & Permissions](#iam-roles--permissions)
8. [Event Sources & Triggers](#event-sources--triggers)
9. [Layers](#layers)
10. [Versions & Aliases](#versions--aliases)
11. [Concurrency & Scaling](#concurrency--scaling)
12. [Monitoring & Logging](#monitoring--logging)
13. [Testing & Invocation](#testing--invocation)
14. [VPC Configuration](#vpc-configuration)
15. [Best Practices](#best-practices)

---

## Introduction

AWS Lambda is a serverless compute service that runs your code in response to events and automatically manages the underlying compute resources. You pay only for the compute time you consume.

**Use Cases:**
- API backends (with API Gateway)
- Data processing (S3, DynamoDB streams)
- Real-time file processing
- Scheduled tasks (cron jobs)
- ETL pipelines
- IoT backends
- Chatbots and Alexa skills

**Benefits:**
- No server management
- Automatic scaling
- Pay-per-use pricing
- Built-in fault tolerance
- Integrated with AWS services

---

## Key Concepts

### Functions
- Code + configuration that Lambda executes
- Triggered by events or direct invocation
- Stateless (ephemeral storage in /tmp)
- Maximum execution time: 15 minutes

### Runtime
- Execution environment for your code
- Supports: Python, Node.js, Java, Go, .NET, Ruby, Custom Runtime (containers)

### Handler
- Method/function that Lambda calls to start execution
- Format: `filename.function_name`
- Example: `index.handler` or `lambda_function.lambda_handler`

### Execution Role
- IAM role that grants Lambda permissions to AWS services
- Required for every function

### Event Sources
- Services that trigger Lambda functions
- S3, DynamoDB, Kinesis, SQS, SNS, API Gateway, EventBridge, etc.

### Invocation Types
- **Synchronous**: Wait for response (API Gateway, direct invoke)
- **Asynchronous**: Don't wait for response (S3, SNS)
- **Stream-based**: Poll and invoke (DynamoDB Streams, Kinesis)

---

## Function Management

### 1. Create a Lambda Function (Python)

**First, create the function code:**
```python
# lambda_function.py
def lambda_handler(event, context):
    print(f"Event received: {event}")
    return {
        'statusCode': 200,
        'body': 'Hello from Lambda!'
    }
```

**Zip the code:**
```bash
zip function.zip lambda_function.py
```

**Create the function:**
```bash
aws lambda create-function \
    --function-name my-first-function \
    --runtime python3.11 \
    --role arn:aws:iam::123456789012:role/lambda-execution-role \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip \
    --description "My first Lambda function"
```

**Expected Output:**
```json
{
    "FunctionName": "my-first-function",
    "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function",
    "Runtime": "python3.11",
    "Role": "arn:aws:iam::123456789012:role/lambda-execution-role",
    "Handler": "lambda_function.lambda_handler",
    "CodeSize": 325,
    "Description": "My first Lambda function",
    "Timeout": 3,
    "MemorySize": 128,
    "LastModified": "2025-11-17T10:30:00.000+0000",
    "State": "Active"
}
```

**Why:** Creates a new Lambda function with specified configuration.

**When to Use:** Starting a new serverless application, adding new functionality.

**LocalStack:**
```bash
aws lambda create-function --function-name my-first-function --runtime python3.11 --role arn:aws:iam::123456789012:role/lambda-execution-role --handler lambda_function.lambda_handler --zip-file fileb://function.zip --endpoint-url=http://localhost:4566
```

---

### 2. List Functions

```bash
aws lambda list-functions
```

**Expected Output:**
```json
{
    "Functions": [
        {
            "FunctionName": "my-first-function",
            "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function",
            "Runtime": "python3.11",
            "Role": "arn:aws:iam::123456789012:role/lambda-execution-role",
            "Handler": "lambda_function.lambda_handler",
            "CodeSize": 325,
            "LastModified": "2025-11-17T10:30:00.000+0000",
            "Timeout": 3,
            "MemorySize": 128
        }
    ]
}
```

**Why:** View all Lambda functions in your account/region.

**When to Use:** Auditing, inventory, finding function names.

**Filter by runtime:**
```bash
aws lambda list-functions --query "Functions[?Runtime=='python3.11']"
```

---

### 3. Get Function Configuration

```bash
aws lambda get-function-configuration --function-name my-first-function
```

**Expected Output:**
```json
{
    "FunctionName": "my-first-function",
    "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function",
    "Runtime": "python3.11",
    "Role": "arn:aws:iam::123456789012:role/lambda-execution-role",
    "Handler": "lambda_function.lambda_handler",
    "CodeSize": 325,
    "Timeout": 3,
    "MemorySize": 128,
    "LastModified": "2025-11-17T10:30:00.000+0000",
    "State": "Active",
    "Environment": {
        "Variables": {}
    }
}
```

**Why:** View function settings without downloading code.

**When to Use:** Checking configuration, troubleshooting, documentation.

---

### 4. Get Function Details (including code location)

```bash
aws lambda get-function --function-name my-first-function
```

**Expected Output:**
```json
{
    "Configuration": {
        "FunctionName": "my-first-function",
        "Runtime": "python3.11",
        "Handler": "lambda_function.lambda_handler",
        ...
    },
    "Code": {
        "RepositoryType": "S3",
        "Location": "https://awslambda-us-east-1-tasks.s3.us-east-1.amazonaws.com/snapshots/..."
    }
}
```

**Why:** Get complete function details including pre-signed URL to download code.

**When to Use:** Backup code, review deployed code, debugging.

---

### 5. Delete Function

```bash
aws lambda delete-function --function-name my-first-function
```

**Expected Output:**
```
(No output on success - HTTP 204)
```

**Why:** Remove function and all versions/aliases.

**When to Use:** Cleanup, decommissioning, removing test functions.

---

## Deployment & Code Updates

### 1. Update Function Code (from zip file)

```bash
# Update code
zip function.zip lambda_function.py

# Deploy update
aws lambda update-function-code \
    --function-name my-first-function \
    --zip-file fileb://function.zip
```

**Expected Output:**
```json
{
    "FunctionName": "my-first-function",
    "LastModified": "2025-11-17T11:00:00.000+0000",
    "CodeSize": 412,
    "State": "Active"
}
```

**Why:** Deploy new code to existing function.

**When to Use:** Bug fixes, feature updates, continuous deployment.

---

### 2. Update Function Code (from S3)

```bash
# Upload code to S3 first
aws s3 cp function.zip s3://my-lambda-code-bucket/function.zip

# Update Lambda from S3
aws lambda update-function-code \
    --function-name my-first-function \
    --s3-bucket my-lambda-code-bucket \
    --s3-key function.zip
```

**Expected Output:**
```json
{
    "FunctionName": "my-first-function",
    "LastModified": "2025-11-17T11:05:00.000+0000",
    "CodeSize": 412
}
```

**Why:** Update code from S3 (better for large deployments, CI/CD pipelines).

**When to Use:** Large packages (>50MB), automated deployments, team workflows.

---

### 3. Create Function with Dependencies (Python)

```bash
# Create project structure
mkdir lambda-project
cd lambda-project

# Install dependencies
pip install requests -t .

# Create function
cat > lambda_function.py << 'EOF'
import requests

def lambda_handler(event, context):
    response = requests.get('https://api.github.com')
    return {
        'statusCode': 200,
        'body': response.text
    }
EOF

# Zip everything
zip -r function.zip .

# Deploy
aws lambda update-function-code --function-name my-first-function --zip-file fileb://function.zip
```

**Why:** Include external libraries/dependencies in function package.

**When to Use:** Using third-party libraries, complex applications.

---

### 4. Create Node.js Function

**Create package.json:**
```json
{
  "name": "my-lambda-function",
  "version": "1.0.0",
  "dependencies": {
    "axios": "^1.6.0"
  }
}
```

**Create index.js:**
```javascript
const axios = require('axios');

exports.handler = async (event) => {
    const response = await axios.get('https://api.github.com');
    return {
        statusCode: 200,
        body: JSON.stringify(response.data)
    };
};
```

**Deploy:**
```bash
npm install
zip -r function.zip .
aws lambda create-function --function-name my-node-function --runtime nodejs20.x --role arn:aws:iam::123456789012:role/lambda-role --handler index.handler --zip-file fileb://function.zip
```

**Why:** Create Lambda function with Node.js runtime.

**When to Use:** JavaScript/TypeScript applications, npm ecosystem.

---

### 5. Deploy from Container Image

**Create Dockerfile:**
```dockerfile
FROM public.ecr.aws/lambda/python:3.11

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY lambda_function.py .

CMD ["lambda_function.lambda_handler"]
```

**Build and push:**
```bash
# Build image
docker build -t my-lambda-function .

# Tag for ECR
docker tag my-lambda-function:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-lambda-function:latest

# Push to ECR
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-lambda-function:latest

# Create function from image
aws lambda create-function \
    --function-name my-container-function \
    --package-type Image \
    --code ImageUri=123456789012.dkr.ecr.us-east-1.amazonaws.com/my-lambda-function:latest \
    --role arn:aws:iam::123456789012:role/lambda-role
```

**Expected Output:**
```json
{
    "FunctionName": "my-container-function",
    "PackageType": "Image",
    "CodeSize": 0,
    "State": "Pending"
}
```

**Why:** Use custom runtime, large dependencies (>250MB), Docker workflow.

**When to Use:** Complex dependencies, custom OS packages, team uses Docker.

---

## Configuration

### 1. Update Function Configuration

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --timeout 30 \
    --memory-size 512 \
    --description "Updated function configuration"
```

**Expected Output:**
```json
{
    "FunctionName": "my-first-function",
    "Timeout": 30,
    "MemorySize": 512,
    "Description": "Updated function configuration",
    "LastModified": "2025-11-17T11:15:00.000+0000"
}
```

**Why:** Adjust function settings without redeploying code.

**When to Use:** Performance tuning, timeout adjustments, memory optimization.

**Memory:** 128MB - 10,240MB (CPU scales with memory)
**Timeout:** 1 second - 900 seconds (15 minutes)

---

### 2. Set Function Timeout

```bash
aws lambda update-function-configuration --function-name my-first-function --timeout 60
```

**Why:** Prevent functions from running too long and incurring costs.

**When to Use:** Processing tasks with known duration, cost control.

---

### 3. Set Function Memory

```bash
aws lambda update-function-configuration --function-name my-first-function --memory-size 1024
```

**Why:** More memory = more CPU power. Optimize performance vs. cost.

**When to Use:** CPU-intensive tasks, performance optimization.

**Note:** Memory allocated in 1MB increments from 128MB to 10,240MB. CPU scales proportionally.

---

### 4. Update Handler

```bash
aws lambda update-function-configuration --function-name my-first-function --handler new_module.handler_function
```

**Why:** Change entry point without redeploying code.

**When to Use:** Testing different handlers, reorganizing code.

---

### 5. Update Runtime

```bash
aws lambda update-function-configuration --function-name my-first-function --runtime python3.12
```

**Why:** Upgrade to newer runtime version for security/features.

**When to Use:** Runtime deprecation, new features, security patches.

---

## Environment Variables & Secrets

### 1. Set Environment Variables

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --environment "Variables={DB_HOST=db.example.com,DB_PORT=5432,LOG_LEVEL=INFO}"
```

**Expected Output:**
```json
{
    "FunctionName": "my-first-function",
    "Environment": {
        "Variables": {
            "DB_HOST": "db.example.com",
            "DB_PORT": "5432",
            "LOG_LEVEL": "INFO"
        }
    }
}
```

**Why:** Configure function without hardcoding values in code.

**When to Use:** Configuration management, different environments (dev/prod).

**Access in Python:**
```python
import os
db_host = os.environ['DB_HOST']
```

---

### 2. Get Environment Variables

```bash
aws lambda get-function-configuration --function-name my-first-function --query 'Environment'
```

**Expected Output:**
```json
{
    "Variables": {
        "DB_HOST": "db.example.com",
        "DB_PORT": "5432",
        "LOG_LEVEL": "INFO"
    }
}
```

**Why:** View current environment variables for debugging.

**When to Use:** Troubleshooting, documentation, auditing.

---

### 3. Encrypt Environment Variables (KMS)

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --kms-key-arn arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012
```

**Expected Output:**
```json
{
    "KMSKeyArn": "arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012"
}
```

**Why:** Encrypt sensitive environment variables at rest.

**When to Use:** Secrets, credentials, compliance requirements.

**Note:** Variables are always encrypted in transit. KMS adds encryption at rest.

---

### 4. Use AWS Secrets Manager (Best Practice)

**Lambda code:**
```python
import boto3
import json

secrets_client = boto3.client('secretsmanager')

def lambda_handler(event, context):
    secret_response = secrets_client.get_secret_value(SecretId='my-database-secret')
    secret = json.loads(secret_response['SecretString'])

    db_password = secret['password']
    # Use password...

    return {'statusCode': 200}
```

**Why:** Centralized secret management, rotation, auditing.

**When to Use:** Passwords, API keys, certificates, sensitive data.

---

## IAM Roles & Permissions

### 1. Create Execution Role

**Create trust policy (trust-policy.json):**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "lambda.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

**Create role:**
```bash
aws iam create-role \
    --role-name lambda-execution-role \
    --assume-role-policy-document file://trust-policy.json
```

**Expected Output:**
```json
{
    "Role": {
        "RoleName": "lambda-execution-role",
        "Arn": "arn:aws:iam::123456789012:role/lambda-execution-role",
        "CreateDate": "2025-11-17T11:30:00+00:00"
    }
}
```

**Why:** Lambda needs permissions to execute and access AWS services.

**When to Use:** Creating new Lambda functions.

---

### 2. Attach Basic Execution Policy

```bash
aws iam attach-role-policy \
    --role-name lambda-execution-role \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

**Expected Output:**
```
(No output on success)
```

**Why:** Grants CloudWatch Logs permissions (required for logging).

**When to Use:** All Lambda functions need this for basic logging.

**Permissions included:**
- Create log groups
- Create log streams
- Put log events

---

### 3. Attach Additional Policies

```bash
# S3 read access
aws iam attach-role-policy \
    --role-name lambda-execution-role \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# DynamoDB full access
aws iam attach-role-policy \
    --role-name lambda-execution-role \
    --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess
```

**Why:** Grant function access to specific AWS services.

**When to Use:** Function needs to interact with S3, DynamoDB, SQS, etc.

---

### 4. Create Custom Policy

**policy.json:**
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
            "Resource": "arn:aws:s3:::my-specific-bucket/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetItem",
                "dynamodb:PutItem"
            ],
            "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/MyTable"
        }
    ]
}
```

```bash
aws iam create-policy --policy-name lambda-custom-policy --policy-document file://policy.json

aws iam attach-role-policy \
    --role-name lambda-execution-role \
    --policy-arn arn:aws:iam::123456789012:policy/lambda-custom-policy
```

**Why:** Least privilege principle - grant only needed permissions.

**When to Use:** Production functions, security best practices.

---

### 5. Update Function Role

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --role arn:aws:iam::123456789012:role/new-lambda-role
```

**Expected Output:**
```json
{
    "Role": "arn:aws:iam::123456789012:role/new-lambda-role"
}
```

**Why:** Change function permissions without recreating function.

**When to Use:** Permission updates, security hardening.

---

## Event Sources & Triggers

### 1. Add S3 Trigger

```bash
# Grant S3 permission to invoke Lambda
aws lambda add-permission \
    --function-name my-first-function \
    --statement-id s3-trigger-permission \
    --action lambda:InvokeFunction \
    --principal s3.amazonaws.com \
    --source-arn arn:aws:s3:::my-bucket

# Configure S3 notification
aws s3api put-bucket-notification-configuration \
    --bucket my-bucket \
    --notification-configuration file://s3-notification.json
```

**s3-notification.json:**
```json
{
    "LambdaFunctionConfigurations": [
        {
            "LambdaFunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function",
            "Events": ["s3:ObjectCreated:*"],
            "Filter": {
                "Key": {
                    "FilterRules": [
                        {
                            "Name": "prefix",
                            "Value": "uploads/"
                        },
                        {
                            "Name": "suffix",
                            "Value": ".jpg"
                        }
                    ]
                }
            }
        }
    ]
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Process files automatically when uploaded to S3.

**When to Use:** Image processing, data transformation, file validation.

---

### 2. Add DynamoDB Stream Trigger

```bash
# Create event source mapping
aws lambda create-event-source-mapping \
    --function-name my-first-function \
    --event-source-arn arn:aws:dynamodb:us-east-1:123456789012:table/MyTable/stream/2025-11-17T00:00:00.000 \
    --starting-position LATEST \
    --batch-size 100
```

**Expected Output:**
```json
{
    "UUID": "a1b2c3d4-5678-90ab-cdef-EXAMPLE11111",
    "BatchSize": 100,
    "EventSourceArn": "arn:aws:dynamodb:us-east-1:123456789012:table/MyTable/stream/2025-11-17T00:00:00.000",
    "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function",
    "LastModified": "2025-11-17T12:00:00+00:00",
    "State": "Creating",
    "StateTransitionReason": "User action"
}
```

**Why:** React to changes in DynamoDB table (insert, update, delete).

**When to Use:** Data synchronization, audit logging, derived data.

**Starting Positions:**
- LATEST: Start reading new records
- TRIM_HORIZON: Start from oldest record
- AT_TIMESTAMP: Start from specific time

---

### 3. Add SQS Trigger

```bash
aws lambda create-event-source-mapping \
    --function-name my-first-function \
    --event-source-arn arn:aws:sqs:us-east-1:123456789012:my-queue \
    --batch-size 10 \
    --maximum-batching-window-in-seconds 5
```

**Expected Output:**
```json
{
    "UUID": "a1b2c3d4-5678-90ab-cdef-EXAMPLE22222",
    "BatchSize": 10,
    "MaximumBatchingWindowInSeconds": 5,
    "EventSourceArn": "arn:aws:sqs:us-east-1:123456789012:my-queue",
    "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function",
    "State": "Creating"
}
```

**Why:** Process messages from SQS queue asynchronously.

**When to Use:** Decoupled architectures, job processing, message handling.

**Note:** Lambda polls SQS automatically. Messages deleted if function succeeds.

---

### 4. List Event Source Mappings

```bash
aws lambda list-event-source-mappings --function-name my-first-function
```

**Expected Output:**
```json
{
    "EventSourceMappings": [
        {
            "UUID": "a1b2c3d4-5678-90ab-cdef-EXAMPLE11111",
            "BatchSize": 100,
            "EventSourceArn": "arn:aws:dynamodb:...",
            "FunctionArn": "arn:aws:lambda:...",
            "State": "Enabled"
        }
    ]
}
```

**Why:** View all triggers configured for function.

**When to Use:** Auditing, troubleshooting, documentation.

---

### 5. Update Event Source Mapping

```bash
aws lambda update-event-source-mapping \
    --uuid a1b2c3d4-5678-90ab-cdef-EXAMPLE11111 \
    --batch-size 50 \
    --enabled
```

**Expected Output:**
```json
{
    "UUID": "a1b2c3d4-5678-90ab-cdef-EXAMPLE11111",
    "BatchSize": 50,
    "State": "Updating"
}
```

**Why:** Modify trigger settings (batch size, enable/disable).

**When to Use:** Performance tuning, temporary disable during maintenance.

---

### 6. Delete Event Source Mapping

```bash
aws lambda delete-event-source-mapping --uuid a1b2c3d4-5678-90ab-cdef-EXAMPLE11111
```

**Expected Output:**
```json
{
    "UUID": "a1b2c3d4-5678-90ab-cdef-EXAMPLE11111",
    "State": "Deleting"
}
```

**Why:** Remove trigger from function.

**When to Use:** Changing architecture, cleanup, troubleshooting.

---

### 7. Add API Gateway Trigger

```bash
# Grant API Gateway permission
aws lambda add-permission \
    --function-name my-first-function \
    --statement-id apigateway-invoke \
    --action lambda:InvokeFunction \
    --principal apigateway.amazonaws.com \
    --source-arn "arn:aws:execute-api:us-east-1:123456789012:api-id/*/*"
```

**Expected Output:**
```json
{
    "Statement": "{\"Sid\":\"apigateway-invoke\",\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"apigateway.amazonaws.com\"},\"Action\":\"lambda:InvokeFunction\",\"Resource\":\"arn:aws:lambda:us-east-1:123456789012:function:my-first-function\"}"
}
```

**Why:** Allow API Gateway to invoke your function.

**When to Use:** Building REST APIs, HTTP APIs, WebSocket APIs.

---

### 8. Add EventBridge (CloudWatch Events) Trigger

```bash
# Create rule
aws events put-rule \
    --name my-scheduled-rule \
    --schedule-expression "rate(5 minutes)"

# Add Lambda as target
aws events put-targets \
    --rule my-scheduled-rule \
    --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:123456789012:function:my-first-function"

# Grant permission
aws lambda add-permission \
    --function-name my-first-function \
    --statement-id eventbridge-invoke \
    --action lambda:InvokeFunction \
    --principal events.amazonaws.com \
    --source-arn arn:aws:events:us-east-1:123456789012:rule/my-scheduled-rule
```

**Expected Output:**
```json
{
    "Statement": "{...}"
}
```

**Why:** Schedule function execution (cron jobs) or react to AWS events.

**When to Use:** Scheduled tasks, infrastructure monitoring, custom events.

**Schedule expressions:**
- `rate(5 minutes)`
- `rate(1 hour)`
- `cron(0 12 * * ? *)` - Daily at 12:00 UTC

---

## Layers

### 1. Create Layer

```bash
# Package dependencies
mkdir python
pip install requests -t python/
zip -r layer.zip python/

# Create layer
aws lambda publish-layer-version \
    --layer-name my-dependencies-layer \
    --description "Common dependencies" \
    --zip-file fileb://layer.zip \
    --compatible-runtimes python3.11 python3.12
```

**Expected Output:**
```json
{
    "LayerArn": "arn:aws:lambda:us-east-1:123456789012:layer:my-dependencies-layer",
    "LayerVersionArn": "arn:aws:lambda:us-east-1:123456789012:layer:my-dependencies-layer:1",
    "Version": 1,
    "CompatibleRuntimes": ["python3.11", "python3.12"],
    "CreatedDate": "2025-11-17T13:00:00.000+0000"
}
```

**Why:** Share code/dependencies across multiple functions. Reduces deployment package size.

**When to Use:** Common libraries, custom utilities, large dependencies.

**Structure for Python:**
```
layer.zip
└── python/
    └── (libraries)
```

**Structure for Node.js:**
```
layer.zip
└── nodejs/
    └── node_modules/
        └── (libraries)
```

---

### 2. List Layers

```bash
aws lambda list-layers
```

**Expected Output:**
```json
{
    "Layers": [
        {
            "LayerName": "my-dependencies-layer",
            "LayerArn": "arn:aws:lambda:us-east-1:123456789012:layer:my-dependencies-layer",
            "LatestMatchingVersion": {
                "LayerVersionArn": "arn:aws:lambda:us-east-1:123456789012:layer:my-dependencies-layer:1",
                "Version": 1,
                "CompatibleRuntimes": ["python3.11"]
            }
        }
    ]
}
```

**Why:** View available layers in account.

**When to Use:** Finding layer ARNs, auditing.

---

### 3. List Layer Versions

```bash
aws lambda list-layer-versions --layer-name my-dependencies-layer
```

**Expected Output:**
```json
{
    "LayerVersions": [
        {
            "LayerVersionArn": "arn:aws:lambda:us-east-1:123456789012:layer:my-dependencies-layer:2",
            "Version": 2,
            "CreatedDate": "2025-11-17T14:00:00.000+0000"
        },
        {
            "LayerVersionArn": "arn:aws:lambda:us-east-1:123456789012:layer:my-dependencies-layer:1",
            "Version": 1,
            "CreatedDate": "2025-11-17T13:00:00.000+0000"
        }
    ]
}
```

**Why:** View layer version history.

**When to Use:** Selecting specific version, rollback.

---

### 4. Attach Layer to Function

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --layers arn:aws:lambda:us-east-1:123456789012:layer:my-dependencies-layer:1
```

**Expected Output:**
```json
{
    "FunctionName": "my-first-function",
    "Layers": [
        {
            "Arn": "arn:aws:lambda:us-east-1:123456789012:layer:my-dependencies-layer:1"
        }
    ]
}
```

**Why:** Add shared code/dependencies to function.

**When to Use:** Using common libraries, reducing deployment size.

**Note:** Maximum 5 layers per function. Total unzipped size limit: 250MB.

---

### 5. Attach Multiple Layers

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --layers \
        arn:aws:lambda:us-east-1:123456789012:layer:layer1:1 \
        arn:aws:lambda:us-east-1:123456789012:layer:layer2:3
```

**Why:** Use multiple shared libraries/utilities.

**When to Use:** Complex functions with multiple dependencies.

**Layer order matters:** Lambda merges layers in order. Later layers override earlier ones.

---

### 6. Delete Layer Version

```bash
aws lambda delete-layer-version --layer-name my-dependencies-layer --version-number 1
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove old/unused layer versions.

**When to Use:** Cleanup, version management.

**Note:** Functions using that version will continue working (immutable).

---

## Versions & Aliases

### 1. Publish Version

```bash
aws lambda publish-version --function-name my-first-function --description "Production release v1.0"
```

**Expected Output:**
```json
{
    "FunctionName": "my-first-function",
    "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function:1",
    "Version": "1",
    "Description": "Production release v1.0",
    "LastModified": "2025-11-17T14:30:00.000+0000",
    "CodeSize": 512
}
```

**Why:** Create immutable snapshot of function code + configuration.

**When to Use:** Production deployments, rollback capability, traffic splitting.

**Note:** $LATEST = unpublished version (mutable). Numbered versions = immutable.

---

### 2. List Versions

```bash
aws lambda list-versions-by-function --function-name my-first-function
```

**Expected Output:**
```json
{
    "Versions": [
        {
            "FunctionName": "my-first-function",
            "Version": "$LATEST",
            "LastModified": "2025-11-17T15:00:00.000+0000"
        },
        {
            "FunctionName": "my-first-function",
            "Version": "2",
            "LastModified": "2025-11-17T14:45:00.000+0000"
        },
        {
            "FunctionName": "my-first-function",
            "Version": "1",
            "LastModified": "2025-11-17T14:30:00.000+0000"
        }
    ]
}
```

**Why:** View version history for rollback or reference.

**When to Use:** Auditing, rollback, documentation.

---

### 3. Create Alias

```bash
aws lambda create-alias \
    --function-name my-first-function \
    --name production \
    --function-version 1 \
    --description "Production environment"
```

**Expected Output:**
```json
{
    "AliasArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function:production",
    "Name": "production",
    "FunctionVersion": "1",
    "Description": "Production environment"
}
```

**Why:** Create pointer to specific version. Easier to manage than version numbers.

**When to Use:** Blue/green deployments, environment separation (prod/staging/dev).

**Benefits:**
- Stable endpoint for clients
- Traffic splitting between versions
- Easy rollback by updating alias

---

### 4. Update Alias (Rollback/Upgrade)

```bash
aws lambda update-alias \
    --function-name my-first-function \
    --name production \
    --function-version 2
```

**Expected Output:**
```json
{
    "AliasArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function:production",
    "Name": "production",
    "FunctionVersion": "2",
    "Description": "Production environment"
}
```

**Why:** Point alias to different version (deployment or rollback).

**When to Use:** Deployments, rollback after issues, testing.

---

### 5. Traffic Splitting (Weighted Alias)

```bash
aws lambda update-alias \
    --function-name my-first-function \
    --name production \
    --function-version 2 \
    --routing-config AdditionalVersionWeights={"1"=0.9}
```

**Expected Output:**
```json
{
    "AliasArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function:production",
    "Name": "production",
    "FunctionVersion": "2",
    "RoutingConfig": {
        "AdditionalVersionWeights": {
            "1": 0.9
        }
    }
}
```

**Why:** Canary or blue/green deployments. Gradually shift traffic to new version.

**When to Use:** Safe production deployments, A/B testing.

**Example:** 90% traffic to version 1, 10% to version 2 (specified in FunctionVersion).

---

### 6. List Aliases

```bash
aws lambda list-aliases --function-name my-first-function
```

**Expected Output:**
```json
{
    "Aliases": [
        {
            "AliasArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function:production",
            "Name": "production",
            "FunctionVersion": "2"
        },
        {
            "AliasArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function:staging",
            "Name": "staging",
            "FunctionVersion": "$LATEST"
        }
    ]
}
```

**Why:** View all aliases for function.

**When to Use:** Auditing, finding alias names, documentation.

---

### 7. Delete Alias

```bash
aws lambda delete-alias --function-name my-first-function --name staging
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove unused aliases.

**When to Use:** Cleanup, environment decommissioning.

---

## Concurrency & Scaling

### 1. Set Reserved Concurrency

```bash
aws lambda put-function-concurrency --function-name my-first-function --reserved-concurrent-executions 100
```

**Expected Output:**
```json
{
    "ReservedConcurrentExecutions": 100
}
```

**Why:** Guarantee capacity for function. Limits max concurrent executions.

**When to Use:** Critical functions, preventing throttling, cost control.

**Account Limit:** 1000 concurrent executions by default (can be increased).

**Note:** Reserved concurrency reduces available concurrency for other functions.

---

### 2. Get Function Concurrency

```bash
aws lambda get-function-concurrency --function-name my-first-function
```

**Expected Output:**
```json
{
    "ReservedConcurrentExecutions": 100
}
```

**Why:** View reserved concurrency settings.

**When to Use:** Troubleshooting throttling, capacity planning.

---

### 3. Delete Reserved Concurrency

```bash
aws lambda delete-function-concurrency --function-name my-first-function
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove concurrency limit, allow unrestricted scaling.

**When to Use:** Removing artificial limits, capacity planning changes.

---

### 4. Set Provisioned Concurrency

```bash
aws lambda put-provisioned-concurrency-config \
    --function-name my-first-function \
    --provisioned-concurrent-executions 5 \
    --qualifier production
```

**Expected Output:**
```json
{
    "RequestedProvisionedConcurrentExecutions": 5,
    "AvailableProvisionedConcurrentExecutions": 0,
    "AllocatedProvisionedConcurrentExecutions": 0,
    "Status": "IN_PROGRESS"
}
```

**Why:** Pre-warm execution environments to eliminate cold starts.

**When to Use:** Latency-sensitive applications, consistent performance.

**Cost:** Pay for provisioned concurrency even when idle.

---

### 5. Get Provisioned Concurrency

```bash
aws lambda get-provisioned-concurrency-config --function-name my-first-function --qualifier production
```

**Expected Output:**
```json
{
    "RequestedProvisionedConcurrentExecutions": 5,
    "AvailableProvisionedConcurrentExecutions": 5,
    "AllocatedProvisionedConcurrentExecutions": 5,
    "Status": "READY"
}
```

**Why:** Verify provisioned concurrency status.

**When to Use:** Monitoring, troubleshooting cold starts.

---

### 6. Delete Provisioned Concurrency

```bash
aws lambda delete-provisioned-concurrency-config --function-name my-first-function --qualifier production
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove provisioned concurrency to save costs.

**When to Use:** Cost optimization, testing completed.

---

## Monitoring & Logging

### 1. View CloudWatch Logs

```bash
# List log groups
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/my-first-function

# Get log streams
aws logs describe-log-streams \
    --log-group-name /aws/lambda/my-first-function \
    --order-by LastEventTime \
    --descending \
    --max-items 5

# Get log events
aws logs get-log-events \
    --log-group-name /aws/lambda/my-first-function \
    --log-stream-name '2025/11/17/[$LATEST]abc123'
```

**Expected Output:**
```json
{
    "events": [
        {
            "timestamp": 1700227200000,
            "message": "START RequestId: abc-123 Version: $LATEST",
            "ingestionTime": 1700227201000
        },
        {
            "timestamp": 1700227200100,
            "message": "Event received: {...}",
            "ingestionTime": 1700227201000
        }
    ]
}
```

**Why:** Debug function execution, monitor behavior.

**When to Use:** Troubleshooting errors, performance analysis.

---

### 2. Get Function Metrics

```bash
aws cloudwatch get-metric-statistics \
    --namespace AWS/Lambda \
    --metric-name Invocations \
    --dimensions Name=FunctionName,Value=my-first-function \
    --start-time 2025-11-17T00:00:00Z \
    --end-time 2025-11-17T23:59:59Z \
    --period 3600 \
    --statistics Sum
```

**Expected Output:**
```json
{
    "Datapoints": [
        {
            "Timestamp": "2025-11-17T10:00:00+00:00",
            "Sum": 150.0,
            "Unit": "Count"
        }
    ]
}
```

**Why:** Monitor function usage, errors, performance.

**When to Use:** Performance analysis, cost tracking, alerting.

**Key Metrics:**
- **Invocations**: Number of times function is invoked
- **Errors**: Number of failed invocations
- **Duration**: Execution time
- **Throttles**: Throttled invocations
- **ConcurrentExecutions**: Concurrent invocations
- **DeadLetterErrors**: Failed async event delivery

---

### 3. Enable X-Ray Tracing

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --tracing-config Mode=Active
```

**Expected Output:**
```json
{
    "TracingConfig": {
        "Mode": "Active"
    }
}
```

**Why:** Detailed performance tracing and service map visualization.

**When to Use:** Debugging complex workflows, performance optimization, understanding service dependencies.

**Python Code:**
```python
from aws_xray_sdk.core import xray_recorder

@xray_recorder.capture('process_data')
def process_data(data):
    # Your code
    pass
```

---

### 4. Add Dead Letter Queue (DLQ)

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --dead-letter-config TargetArn=arn:aws:sqs:us-east-1:123456789012:my-dlq
```

**Expected Output:**
```json
{
    "DeadLetterConfig": {
        "TargetArn": "arn:aws:sqs:us-east-1:123456789012:my-dlq"
    }
}
```

**Why:** Capture failed async invocations for analysis/retry.

**When to Use:** Asynchronous functions, ensuring no data loss.

**Note:** Only for asynchronous invocations (S3, SNS, EventBridge).

---

### 5. Configure Async Invoke Config

```bash
aws lambda put-function-event-invoke-config \
    --function-name my-first-function \
    --maximum-retry-attempts 1 \
    --maximum-event-age-in-seconds 3600 \
    --destination-config '{
        "OnSuccess": {
            "Destination": "arn:aws:sqs:us-east-1:123456789012:success-queue"
        },
        "OnFailure": {
            "Destination": "arn:aws:sqs:us-east-1:123456789012:failure-queue"
        }
    }'
```

**Expected Output:**
```json
{
    "LastModified": "2025-11-17T16:00:00+00:00",
    "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-first-function",
    "MaximumRetryAttempts": 1,
    "MaximumEventAgeInSeconds": 3600,
    "DestinationConfig": {
        "OnSuccess": {
            "Destination": "arn:aws:sqs:us-east-1:123456789012:success-queue"
        },
        "OnFailure": {
            "Destination": "arn:aws:sqs:us-east-1:123456789012:failure-queue"
        }
    }
}
```

**Why:** Control retry behavior and route execution results.

**When to Use:** Fine-grained async control, result processing.

---

## Testing & Invocation

### 1. Invoke Function (Synchronous)

```bash
aws lambda invoke \
    --function-name my-first-function \
    --payload '{"key": "value"}' \
    response.json
```

**Expected Output:**
```json
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
```

**response.json:**
```json
{
    "statusCode": 200,
    "body": "Hello from Lambda!"
}
```

**Why:** Test function, manual invocation, debugging.

**When to Use:** Development, testing, troubleshooting.

---

### 2. Invoke with Payload File

```bash
# Create payload
echo '{"name": "John", "action": "process"}' > event.json

# Invoke
aws lambda invoke --function-name my-first-function --payload file://event.json response.json
```

**Why:** Test with complex/large payloads.

**When to Use:** Integration testing, realistic scenarios.

---

### 3. Asynchronous Invocation

```bash
aws lambda invoke \
    --function-name my-first-function \
    --invocation-type Event \
    --payload '{"key": "value"}' \
    response.json
```

**Expected Output:**
```json
{
    "StatusCode": 202
}
```

**Why:** Fire-and-forget invocation. Lambda queues event and returns immediately.

**When to Use:** Background processing, non-critical operations.

**Note:** Status 202 means accepted, not executed yet.

---

### 4. Dry Run Invocation

```bash
aws lambda invoke \
    --function-name my-first-function \
    --invocation-type DryRun \
    --payload '{"key": "value"}' \
    response.json
```

**Expected Output:**
```json
{
    "StatusCode": 204
}
```

**Why:** Verify permissions without executing function.

**When to Use:** Testing IAM permissions, pre-deployment checks.

---

### 5. Invoke Specific Version

```bash
aws lambda invoke --function-name my-first-function:1 --payload '{"key": "value"}' response.json
```

**Why:** Test specific version without affecting $LATEST or aliases.

**When to Use:** Testing before promoting to production.

---

### 6. Invoke Alias

```bash
aws lambda invoke --function-name my-first-function:production --payload '{"key": "value"}' response.json
```

**Why:** Test production configuration, simulate real invocations.

**When to Use:** Integration testing, troubleshooting production.

---

### 7. Get Invocation Logs in Response

```bash
aws lambda invoke \
    --function-name my-first-function \
    --log-type Tail \
    --payload '{"key": "value"}' \
    response.json
```

**Expected Output:**
```json
{
    "StatusCode": 200,
    "LogResult": "U1RBUlQgUmVxdWVzdElkOiBhYmMtMTIzCkV2ZW50IHJlY2VpdmVkOiB7ImtleSI6InZhbHVlIn0KRU5EIFJlcXVlc3RJZD==",
    "ExecutedVersion": "$LATEST"
}
```

**Decode logs:**
```bash
echo "U1RBUlQgUmVxdWVzdElkOiBhYmMtMTIzCkV2ZW50IHJlY2VpdmVkOiB7ImtleSI6InZhbHVlIn0KRU5EIFJlcXVlc3RJZD==" | base64 --decode
```

**Why:** Get last 4KB of logs immediately with invocation.

**When to Use:** Quick debugging, CLI testing.

---

## VPC Configuration

### 1. Configure VPC Access

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --vpc-config SubnetIds=subnet-12345,subnet-67890,SecurityGroupIds=sg-abcdef
```

**Expected Output:**
```json
{
    "VpcConfig": {
        "SubnetIds": ["subnet-12345", "subnet-67890"],
        "SecurityGroupIds": ["sg-abcdef"],
        "VpcId": "vpc-123456"
    }
}
```

**Why:** Access resources in VPC (RDS, ElastiCache, internal APIs).

**When to Use:** Accessing private resources, security requirements.

**Important:**
- Use private subnets with NAT Gateway for internet access
- Need VPC execution role permissions
- Cold starts may be longer
- Consider VPC endpoints for AWS services

---

### 2. Remove VPC Configuration

```bash
aws lambda update-function-configuration \
    --function-name my-first-function \
    --vpc-config SubnetIds=[],SecurityGroupIds=[]
```

**Expected Output:**
```json
{
    "VpcConfig": {
        "SubnetIds": [],
        "SecurityGroupIds": []
    }
}
```

**Why:** Remove VPC access (faster cold starts, simpler configuration).

**When to Use:** Function no longer needs VPC resources.

---

## Best Practices

### Development
1. **Use environment variables** for configuration
2. **Minimize package size** - Use layers for dependencies
3. **Separate handler from business logic** - Makes testing easier
4. **Initialize outside handler** - Reuse connections across invocations
5. **Use CloudWatch Logs** - Structured logging with JSON
6. **Enable X-Ray** - Distributed tracing for debugging
7. **Set appropriate timeout** - Default 3s often too short
8. **Idempotent functions** - Safe to retry

### Example Function Structure
```python
import boto3
import os
import json

# Initialize outside handler (reused across invocations)
s3_client = boto3.client('s3')
db_host = os.environ['DB_HOST']

def lambda_handler(event, context):
    """Main handler - minimal logic"""
    try:
        result = process_event(event)
        return {
            'statusCode': 200,
            'body': json.dumps(result)
        }
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }

def process_event(event):
    """Business logic - easily testable"""
    # Your logic here
    return {'message': 'Success'}
```

### Performance
1. **Allocate adequate memory** - More memory = more CPU
2. **Use provisioned concurrency** for latency-sensitive apps
3. **Optimize cold starts**:
   - Minimize dependencies
   - Use smaller runtimes (Python/Node.js faster than Java)
   - Consider container images for large dependencies
4. **Connection pooling** - Reuse DB connections
5. **Async where possible** - Don't block on I/O
6. **Monitor duration** - Optimize expensive operations

### Security
1. **Least privilege IAM roles** - Only required permissions
2. **Encrypt environment variables** with KMS for secrets
3. **Use Secrets Manager/Parameter Store** for credentials
4. **VPC configuration** for private resources
5. **Enable X-Ray** for security monitoring
6. **Validate input** - Never trust event data
7. **Rotate credentials regularly**
8. **Use layers for sensitive code** - Separate from application

### Cost Optimization
1. **Right-size memory** - Balance cost vs. performance
2. **Use ARM (Graviton2)** - 20% better price/performance
3. **Minimize execution time** - You pay per 1ms
4. **Use lifecycle policies** - Clean old versions
5. **Reserved concurrency** - Prevent runaway costs
6. **Monitor invocations** - Set billing alerts
7. **Avoid provisioned concurrency** unless needed

### Reliability
1. **Set dead letter queues** - Capture failed events
2. **Configure retries** - Appropriate for use case
3. **Use versions and aliases** - Safe deployments
4. **Implement circuit breakers** - Prevent cascading failures
5. **Monitor error rates** - CloudWatch alarms
6. **Test failure scenarios** - Chaos engineering
7. **Multi-region deployments** - High availability

### CI/CD Integration
```bash
# Build
zip -r function.zip .

# Deploy
aws lambda update-function-code \
    --function-name my-function \
    --zip-file fileb://function.zip

# Publish version
VERSION=$(aws lambda publish-version \
    --function-name my-function \
    --query 'Version' \
    --output text)

# Update alias (blue/green)
aws lambda update-alias \
    --function-name my-function \
    --name production \
    --function-version $VERSION

# Wait for deployment
aws lambda wait function-updated --function-name my-function
```

---

## Common Use Cases

### 1. API Backend (with API Gateway)
```python
def lambda_handler(event, context):
    # Parse request
    http_method = event['httpMethod']
    path = event['path']
    body = json.loads(event.get('body', '{}'))

    if http_method == 'GET':
        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({'message': 'GET request'})
        }
    elif http_method == 'POST':
        # Process POST data
        return {
            'statusCode': 201,
            'body': json.dumps({'message': 'Created'})
        }
```

### 2. S3 Image Processing
```python
import boto3
from PIL import Image
import io

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get S3 event details
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    # Download image
    response = s3.get_object(Bucket=bucket, Key=key)
    image = Image.open(response['Body'])

    # Resize
    image.thumbnail((200, 200))

    # Upload thumbnail
    buffer = io.BytesIO()
    image.save(buffer, 'JPEG')
    buffer.seek(0)

    s3.put_object(
        Bucket=bucket,
        Key=f'thumbnails/{key}',
        Body=buffer,
        ContentType='image/jpeg'
    )
```

### 3. DynamoDB Stream Processing
```python
def lambda_handler(event, context):
    for record in event['Records']:
        if record['eventName'] == 'INSERT':
            new_image = record['dynamodb']['NewImage']
            # Process new record
        elif record['eventName'] == 'MODIFY':
            old_image = record['dynamodb']['OldImage']
            new_image = record['dynamodb']['NewImage']
            # Process update
        elif record['eventName'] == 'REMOVE':
            old_image = record['dynamodb']['OldImage']
            # Process deletion
```

### 4. Scheduled Task (EventBridge)
```python
def lambda_handler(event, context):
    # Cleanup old records
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('MyTable')

    # Delete old items
    cutoff_time = int(time.time()) - (30 * 86400)  # 30 days ago

    # Scan and delete (use carefully, expensive for large tables)
    response = table.scan(
        FilterExpression='created_at < :cutoff',
        ExpressionAttributeValues={':cutoff': cutoff_time}
    )

    for item in response['Items']:
        table.delete_item(Key={'id': item['id']})
```

### 5. SQS Message Processing
```python
def lambda_handler(event, context):
    for record in event['Records']:
        # Parse message
        body = json.loads(record['body'])

        try:
            # Process message
            result = process_message(body)
            print(f"Processed: {result}")
        except Exception as e:
            # Message will return to queue (or go to DLQ after max retries)
            print(f"Error processing message: {e}")
            raise  # Raise to indicate failure
```

---

## Troubleshooting

### Access Denied Errors
- Check execution role permissions
- Verify resource policies (S3, SQS, etc.)
- Check VPC endpoint policies if in VPC

### Timeout Issues
- Increase function timeout
- Optimize code performance
- Check for slow API calls
- Use async operations

### Cold Start Problems
- Use provisioned concurrency
- Minimize package size
- Optimize initialization code
- Consider keeping functions "warm" with EventBridge

### Memory Issues
- Increase memory allocation
- Monitor actual usage in CloudWatch
- Check for memory leaks
- Optimize data structures

### VPC Connectivity
- Verify subnet configuration (private with NAT)
- Check security group rules
- Ensure VPC endpoints for AWS services
- Verify route tables

---

## Quick Reference

```bash
# Create function
aws lambda create-function --function-name NAME --runtime python3.11 --role ROLE_ARN --handler lambda_function.lambda_handler --zip-file fileb://function.zip

# Update code
aws lambda update-function-code --function-name NAME --zip-file fileb://function.zip

# Update config
aws lambda update-function-configuration --function-name NAME --timeout 60 --memory-size 512

# Invoke
aws lambda invoke --function-name NAME --payload '{}' response.json

# View logs
aws logs tail /aws/lambda/NAME --follow

# Publish version
aws lambda publish-version --function-name NAME

# Create alias
aws lambda create-alias --function-name NAME --name production --function-version 1

# List functions
aws lambda list-functions

# Delete function
aws lambda delete-function --function-name NAME
```

---

## Summary

Lambda enables serverless computing, eliminating server management and automatically scaling your applications. Master these commands to:
- Build serverless APIs
- Process data in real-time
- Automate workflows
- Integrate AWS services
- Scale automatically

Practice creating functions, configuring triggers, and monitoring execution. Lambda's event-driven model combined with pay-per-use pricing makes it ideal for modern cloud applications.

For LocalStack testing, add `--endpoint-url=http://localhost:4566` to all commands.

Happy serverless computing!
