AWS Services Learning Guide with LocalStack
A Comprehensive Hands-On TutorialTable of Contents

Prerequisites & Setup
Week 1-2: S3 (Simple Storage Service)
Week 2: SQS (Simple Queue Service)
Week 3: Lambda
Week 4: API Gateway
Week 5: Cognito
Week 6: SES (Simple Email Service)
Integration Projects
Terraform Migration Guide
Prerequisites & SetupEnvironment Variables
Set these up first to make your life easier:bash# Add to your ~/.bashrc or ~/.zshrc
export AWS_ENDPOINT=http://localhost:4566
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1

# Reload your shell or run:
source ~/.bashrcWhy? These environment variables tell the AWS CLI to use LocalStack instead of real AWS. The credentials are dummy values that LocalStack accepts.Verify LocalStack is Runningbashcurl http://localhost:4566/_localstack/health | jqExpected Output:
json{
  "services": {
    "s3": "running",
    "lambda": "running",
    "apigateway": "running",
    "sqs": "running",
    "ses": "running",
    "cognito-idp": "running"
  }
}What this means: All the services you need are available and ready to use.Week 1-2: S3 (Simple Storage Service)What is S3?
S3 is object storage - think of it as a filesystem in the cloud where you store files (objects) in containers (buckets). It's the foundation of AWS and used by almost every application.1. Creating Your First Bucketbashaws --endpoint-url=$AWS_ENDPOINT s3 mb s3://my-learning-bucketExpected Output:
make_bucket: my-learning-bucketWhat happened? You created a bucket named "my-learning-bucket". Bucket names must be globally unique (in real AWS) and follow DNS naming conventions.Why "mb"? It stands for "make bucket" - similar to the Unix mkdir command.2. Listing Bucketsbashaws --endpoint-url=$AWS_ENDPOINT s3 lsExpected Output:
2025-11-17 10:30:45 my-learning-bucketWhat this shows: The timestamp when the bucket was created and the bucket name.3. Uploading Files (Objects)First, create a test file:
bashecho "This is my first S3 object" > test-file.txtUpload it:
bashaws --endpoint-url=$AWS_ENDPOINT s3 cp test-file.txt s3://my-learning-bucket/Expected Output:
upload: ./test-file.txt to s3://my-learning-bucket/test-file.txtWhat happened? The file was copied from your local filesystem to the S3 bucket.4. Listing Objects in a Bucketbashaws --endpoint-url=$AWS_ENDPOINT s3 ls s3://my-learning-bucket/Expected Output:
2025-11-17 10:35:22         27 test-file.txtWhat this shows: The date, file size (27 bytes), and object name.5. Downloading Filesbashaws --endpoint-url=$AWS_ENDPOINT s3 cp s3://my-learning-bucket/test-file.txt downloaded-file.txtExpected Output:
download: s3://my-learning-bucket/test-file.txt to ./downloaded-file.txtVerify:
bashcat downloaded-file.txtOutput: This is my first S3 object6. Creating Folder StructureS3 doesn't have real folders, but you can simulate them with prefixes:bashaws --endpoint-url=$AWS_ENDPOINT s3 cp test-file.txt s3://my-learning-bucket/documents/reports/test-file.txtList with prefix:
bashaws --endpoint-url=$AWS_ENDPOINT s3 ls s3://my-learning-bucket/documents/reports/Expected Output:
2025-11-17 10:40:15         27 test-file.txtWhy this matters: Most applications organize S3 objects using prefixes to simulate a directory structure.7. Uploading Multiple Filesbash# Create multiple files
echo "File 1" > file1.txt
echo "File 2" > file2.txt
echo "File 3" > file3.txt

# Upload all at once
aws --endpoint-url=$AWS_ENDPOINT s3 cp . s3://my-learning-bucket/batch/ --recursive --exclude "*" --include "file*.txt"Expected Output:
upload: ./file1.txt to s3://my-learning-bucket/batch/file1.txt
upload: ./file2.txt to s3://my-learning-bucket/batch/file2.txt
upload: ./file3.txt to s3://my-learning-bucket/batch/file3.txtWhy --recursive? It tells AWS CLI to process all files in the directory.8. Deleting ObjectsSingle file:
bashaws --endpoint-url=$AWS_ENDPOINT s3 rm s3://my-learning-bucket/test-file.txtExpected Output:
delete: s3://my-learning-bucket/test-file.txtDelete all objects with a prefix:
bashaws --endpoint-url=$AWS_ENDPOINT s3 rm s3://my-learning-bucket/batch/ --recursive9. Bucket VersioningEnable versioning:
bashaws --endpoint-url=$AWS_ENDPOINT s3api put-bucket-versioning \
  --bucket my-learning-bucket \
  --versioning-configuration Status=EnabledNo output means success.Check versioning status:
bashaws --endpoint-url=$AWS_ENDPOINT s3api get-bucket-versioning --bucket my-learning-bucketExpected Output:
json{
    "Status": "Enabled"
}Why versioning? It keeps all versions of an object, protecting against accidental deletion or overwrites.Upload the same file twice:
bashecho "Version 1" > versioned-file.txt
aws --endpoint-url=$AWS_ENDPOINT s3 cp versioned-file.txt s3://my-learning-bucket/

echo "Version 2" > versioned-file.txt
aws --endpoint-url=$AWS_ENDPOINT s3 cp versioned-file.txt s3://my-learning-bucket/List all versions:
bashaws --endpoint-url=$AWS_ENDPOINT s3api list-object-versions --bucket my-learning-bucketExpected Output:
json{
    "Versions": [
        {
            "Key": "versioned-file.txt",
            "VersionId": "ABC123",
            "IsLatest": true,
            "Size": 10
        },
        {
            "Key": "versioned-file.txt",
            "VersionId": "XYZ789",
            "IsLatest": false,
            "Size": 10
        }
    ]
}10. Pre-signed URLsGenerate a temporary URL to share a file:
bashaws --endpoint-url=$AWS_ENDPOINT s3 presign s3://my-learning-bucket/test-file.txt --expires-in 3600Expected Output:
http://localhost:4566/my-learning-bucket/test-file.txt?X-Amz-Algorithm=...&X-Amz-Expires=3600...Why pre-signed URLs? They allow temporary access to private objects without making them public. Expires in 3600 seconds (1 hour).11. Deleting a BucketFirst, empty it:
bashaws --endpoint-url=$AWS_ENDPOINT s3 rm s3://my-learning-bucket/ --recursiveThen delete:
bashaws --endpoint-url=$AWS_ENDPOINT s3 rb s3://my-learning-bucketExpected Output:
remove_bucket: my-learning-bucketWhy two steps? AWS won't delete a bucket that contains objects - it's a safety feature.S3 Key Concepts Summary

Bucket: Container for objects (max 100 per account by default)
Object: File stored in S3 (up to 5TB per object)
Key: Object name/path (e.g., documents/report.pdf)
Prefix: Used to organize objects like folders
Versioning: Keeps multiple versions of objects
Pre-signed URLs: Temporary access to private objects
Week 2: SQS (Simple Queue Service)What is SQS?
SQS is a message queue service - think of it as a waiting line for messages between services. Producer services send messages, consumer services receive and process them. This decouples your architecture.1. Creating a Standard Queuebashaws --endpoint-url=$AWS_ENDPOINT sqs create-queue --queue-name my-first-queueExpected Output:
json{
    "QueueUrl": "http://localhost:4566/000000000000/my-first-queue"
}What's a QueueUrl? It's the endpoint you'll use to interact with this specific queue. Save this!Store it in a variable:
bashQUEUE_URL=$(aws --endpoint-url=$AWS_ENDPOINT sqs create-queue --queue-name my-first-queue --query 'QueueUrl' --output text)
echo $QUEUE_URL2. Listing Queuesbashaws --endpoint-url=$AWS_ENDPOINT sqs list-queuesExpected Output:
json{
    "QueueUrls": [
        "http://localhost:4566/000000000000/my-first-queue"
    ]
}3. Sending MessagesSend a simple message:
bashaws --endpoint-url=$AWS_ENDPOINT sqs send-message \
  --queue-url $QUEUE_URL \
  --message-body "Hello from SQS!"Expected Output:
json{
    "MessageId": "12345678-1234-1234-1234-123456789012",
    "MD5OfMessageBody": "5d41402abc4b2a76b9719d911017c592"
}What's MD5? A checksum to verify message integrity.Send a JSON message:
bashaws --endpoint-url=$AWS_ENDPOINT sqs send-message \
  --queue-url $QUEUE_URL \
  --message-body '{"orderId": "12345", "customer": "John Doe", "amount": 99.99}'4. Receiving Messagesbashaws --endpoint-url=$AWS_ENDPOINT sqs receive-message --queue-url $QUEUE_URLExpected Output:
json{
    "Messages": [
        {
            "MessageId": "12345678-1234-1234-1234-123456789012",
            "ReceiptHandle": "AQEB...",
            "MD5OfBody": "5d41402abc4b2a76b9719d911017c592",
            "Body": "Hello from SQS!"
        }
    ]
}What's ReceiptHandle? A temporary token you need to delete the message after processing. Think of it as a "claim ticket."5. Deleting MessagesAfter processing, always delete the message:
bashRECEIPT_HANDLE=$(aws --endpoint-url=$AWS_ENDPOINT sqs receive-message --queue-url $QUEUE_URL --query 'Messages[0].ReceiptHandle' --output text)

aws --endpoint-url=$AWS_ENDPOINT sqs delete-message \
  --queue-url $QUEUE_URL \
  --receipt-handle $RECEIPT_HANDLENo output means success.Why delete? If you don't delete it, the message becomes visible again after the visibility timeout expires.6. Visibility TimeoutWhen you receive a message, it becomes invisible to other consumers for a period (default 30 seconds). This prevents multiple consumers from processing the same message.Check queue attributes:
bashaws --endpoint-url=$AWS_ENDPOINT sqs get-queue-attributes \
  --queue-url $QUEUE_URL \
  --attribute-names AllExpected Output:
json{
    "Attributes": {
        "VisibilityTimeout": "30",
        "MaximumMessageSize": "262144",
        "MessageRetentionPeriod": "345600",
        "ApproximateNumberOfMessages": "0"
    }
}Change visibility timeout:
bashaws --endpoint-url=$AWS_ENDPOINT sqs set-queue-attributes \
  --queue-url $QUEUE_URL \
  --attributes VisibilityTimeout=60Why change it? If your processing takes longer than 30 seconds, increase it to prevent duplicate processing.7. Message AttributesSend a message with metadata:
bashaws --endpoint-url=$AWS_ENDPOINT sqs send-message \
  --queue-url $QUEUE_URL \
  --message-body "Order notification" \
  --message-attributes '{"Priority":{"DataType":"String","StringValue":"High"},"OrderType":{"DataType":"String","StringValue":"Express"}}'Expected Output:
json{
    "MessageId": "abc-123",
    "MD5OfMessageBody": "...",
    "MD5OfMessageAttributes": "..."
}Receive with attributes:
bashaws --endpoint-url=$AWS_ENDPOINT sqs receive-message \
  --queue-url $QUEUE_URL \
  --attribute-names All \
  --message-attribute-names AllWhy attributes? They let you add metadata without modifying the message body - useful for filtering and routing.8. Creating a FIFO QueueFIFO = First-In-First-Out (guaranteed ordering)bashaws --endpoint-url=$AWS_ENDPOINT sqs create-queue \
  --queue-name my-fifo-queue.fifo \
  --attributes FifoQueue=true,ContentBasedDeduplication=trueExpected Output:
json{
    "QueueUrl": "http://localhost:4566/000000000000/my-fifo-queue.fifo"
}Why .fifo suffix? FIFO queues must end with .fifo.Send a message to FIFO queue:
bashFIFO_QUEUE_URL=$(aws --endpoint-url=$AWS_ENDPOINT sqs get-queue-url --queue-name my-fifo-queue.fifo --query 'QueueUrl' --output text)

aws --endpoint-url=$AWS_ENDPOINT sqs send-message \
  --queue-url $FIFO_QUEUE_URL \
  --message-body "First message" \
  --message-group-id "group1"What's MessageGroupId? Messages with the same group ID are processed in order. Different groups can be processed in parallel.9. Dead Letter Queue (DLQ)A DLQ receives messages that fail processing multiple times.Create a DLQ:
bashaws --endpoint-url=$AWS_ENDPOINT sqs create-queue --queue-name my-dlq

DLQ_URL=$(aws --endpoint-url=$AWS_ENDPOINT sqs get-queue-url --queue-name my-dlq --query 'QueueUrl' --output text)

# Get DLQ ARN
DLQ_ARN=$(aws --endpoint-url=$AWS_ENDPOINT sqs get-queue-attributes \
  --queue-url $DLQ_URL \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' \
  --output text)

echo $DLQ_ARNConfigure main queue to use DLQ:
bashaws --endpoint-url=$AWS_ENDPOINT sqs set-queue-attributes \
  --queue-url $QUEUE_URL \
  --attributes '{
    "RedrivePolicy": "{\"deadLetterTargetArn\":\"'$DLQ_ARN'\",\"maxReceiveCount\":\"3\"}"
  }'What this means: After a message is received 3 times without being deleted, it moves to the DLQ.Why DLQs? They prevent problematic messages from blocking your queue and let you investigate failures.10. Purging a QueueDelete all messages at once:
bashaws --endpoint-url=$AWS_ENDPOINT sqs purge-queue --queue-url $QUEUE_URLNo output means success.Warning: This is irreversible! Use with caution.11. Deleting a Queuebashaws --endpoint-url=$AWS_ENDPOINT sqs delete-queue --queue-url $QUEUE_URLNote: Deleted queues may take up to 60 seconds to fully disappear.SQS Key Concepts Summary

Standard Queue: At-least-once delivery, best-effort ordering
FIFO Queue: Exactly-once delivery, guaranteed ordering
Message: Data sent between services (max 256KB)
Visibility Timeout: How long a message is invisible after being received
Receipt Handle: Temporary token to delete/modify a message
Dead Letter Queue: Captures messages that fail repeatedly
Message Group ID: For FIFO queues, messages in same group are ordered
Week 3: LambdaWhat is Lambda?
Lambda is serverless compute - you write code, AWS runs it. No servers to manage. You pay only for execution time. Perfect for event-driven architectures.1. Creating Your First Lambda FunctionCreate a simple Python function:
bashcat > lambda_function.py << 'EOF'
def lambda_handler(event, context):
    print(f"Received event: {event}")
    
    return {
        'statusCode': 200,
        'body': f'Hello from Lambda! You sent: {event}'
    }
EOFWhat's lambda_handler? The entry point - Lambda calls this function when triggered.

event: Input data (JSON)
context: Runtime information (request ID, remaining time, etc.)
Zip it:
bashzip function.zip lambda_function.pyCreate the Lambda function:
bashaws --endpoint-url=$AWS_ENDPOINT lambda create-function \
  --function-name my-first-lambda \
  --runtime python3.11 \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zipExpected Output:
json{
    "FunctionName": "my-first-lambda",
    "FunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:my-first-lambda",
    "Runtime": "python3.11",
    "Role": "arn:aws:iam::000000000000:role/lambda-role",
    "Handler": "lambda_function.lambda_handler",
    "CodeSize": 324,
    "State": "Active"
}What's the handler? Format is filename.function_name. Lambda looks for lambda_handler function in lambda_function.py.2. Invoking LambdaSimple invocation:
bashaws --endpoint-url=$AWS_ENDPOINT lambda invoke \
  --function-name my-first-lambda \
  --payload '{"name": "Sijan", "action": "learning"}' \
  response.jsonExpected Output:
json{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}Check the response:
bashcat response.jsonOutput:
json{
    "statusCode": 200,
    "body": "Hello from Lambda! You sent: {'name': 'Sijan', 'action': 'learning'}"
}3. Viewing Lambda Logsbashaws --endpoint-url=$AWS_ENDPOINT logs tail /aws/lambda/my-first-lambda --followWhat you'll see:
START RequestId: abc-123
Received event: {'name': 'Sijan', 'action': 'learning'}
END RequestId: abc-123
REPORT Duration: 1.23 ms	Billed Duration: 2 ms	Memory Size: 128 MB	Max Memory Used: 35 MBWhy this matters: Logs are critical for debugging. You see execution time, memory usage, and your print statements.4. Updating Lambda CodeModify the function:
bashcat > lambda_function.py << 'EOF'
import json

def lambda_handler(event, context):
    name = event.get('name', 'stranger')
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f'Hello {name}!',
            'event_received': event
        })
    }
EOFZip and update:
bashzip function.zip lambda_function.py

aws --endpoint-url=$AWS_ENDPOINT lambda update-function-code \
  --function-name my-first-lambda \
  --zip-file fileb://function.zipExpected Output:
json{
    "FunctionName": "my-first-lambda",
    "LastModified": "2025-11-17T11:00:00.000+0000",
    "CodeSize": 389
}Test it:
bashaws --endpoint-url=$AWS_ENDPOINT lambda invoke \
  --function-name my-first-lambda \
  --payload '{"name": "Sijan"}' \
  response.json && cat response.json5. Environment VariablesSet environment variables:
bashaws --endpoint-url=$AWS_ENDPOINT lambda update-function-configuration \
  --function-name my-first-lambda \
  --environment "Variables={ENV=development,DB_HOST=localhost,API_KEY=secret123}"Expected Output:
json{
    "FunctionName": "my-first-lambda",
    "Environment": {
        "Variables": {
            "ENV": "development",
            "DB_HOST": "localhost",
            "API_KEY": "secret123"
        }
    }
}Use them in your function:
bashcat > lambda_function.py << 'EOF'
import os
import json

def lambda_handler(event, context):
    env = os.environ.get('ENV', 'unknown')
    db_host = os.environ.get('DB_HOST', 'not-set')
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'environment': env,
            'database': db_host,
            'message': 'Environment variables loaded!'
        })
    }
EOFWhy environment variables? Keep secrets and configuration separate from code. Change them without redeploying code.6. Lambda with SQS TriggerCreate an SQS queue:
bashaws --endpoint-url=$AWS_ENDPOINT sqs create-queue --queue-name lambda-trigger-queue

QUEUE_URL=$(aws --endpoint-url=$AWS_ENDPOINT sqs get-queue-url --queue-name lambda-trigger-queue --query 'QueueUrl' --output text)

QUEUE_ARN=$(aws --endpoint-url=$AWS_ENDPOINT sqs get-queue-attributes \
  --queue-url $QUEUE_URL \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' \
  --output text)Create Lambda to process SQS messages:
bashcat > sqs_processor.py << 'EOF'
import json

def lambda_handler(event, context):
    # Lambda receives batches of messages
    for record in event['Records']:
        message_body = record['body']
        print(f"Processing message: {message_body}")
        
        # Your processing logic here
        data = json.loads(message_body)
        print(f"Order ID: {data.get('orderId')}")
    
    return {
        'statusCode': 200,
        'body': 'Processed all messages'
    }
EOF

zip sqs_processor.zip sqs_processor.py

aws --endpoint-url=$AWS_ENDPOINT lambda create-function \
  --function-name sqs-processor \
  --runtime python3.11 \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --handler sqs_processor.lambda_handler \
  --zip-file fileb://sqs_processor.zipCreate event source mapping:
bashaws --endpoint-url=$AWS_ENDPOINT lambda create-event-source-mapping \
  --function-name sqs-processor \
  --event-source-arn $QUEUE_ARN \
  --batch-size 10Expected Output:
json{
    "UUID": "12345-abcde",
    "BatchSize": 10,
    "EventSourceArn": "arn:aws:sqs:us-east-1:000000000000:lambda-trigger-queue",
    "FunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:sqs-processor",
    "State": "Enabled"
}What happened? Lambda now automatically polls the SQS queue and processes messages in batches of up to 10.Test it:
bash# Send a message
aws --endpoint-url=$AWS_ENDPOINT sqs send-message \
  --queue-url $QUEUE_URL \
  --message-body '{"orderId": "12345", "product": "Laptop"}'

# Check Lambda logs (wait a few seconds)
aws --endpoint-url=$AWS_ENDPOINT logs tail /aws/lambda/sqs-processorWhy this pattern? Completely decoupled architecture. Producers send to queue, Lambda automatically processes. No polling code needed.7. Lambda Timeout and MemoryView current configuration:
bashaws --endpoint-url=$AWS_ENDPOINT lambda get-function-configuration \
  --function-name my-first-lambdaUpdate timeout and memory:
bashaws --endpoint-url=$AWS_ENDPOINT lambda update-function-configuration \
  --function-name my-first-lambda \
  --timeout 30 \
  --memory-size 256Expected Output:
json{
    "FunctionName": "my-first-lambda",
    "Timeout": 30,
    "MemorySize": 256
}What changed?

Timeout: Max execution time (default 3 sec, max 900 sec / 15 min)
Memory: 256 MB (affects CPU allocation too - more memory = more CPU)
Why adjust? Match your function's needs. Complex processing needs more time/memory. Quick APIs need less.8. Lambda LayersLayers let you share code/dependencies across functions. Create a layer with a common library:bashmkdir -p python/lib/python3.11/site-packages
pip install requests -t python/lib/python3.11/site-packages

zip -r layer.zip pythonCreate the layer:
bashaws --endpoint-url=$AWS_ENDPOINT lambda publish-layer-version \
  --layer-name common-dependencies \
  --zip-file fileb://layer.zip \
  --compatible-runtimes python3.11Expected Output:
json{
    "LayerArn": "arn:aws:lambda:us-east-1:000000000000:layer:common-dependencies",
    "LayerVersionArn": "arn:aws:lambda:us-east-1:000000000000:layer:common-dependencies:1",
    "Version": 1
}Attach layer to function:
bashLAYER_ARN=$(aws --endpoint-url=$AWS_ENDPOINT lambda list-layer-versions \
  --layer-name common-dependencies \
  --query 'LayerVersions[0].LayerVersionArn' \
  --output text)

aws --endpoint-url=$AWS_ENDPOINT lambda update-function-configuration \
  --function-name my-first-lambda \
  --layers $LAYER_ARNNow use requests in your function:
bashcat > lambda_function.py << 'EOF'
import requests

def lambda_handler(event, context):
    # requests is available from the layer
    return {
        'statusCode': 200,
        'body': 'Layer attached successfully!'
    }
EOFWhy layers? Don't package large dependencies in every function. Share them via layers.9. Asynchronous InvocationInvoke without waiting for response:
bashaws --endpoint-url=$AWS_ENDPOINT lambda invoke \
  --function-name my-first-lambda \
  --invocation-type Event \
  --payload '{"async": true}' \
  response.jsonExpected Output:
json{
    "StatusCode": 202
}202 means: Request accepted, Lambda will execute it asynchronously.Check response file:
bashcat response.jsonOutput: Empty (Lambda hasn't finished yet)When to use async? When you don't need immediate results - fire and forget.10. List All Functionsbashaws --endpoint-url=$AWS_ENDPOINT lambda list-functionsExpected Output:
json{
    "Functions": [
        {
            "FunctionName": "my-first-lambda",
            "Runtime": "python3.11",
            "Handler": "lambda_function.lambda_handler",
            "CodeSize": 389,
            "Timeout": 30,
            "MemorySize": 256
        },
        {
            "FunctionName": "sqs-processor",
            "Runtime": "python3.11",
            "Handler": "sqs_processor.lambda_handler"
        }
    ]
}11. Deleting Lambda Functionsbashaws --endpoint-url=$AWS_ENDPOINT lambda delete-function --function-name my-first-lambdaNo output means success.Lambda Key Concepts Summary

Function: Your code + configuration
Handler: Entry point function Lambda calls
Event: Input data (JSON)
Context: Runtime information
Environment Variables: Configuration separate from code
Layers: Reusable code/dependencies
Timeout: Max execution time (3-900 seconds)
Memory: 128 MB to 10 GB (affects CPU too)
Event Source Mapping: Automatic trigger from SQS, Kinesis, DynamoDB
Invocation Types: Synchronous (wait for response) or Asynchronous (fire and forget)
Week 4: API GatewayWhat is API Gateway?
API Gateway creates REST APIs that trigger Lambda functions. It handles HTTP requests, routing, authentication, rate limiting, and more. It's your API's front door.1. Creating a REST APIbashaws --endpoint-url=$AWS_ENDPOINT apigateway create-rest-api \
  --name "my-learning-api" \
  --description "API for learning"Expected Output:
json{
    "id": "abc123xyz",
    "name": "my-learning-api",
    "createdDate": "2025-11-17T12:00:00+00:00"
}Save the API ID:
bashAPI_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-rest-apis \
  --query "items[?name=='my-learning-api'].id" \
  --output text)

echo $API_IDWhat's an API ID? Unique identifier for your API. You'll need it for all subsequent operations.2. Getting the Root ResourceEvery API starts with a root resource (/):bashROOT_RESOURCE_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-resources \
  --rest-api-id $API_ID \
  --query 'items[0].id' \
  --output text)

echo $ROOT_RESOURCE_ID3. Creating a Resource (Endpoint Path)Create /hello endpoint:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-resource \
  --rest-api-id $API_ID \
  --parent-id $ROOT_RESOURCE_ID \
  --path-part helloExpected Output:
json{
    "id": "def456",
    "parentId": "abc123",
    "pathPart": "hello",
    "path": "/hello"
}Save the resource ID:
bashHELLO_RESOURCE_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-resources \
  --rest-api-id $API_ID \
  --query "items[?path=='/hello'].id" \
  --output text)What's a resource? A path in your API (like /hello, /users, /orders)4. Creating a Method (HTTP Verb)Add GET method to /hello:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway put-method \
  --rest-api-id $API_ID \
  --resource-id $HELLO_RESOURCE_ID \
  --http-method GET \
  --authorization-type NONEExpected Output:
json{
    "httpMethod": "GET",
    "authorizationType": "NONE"
}What's a method? The HTTP verb (GET, POST, PUT, DELETE) for your endpoint.5. Creating Mock Integration (Testing)Before connecting Lambda, let's create a mock response:bashaws --endpoint-url=$AWS_ENDPOINT apigateway put-integration \
  --rest-api-id $API_ID \
  --resource-id $HELLO_RESOURCE_ID \
  --http-method GET \
  --type MOCK \
  --request-templates '{"application/json": "{\"statusCode\": 200}"}'Expected Output:
json{
    "type": "MOCK",
    "requestTemplates": {
        "application/json": "{\"statusCode\": 200}"
    }
}Set up the response:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway put-integration-response \
  --rest-api-id $API_ID \
  --resource-id $HELLO_RESOURCE_ID \
  --http-method GET \
  --status-code 200 \
  --response-templates '{"application/json": "{\"message\": \"Hello from API Gateway!\"}"}'Add method response:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway put-method-response \
  --rest-api-id $API_ID \
  --resource-id $HELLO_RESOURCE_ID \
  --http-method GET \
  --status-code 200What's the difference?

Integration: Backend (Lambda, HTTP, Mock)
Integration Response: Transform backend response
Method Response: Define API's response contract
6. Deploying the APICreate a deployment stage:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name devExpected Output:
json{
    "id": "deploy123",
    "createdDate": "2025-11-17T12:05:00+00:00"
}What's a stage? Environment for your API (dev, staging, prod). Each has its own URL.7. Testing Your APIbashcurl http://localhost:4566/restapis/$API_ID/dev/_user_request_/helloExpected Output:
json{"message": "Hello from API Gateway!"}Success! Your first API endpoint is working.8. Integrating with LambdaFirst, create a Lambda function for the API:
bashcat > api_lambda.py << 'EOF'
import json

def lambda_handler(event, context):
    # API Gateway sends event with request details
    http_method = event.get('httpMethod')
    path = event.get('path')
    query_params = event.get('queryStringParameters', {})
    body = event.get('body', '')
    
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': json.dumps({
            'message': 'Hello from Lambda via API Gateway!',
            'method': http_method,
            'path': path,
            'query': query_params
        })
    }
EOF

zip api_lambda.zip api_lambda.py

aws --endpoint-url=$AWS_ENDPOINT lambda create-function \
  --function-name api-backend \
  --runtime python3.11 \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --handler api_lambda.lambda_handler \
  --zip-file fileb://api_lambda.zipGet Lambda ARN:
bashLAMBDA_ARN=$(aws --endpoint-url=$AWS_ENDPOINT lambda get-function \
  --function-name api-backend \
  --query 'Configuration.FunctionArn' \
  --output text)

echo $LAMBDA_ARNCreate /greet resource with POST method:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-resource \
  --rest-api-id $API_ID \
  --parent-id $ROOT_RESOURCE_ID \
  --path-part greet

GREET_RESOURCE_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-resources \
  --rest-api-id $API_ID \
  --query "items[?path=='/greet'].id" \
  --output text)

aws --endpoint-url=$AWS_ENDPOINT apigateway put-method \
  --rest-api-id $API_ID \
  --resource-id $GREET_RESOURCE_ID \
  --http-method POST \
  --authorization-type NONEConnect to Lambda:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway put-integration \
  --rest-api-id $API_ID \
  --resource-id $GREET_RESOURCE_ID \
  --http-method POST \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/$LAMBDA_ARN/invocations"Expected Output:
json{
    "type": "AWS_PROXY",
    "httpMethod": "POST",
    "uri": "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:000000000000:function:api-backend/invocations"
}What's AWS_PROXY? Lambda handles the entire request/response. Simpler than configuring mappings.Deploy again:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name devTest it:
bashcurl -X POST http://localhost:4566/restapis/$API_ID/dev/_user_request_/greet \
  -H "Content-Type: application/json" \
  -d '{"name": "Sijan"}'Expected Output:
json{
    "message": "Hello from Lambda via API Gateway!",
    "method": "POST",
    "path": "/greet",
    "query": null
}9. Path ParametersCreate an endpoint with path parameter: /users/{userId}bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-resource \
  --rest-api-id $API_ID \
  --parent-id $ROOT_RESOURCE_ID \
  --path-part users

USERS_RESOURCE_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-resources \
  --rest-api-id $API_ID \
  --query "items[?path=='/users'].id" \
  --output text)

aws --endpoint-url=$AWS_ENDPOINT apigateway create-resource \
  --rest-api-id $API_ID \
  --parent-id $USERS_RESOURCE_ID \
  --path-part '{userId}'

USER_ID_RESOURCE_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-resources \
  --rest-api-id $API_ID \
  --query "items[?path=='/users/{userId}'].id" \
  --output text)Add GET method:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway put-method \
  --rest-api-id $API_ID \
  --resource-id $USER_ID_RESOURCE_ID \
  --http-method GET \
  --authorization-type NONE \
  --request-parameters method.request.path.userId=trueWhat's request-parameters? Makes userId a required path parameter.Create Lambda to handle it:
bashcat > user_lambda.py << 'EOF'
import json

def lambda_handler(event, context):
    # Extract userId from path parameters
    user_id = event['pathParameters']['userId']
    
    # Simulate fetching user data
    user_data = {
        'userId': user_id,
        'name': f'User {user_id}',
        'email': f'user{user_id}@example.com'
    }
    
    return {
        'statusCode': 200,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps(user_data)
    }
EOF

zip user_lambda.zip user_lambda.py

aws --endpoint-url=$AWS_ENDPOINT lambda create-function \
  --function-name get-user \
  --runtime python3.11 \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --handler user_lambda.lambda_handler \
  --zip-file fileb://user_lambda.zip

USER_LAMBDA_ARN=$(aws --endpoint-url=$AWS_ENDPOINT lambda get-function \
  --function-name get-user \
  --query 'Configuration.FunctionArn' \
  --output text)Connect to API:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway put-integration \
  --rest-api-id $API_ID \
  --resource-id $USER_ID_RESOURCE_ID \
  --http-method GET \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/$USER_LAMBDA_ARN/invocations"

aws --endpoint-url=$AWS_ENDPOINT apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name devTest with different user IDs:
bashcurl http://localhost:4566/restapis/$API_ID/dev/_user_request_/users/123
curl http://localhost:4566/restapis/$API_ID/dev/_user_request_/users/456Expected Output:
json{"userId": "123", "name": "User 123", "email": "user123@example.com"}
{"userId": "456", "name": "User 456", "email": "user456@example.com"}10. Query ParametersUpdate the user Lambda to handle query parameters:
bashcat > user_lambda.py << 'EOF'
import json

def lambda_handler(event, context):
    user_id = event['pathParameters']['userId']
    query_params = event.get('queryStringParameters', {})
    
    # Check if detailed info requested
    include_details = query_params.get('details', 'false') == 'true'
    
    user_data = {
        'userId': user_id,
        'name': f'User {user_id}',
        'email': f'user{user_id}@example.com'
    }
    
    if include_details:
        user_data['address'] = '123 Main St'
        user_data['phone'] = '555-0100'
    
    return {
        'statusCode': 200,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps(user_data)
    }
EOF

zip user_lambda.zip user_lambda.py

aws --endpoint-url=$AWS_ENDPOINT lambda update-function-code \
  --function-name get-user \
  --zip-file fileb://user_lambda.zipTest with query parameter:
bashcurl "http://localhost:4566/restapis/$API_ID/dev/_user_request_/users/123?details=true"Expected Output:
json{
    "userId": "123",
    "name": "User 123",
    "email": "user123@example.com",
    "address": "123 Main St",
    "phone": "555-0100"
}11. CORS ConfigurationEnable CORS for browser requests:bash# Add OPTIONS method for preflight
aws --endpoint-url=$AWS_ENDPOINT apigateway put-method \
  --rest-api-id $API_ID \
  --resource-id $GREET_RESOURCE_ID \
  --http-method OPTIONS \
  --authorization-type NONE

# Mock integration for OPTIONS
aws --endpoint-url=$AWS_ENDPOINT apigateway put-integration \
  --rest-api-id $API_ID \
  --resource-id $GREET_RESOURCE_ID \
  --http-method OPTIONS \
  --type MOCK \
  --request-templates '{"application/json": "{\"statusCode\": 200}"}'

# Integration response with CORS headers
aws --endpoint-url=$AWS_ENDPOINT apigateway put-integration-response \
  --rest-api-id $API_ID \
  --resource-id $GREET_RESOURCE_ID \
  --http-method OPTIONS \
  --status-code 200 \
  --response-parameters '{
    "method.response.header.Access-Control-Allow-Headers": "'"'"'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'"'"'",
    "method.response.header.Access-Control-Allow-Methods": "'"'"'GET,POST,OPTIONS'"'"'",
    "method.response.header.Access-Control-Allow-Origin": "'"'"'*'"'"'"
  }'

# Method response
aws --endpoint-url=$AWS_ENDPOINT apigateway put-method-response \
  --rest-api-id $API_ID \
  --resource-id $GREET_RESOURCE_ID \
  --http-method OPTIONS \
  --status-code 200 \
  --response-parameters '{
    "method.response.header.Access-Control-Allow-Headers": false,
    "method.response.header.Access-Control-Allow-Methods": false,
    "method.response.header.Access-Control-Allow-Origin": false
  }'

aws --endpoint-url=$AWS_ENDPOINT apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name devWhy CORS? Browsers block API requests from different domains unless the API explicitly allows it.12. API Keys and Usage PlansCreate an API key:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-api-key \
  --name "my-api-key" \
  --enabledExpected Output:
json{
    "id": "key123",
    "value": "abc123def456",
    "name": "my-api-key",
    "enabled": true
}Create a usage plan:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-usage-plan \
  --name "basic-plan" \
  --throttle burstLimit=100,rateLimit=50 \
  --quota limit=10000,period=MONTHExpected Output:
json{
    "id": "plan123",
    "name": "basic-plan",
    "throttle": {
        "burstLimit": 100,
        "rateLimit": 50.0
    },
    "quota": {
        "limit": 10000,
        "period": "MONTH"
    }
}What's this?

Rate Limit: 50 requests per second (sustained)
Burst Limit: Up to 100 requests in a burst
Quota: 10,000 requests per month
Link API to usage plan:
bashUSAGE_PLAN_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-usage-plans \
  --query "items[?name=='basic-plan'].id" \
  --output text)

aws --endpoint-url=$AWS_ENDPOINT apigateway create-usage-plan-key \
  --usage-plan-id $USAGE_PLAN_ID \
  --key-id key123 \
  --key-type API_KEYUpdate method to require API key:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway update-method \
  --rest-api-id $API_ID \
  --resource-id $GREET_RESOURCE_ID \
  --http-method POST \
  --patch-operations op=replace,path=/apiKeyRequired,value=true

aws --endpoint-url=$AWS_ENDPOINT apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name devTest with API key:
bashcurl -X POST http://localhost:4566/restapis/$API_ID/dev/_user_request_/greet \
  -H "x-api-key: abc123def456" \
  -H "Content-Type: application/json" \
  -d '{"name": "Sijan"}'13. Request ValidationCreate a request model:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-model \
  --rest-api-id $API_ID \
  --name "GreetingRequest" \
  --content-type "application/json" \
  --schema '{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "type": "object",
    "required": ["name"],
    "properties": {
      "name": {
        "type": "string",
        "minLength": 1
      },
      "greeting": {
        "type": "string"
      }
    }
  }'What's a model? JSON Schema that defines the expected request structure.Create a request validator:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-request-validator \
  --rest-api-id $API_ID \
  --name "validate-body" \
  --validate-request-bodyApply to method:
bashVALIDATOR_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-request-validators \
  --rest-api-id $API_ID \
  --query "items[?name=='validate-body'].id" \
  --output text)

aws --endpoint-url=$AWS_ENDPOINT apigateway update-method \
  --rest-api-id $API_ID \
  --resource-id $GREET_RESOURCE_ID \
  --http-method POST \
  --patch-operations \
    op=replace,path=/requestValidatorId,value=$VALIDATOR_ID \
    op=replace,path=/requestModels/application~1json,value=GreetingRequestWhy validate? Catch bad requests before they reach your Lambda (saves money and improves security).14. Viewing API DocumentationExport API as OpenAPI (Swagger):
bashaws --endpoint-url=$AWS_ENDPOINT apigateway get-export \
  --rest-api-id $API_ID \
  --stage-name dev \
  --export-type swagger \
  swagger.jsonView it:
bashcat swagger.json | jqWhy export? Generate API documentation, import into other tools, share with team.15. Deleting ResourcesDelete API:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway delete-rest-api --rest-api-id $API_IDNo output means success.API Gateway Key Concepts Summary

REST API: Your API's container
Resource: URL path (e.g., /users, /orders)
Method: HTTP verb (GET, POST, PUT, DELETE)
Integration: Backend (Lambda, HTTP endpoint, Mock)
Stage: Environment (dev, prod) with unique URL
Deployment: Snapshot of API configuration pushed to a stage
AWS_PROXY: Lambda handles entire request/response
Path Parameters: Dynamic parts of URL (/users/{userId})
Query Parameters: Optional filters (?details=true)
API Key: Identifier for clients
Usage Plan: Rate limiting and quotas
Request Validation: JSON Schema to validate requests
CORS: Allow browser requests from other domains
Week 5: CognitoWhat is Cognito?
Cognito provides user authentication and authorization. It handles user sign-up, sign-in, password resets, MFA, and generates JWT tokens for API access. No need to build your own auth system.1. Creating a User Poolbashaws --endpoint-url=$AWS_ENDPOINT cognito-idp create-user-pool \
  --pool-name my-user-pool \
  --policies '{
    "PasswordPolicy": {
      "MinimumLength": 8,
      "RequireUppercase": true,
      "RequireLowercase": true,
      "RequireNumbers": true,
      "RequireSymbols": false
    }
  }' \
  --auto-verified-attributes email \
  --username-attributes emailExpected Output:
json{
    "UserPool": {
        "Id": "us-east-1_abc123",
        "Name": "my-user-pool",
        "Policies": {
            "PasswordPolicy": {
                "MinimumLength": 8,
                "RequireUppercase": true,
                "RequireLowercase": true,
                "RequireNumbers": true
            }
        },
        "AutoVerifiedAttributes": ["email"],
        "UsernameAttributes": ["email"]
    }
}Save the User Pool ID:
bashUSER_POOL_ID=$(aws --endpoint-url=$AWS_ENDPOINT cognito-idp list-user-pools \
  --max-results 10 \
  --query "UserPools[?Name=='my-user-pool'].Id" \
  --output text)

echo $USER_POOL_IDWhat's a User Pool? Directory of users with authentication capabilities.Why username-attributes email? Users can sign in with email instead of username.2. Creating an App Clientbashaws --endpoint-url=$AWS_ENDPOINT cognito-idp create-user-pool-client \
  --user-pool-id $USER_POOL_ID \
  --client-name my-app \
  --explicit-auth-flows ALLOW_USER_PASSWORD_AUTH ALLOW_REFRESH_TOKEN_AUTHExpected Output:
json{
    "UserPoolClient": {
        "ClientId": "abc123def456",
        "ClientName": "my-app",
        "UserPoolId": "us-east-1_abc123",
        "ExplicitAuthFlows": [
            "ALLOW_USER_PASSWORD_AUTH",
            "ALLOW_REFRESH_TOKEN_AUTH"
        ]
    }
}Save Client ID:
bashCLIENT_ID=$(aws --endpoint-url=$AWS_ENDPOINT cognito-idp list-user-pool-clients \
  --user-pool-id $USER_POOL_ID \
  --query "UserPoolClients[?ClientName=='my-app'].ClientId" \
  --output text)

echo $CLIENT_IDWhat's an App Client? Represents your application - gets its own client ID to interact with the user pool.Why explicit-auth-flows? Allows direct username/password authentication (simpler for learning, but use OAuth in production).3. Creating a User (Admin)bashaws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-create-user \
  --user-pool-id $USER_POOL_ID \
  --username sijan@example.com \
  --user-attributes Name=email,Value=sijan@example.com Name=email_verified,Value=true \
  --temporary-password TempPass123! \
  --message-action SUPPRESSExpected Output:
json{
    "User": {
        "Username": "sijan@example.com",
        "Attributes": [
            {
                "Name": "email",
                "Value": "sijan@example.com"
            },
            {
                "Name": "email_verified",
                "Value": "true"
            }
        ],
        "UserStatus": "FORCE_CHANGE_PASSWORD",
        "Enabled": true
    }
}What's FORCE_CHANGE_PASSWORD? User must change password on first login.Why SUPPRESS? Don't send welcome email (useful in LocalStack).4. Setting a Permanent Passwordbashaws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-set-user-password \
  --user-pool-id $USER_POOL_ID \
  --username sijan@example.com \
  --password MySecurePass123! \
  --permanentNo output means success.User status is now: CONFIRMED (ready to sign in)5. User Authentication (Sign In)bashaws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-initiate-auth \
  --user-pool-id $USER_POOL_ID \
  --client-id $CLIENT_ID \
  --auth-flow ADMIN_NO_SRP_AUTH \
  --auth-parameters USERNAME=sijan@example.com,PASSWORD=MySecurePass123!Expected Output:
json{
    "AuthenticationResult": {
        "AccessToken": "eyJraWQiOiJ...",
        "IdToken": "eyJraWQiOiJ...",
        "RefreshToken": "eyJjdHkiOiJ...",
        "TokenType": "Bearer",
        "ExpiresIn": 3600
    }
}What are these tokens?

AccessToken: Used to call your APIs (contains scopes/permissions)
IdToken: Contains user information (name, email, etc.)
RefreshToken: Get new access tokens when they expire (valid for 30 days)
Save the tokens:
bashACCESS_TOKEN=$(aws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-initiate-auth \
  --user-pool-id $USER_POOL_ID \
  --client-id $CLIENT_ID \
  --auth-flow ADMIN_NO_SRP_AUTH \
  --auth-parameters USERNAME=sijan@example.com,PASSWORD=MySecurePass123! \
  --query 'AuthenticationResult.AccessToken' \
  --output text)

echo $ACCESS_TOKEN6. Verifying a TokenDecode the IdToken to see user info:
bashID_TOKEN=$(aws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-initiate-auth \
  --user-pool-id $USER_POOL_ID \
  --client-id $CLIENT_ID \
  --auth-flow ADMIN_NO_SRP_AUTH \
  --auth-parameters USERNAME=sijan@example.com,PASSWORD=MySecurePass123! \
  --query 'AuthenticationResult.IdToken' \
  --output text)

# Decode JWT (requires jq and base64)
echo $ID_TOKEN | cut -d'.' -f2 | base64 -d 2>/dev/null | jqExpected Output:
json{
  "sub": "abc-123-def",
  "email_verified": true,
  "email": "sijan@example.com",
  "cognito:username": "sijan@example.com",
  "exp": 1700236800,
  "iat": 1700233200
}What's sub? Unique user ID (never changes, even if email changes).7. Getting User InformationWith access token:
bashaws --endpoint-url=$AWS_ENDPOINT cognito-idp get-user \
  --access-token $ACCESS_TOKENExpected Output:
json{
    "Username": "sijan@example.com",
    "UserAttributes": [
        {
            "Name": "email",
            "Value": "sijan@example.com"
        },
        {
            "Name": "email_verified",
            "Value": "true"
        }
    ]
}8. Refreshing TokensWhen access token expires (after 1 hour):
bashREFRESH_TOKEN=$(aws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-initiate-auth \
  --user-pool-id $USER_POOL_ID \
  --client-id $CLIENT_ID \
  --auth-flow ADMIN_NO_SRP_AUTH \
  --auth-parameters USERNAME=sijan@example.com,PASSWORD=MySecurePass123! \
  --query 'AuthenticationResult.RefreshToken' \
  --output text)

aws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-initiate-auth \
  --user-pool-id $USER_POOL_ID \
  --client-id $CLIENT_ID \
  --auth-flow REFRESH_TOKEN_AUTH \
  --auth-parameters REFRESH_TOKEN=$REFRESH_TOKENExpected Output:
json{
    "AuthenticationResult": {
        "AccessToken": "eyJraWQiOiJ...",
        "IdToken": "eyJraWQiOiJ...",
        "TokenType": "Bearer",
        "ExpiresIn": 3600
    }
}Note: No new refresh token (the old one still works).9. Protecting API Gateway with CognitoCreate a Cognito authorizer for your API:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway create-authorizer \
  --rest-api-id $API_ID \
  --name cognito-authorizer \
  --type COGNITO_USER_POOLS \
  --provider-arns arn:aws:cognito-idp:us-east-1:000000000000:userpool/$USER_POOL_ID \
  --identity-source method.request.header.AuthorizationExpected Output:
json{
    "id": "auth123",
    "name": "cognito-authorizer",
    "type": "COGNITO_USER_POOLS",
    "providerARNs": [
        "arn:aws:cognito-idp:us-east-1:000000000000:userpool/us-east-1_abc123"
    ]
}Save authorizer ID:
bashAUTHORIZER_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-authorizers \
  --rest-api-id $API_ID \
  --query "items[?name=='cognito-authorizer'].id" \
  --output text)Update method to use authorizer:
bashaws --endpoint-url=$AWS_ENDPOINT apigateway update-method \
  --rest-api-id $API_ID \
  --resource-id $GREET_RESOURCE_ID \
  --http-method POST \
  --patch-operations \
    op=replace,path=/authorizationType,value=COGNITO_USER_POOLS \
    op=replace,path=/authorizerId,value=$AUTHORIZER_ID

aws --endpoint-url=$AWS_ENDPOINT apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name devTest without token (should fail):
bashcurl -X POST http://localhost:4566/restapis/$API_ID/dev/_user_request_/greet \
  -H "Content-Type: application/json" \
  -d '{"name": "Sijan"}'Expected Output:
json{"message": "Unauthorized"}Test with token (should succeed):
bashcurl -X POST http://localhost:4566/restapis/$API_ID/dev/_user_request_/greet \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "Sijan"}'Success! Your API is now protected by Cognito authentication.10. Custom User AttributesAdd custom attributes to user pool (must be done at creation):
bashaws --endpoint-url=$AWS_ENDPOINT cognito-idp create-user-pool \
  --pool-name custom-attributes-pool \
  --schema '[
    {
      "Name": "department",
      "AttributeDataType": "String",
      "Mutable": true
    },
    {
      "Name": "employee_id",
      "AttributeDataType": "Number",
      "Mutable": false
    }
  ]'Mutable true/false? Can users change this attribute after creation?11. User GroupsCreate a group:
bashaws --endpoint-url=$AWS_ENDPOINT cognito-idp create-group \
  --group-name admins \
  --user-pool-id $USER_POOL_ID \
  --description "Administrator users"Expected Output:
json{
    "Group": {
        "GroupName": "admins",
        "UserPoolId": "us-east-1_abc123",
        "Description": "Administrator users"
    }
}Add user to group:
bashaws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-add-user-to-group \
  --user-pool-id $USER_POOL_ID \
  --username sijan@example.com \
  --group-name adminsNo output means success.List user's groups:
bashaws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-list-groups-for-user \
  --user-pool-id $USER_POOL_ID \
  --username sijan@example.comExpected Output:
json{
    "Groups": [
        {
            "GroupName": "admins",
            "UserPoolId": "us-east-1_abc123",
            "Description": "Administrator users"
        }
    ]
}Why groups? Implement role-based access control (RBAC). Groups appear in JWT tokens.12. Listing All Usersbashaws --endpoint-url=$AWS_ENDPOINT cognito-idp list-users \
  --user-pool-id $USER_POOL_IDExpected Output:
json{
    "Users": [
        {
            "Username": "sijan@example.com",
            "Attributes": [
                {
                    "Name": "email",
                    "Value": "sijan@example.com"
                }
            ],
            "UserStatus": "CONFIRMED",
            "Enabled": true
        }
    ]
}13. Disabling/Enabling UsersDisable user:
bashaws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-disable-user \
  --user-pool-id $USER_POOL_ID \
  --username sijan@example.comEnable user:
bashaws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-enable-user \
  --user-pool-id $USER_POOL_ID \
  --username sijan@example.comWhy disable? Temporarily block access without deleting the user.14. Deleting a Userbashaws --endpoint-url=$AWS_ENDPOINT cognito-idp admin-delete-user \
  --user-pool-id $USER_POOL_ID \
  --username sijan@example.comNo output means success.Warning: This is permanent!15. Deleting User Poolbashaws --endpoint-url=$AWS_ENDPOINT cognito-idp delete-user-pool \
  --user-pool-id $USER_POOL_IDCognito Key Concepts Summary

User Pool: Directory of users
App Client: Application that can interact with user pool
User: Individual account with credentials
Access Token: For API authorization (short-lived, 1 hour)
ID Token: User information (claims)
Refresh Token: Get new access tokens (long-lived, 30 days)
JWT: JSON Web Token (contains encoded user info)
Groups: Organize users for role-based access
Custom Attributes: Additional user properties
Authorizer: API Gateway integration for authentication
Week 6: SES (Simple Email Service)What is SES?
SES sends transactional emails (password resets, notifications, alerts). It's not for marketing emails. In LocalStack, emails are simulated - they won't actually send.1. Verifying an Email AddressBefore sending, verify the "from" address:
bashaws --endpoint-url=$AWS_ENDPOINT ses verify-email-identity \
  --email-address noreply@example.comNo output means success.Check verification status:
bashaws --endpoint-url=$AWS_ENDPOINT ses get-identity-verification-attributes \
  --identities noreply@example.comExpected Output:
json{
    "VerificationAttributes": {
        "noreply@example.com": {
            "VerificationStatus": "Success"
        }
    }
}In real AWS: You'd receive a verification email. In LocalStack, it's auto-verified.2. Sending a Simple Emailbashaws --endpoint-url=$AWS_ENDPOINT ses send-email \
  --from noreply@example.com \
  --destination ToAddresses=sijan@example.com \
  --message "Subject={Data='Welcome to Our Service'},Body={Text={Data='Thank you for signing up!'}}"Expected Output:
json{
    "MessageId": "abc123-def456-789"
}What's MessageId? Unique identifier for this email (for tracking).3. Sending HTML Emailbashaws --endpoint-url=$AWS_ENDPOINT ses send-email \
  --from noreply@example.com \
  --destination ToAddresses=sijan@example.com \
  --message '{
    "Subject": {
      "Data": "Your Weekly Report"
    },
    "Body": {
      "Html": {
        "Data": "<html><body><h1>Weekly Report</h1><p>Here are your stats:</p><ul><li>Orders: 42</li><li>Revenue: $1,234</li></ul></body></html>"
      }
    }
  }'Why HTML? Better formatting, images, links, branding.4. Sending to Multiple Recipientsbashaws --endpoint-url=$AWS_ENDPOINT ses send-email \
  --from noreply@example.com \
  --destination ToAddresses=user1@example.com,user2@example.com CcAddresses=manager@example.com \
  --message "Subject={Data='Team Meeting Tomorrow'},Body={Text={Data='Don\'t forget our 10am meeting!'}}"ToAddresses vs CcAddresses?

To: Primary recipients
Cc: Carbon copy (everyone sees who received it)
Bcc: Blind carbon copy (recipients don't see each other) - add with BccAddresses
5. Sending Raw Email (MIME)For attachments, use raw email:
bashcat > email.txt << 'EOF'
From: noreply@example.com
To: sijan@example.com
Subject: Invoice #12345
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="boundary123"

--boundary123
Content-Type: text/plain

Dear Customer,

Please find your invoice attached.

--boundary123
Content-Type: application/pdf; name="invoice.pdf"
Content-Disposition: attachment; filename="invoice.pdf"
Content-Transfer-Encoding: base64

JVBERi0xLjQKJeLjz9MKMyAwIG9iago8PC9UeXBlIC9QYWdlCi9QYXJlbnQgMSAwIFIKPj4KZW5kb2JqCg==

--boundary123--
EOF

aws --endpoint-url=$AWS_ENDPOINT ses send-raw-email \
  --raw-message file://email.txtWhat's MIME? Standard for email structure (text, HTML, attachments).6. Using Email TemplatesCreate a template:
bashaws --endpoint-url=$AWS_ENDPOINT ses create-template \
  --template '{
    "TemplateName": "WelcomeEmail",
    "SubjectPart": "Welcome {{name}}!",
    "TextPart": "Hi {{name}},\n\nThank you for joining us!\n\nYour account ID: {{accountId}}",
    "HtmlPart": "<html><body><h1>Welcome {{name}}!</h1><p>Your account ID: <strong>{{accountId}}</strong></p></body></html>"
  }'No output means success.Send using template:
bashaws --endpoint-url=$AWS_ENDPOINT ses send-templated-email \
  --source noreply@example.com \
  --destination ToAddresses=sijan@example.com \
  --template WelcomeEmail \
  --template-data '{"name": "Sijan", "accountId": "ACC-12345"}'Expected Output:
json{
    "MessageId": "template-email-123"
}Why templates? Consistent branding, easy to update, supports variables.7. Listing Email Templatesbashaws --endpoint-url=$AWS_ENDPOINT ses list-templatesExpected Output:
json{
    "TemplatesMetadata": [
        {
            "Name": "WelcomeEmail",
            "CreatedTimestamp": "2025-11-17T13:00:00+00:00"
        }
    ]
}8. Updating a Templatebashaws --endpoint-url=$AWS_ENDPOINT ses update-template \
  --template '{
    "TemplateName": "WelcomeEmail",
    "SubjectPart": "Welcome aboard, {{name}}!",
    "TextPart": "Hi {{name}},\n\nWelcome! Your account: {{accountId}}\n\nBest regards,\nThe Team",
    "HtmlPart": "<html><body><h1>Welcome aboard, {{name}}!</h1><p>Account: {{accountId}}</p></body></html>"
  }'9. Lambda Trigger for SESAutomate email sending with Lambda:
bashcat > email_lambda.py << 'EOF'
import boto3
import json

ses = boto3.client('ses', endpoint_url='http://localhost:4566')

def lambda_handler(event, context):
    # Example: Send email when new user signs up
    user_email = event.get('email')
    user_name = event.get('name')
    
    response = ses.send_email(
        Source='noreply@example.com',
        Destination={'ToAddresses': [user_email]},
        Message={
            'Subject': {'Data': f'Welcome {user_name}!'},
            'Body': {
                'Text': {'Data': f'Hi {user_name},\n\nThank you for signing up!'}
            }
        }
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'Email sent',
            'messageId': response['MessageId']
        })
    }
EOF

zip email_lambda.zip email_lambda.py

aws --endpoint-url=$AWS_ENDPOINT lambda create-function \
  --function-name send-welcome-email \
  --runtime python3.11 \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --handler email_lambda.lambda_handler \
  --zip-file fileb://email_lambda.zipTest it:
bashaws --endpoint-url=$AWS_ENDPOINT lambda invoke \
  --function-name send-welcome-email \
  --payload '{"email": "newuser@example.com", "name": "New User"}' \
  response.json

cat response.json10. Configuration Sets (Advanced)Track email metrics:
bashaws --endpoint-url=$AWS_ENDPOINT ses create-configuration-set \
  --configuration-set Name=analyticsSend email with tracking:
bashaws --endpoint-url=$AWS_ENDPOINT ses send-email \
  --from noreply@example.com \
  --destination ToAddresses=sijan@example.com \
  --message "Subject={Data='Tracked Email'},Body={Text={Data='This email is tracked'}}" \
  --configuration-set-name analyticsWhat's tracked? Sends, deliveries, opens, clicks, bounces, complaints.11. Checking Send Statisticsbashaws --endpoint-url=$AWS_ENDPOINT ses get-send-statisticsExpected Output:
json{
    "SendDataPoints": [
        {
            "Timestamp": "2025-11-17T13:00:00+00:00",
            "DeliveryAttempts": 5,
            "Bounces": 0,
            "Complaints": 0,
            "Rejects": 0
        }
    ]
}12. Listing Verified Emailsbashaws --endpoint-url=$AWS_ENDPOINT ses list-identitiesExpected Output:
json{
    "Identities": [
        "noreply@example.com"
    ]
}13. Deleting a Templatebashaws --endpoint-url=$AWS_ENDPOINT ses delete-template --template-name WelcomeEmail14. Removing Verified Emailbashaws --endpoint-url=$AWS_ENDPOINT ses delete-identity --identity noreply@example.comSES Key Concepts Summary

Identity: Verified email or domain you can send from
Simple Email: Text or HTML email
Raw Email: MIME format (for attachments)
Template: Reusable email with variables
Configuration Set: Track email metrics
MessageId: Unique identifier for each email
Bounce: Email couldn't be delivered
Complaint: Recipient marked as spam
Integration ProjectsNow that you know each service, let's build real-world integrations!Project 1: User Registration FlowArchitecture:
API Gateway → Lambda → Cognito + SESStep 1: Create Registration Lambda
bashcat > register_user.py << 'EOF'
import boto3
import json
import os

cognito = boto3.client('cognito-idp', endpoint_url='http://localhost:4566')
ses = boto3.client('ses', endpoint_url='http://localhost:4566')

USER_POOL_ID = os.environ['USER_POOL_ID']
CLIENT_ID = os.environ['CLIENT_ID']

def lambda_handler(event, context):
    body = json.loads(event['body'])
    email = body['email']
    password = body['password']
    name = body['name']
    
    try:
        # Create user in Cognito
        cognito.admin_create_user(
            UserPoolId=USER_POOL_ID,
            Username=email,
            UserAttributes=[
                {'Name': 'email', 'Value': email},
                {'Name': 'email_verified', 'Value': 'true'},
                {'Name': 'name', 'Value': name}
            ],
            MessageAction='SUPPRESS'
        )
        
        # Set permanent password
        cognito.admin_set_user_password(
            UserPoolId=USER_POOL_ID,
            Username=email,
            Password=password,
            Permanent=True
        )
        
        # Send welcome email
        ses.send_email(
            Source='noreply@example.com',
            Destination={'ToAddresses': [email]},
            Message={
                'Subject': {'Data': f'Welcome {name}!'},
                'Body': {
                    'Html': {
                        'Data': f'<h1>Welcome {name}!</h1><p>Your account has been created successfully.</p>'
                    }
                }
            }
        )
        
        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({
                'message': 'User registered successfully',
                'email': email
            })
        }
        
    except Exception as e:
        return {
            'statusCode': 400,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({'error': str(e)})
        }
EOF

zip register_user.zip register_user.py

aws --endpoint-url=$AWS_ENDPOINT lambda create-function \
  --function-name register-user \
  --runtime python3.11 \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --handler register_user.lambda_handler \
  --zip-file fileb://register_user.zip \
  --environment "Variables={USER_POOL_ID=$USER_POOL_ID,CLIENT_ID=$CLIENT_ID}"Step 2: Create API Endpoint
bash# Add /register resource
aws --endpoint-url=$AWS_ENDPOINT apigateway create-resource \
  --rest-api-id $API_ID \
  --parent-id $ROOT_RESOURCE_ID \
  --path-part register

REGISTER_RESOURCE_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-resources \
  --rest-api-id $API_ID \
  --query "items[?path=='/register'].id" \
  --output text)

# Add POST method
aws --endpoint-url=$AWS_ENDPOINT apigateway put-method \
  --rest-api-id $API_ID \
  --resource-id $REGISTER_RESOURCE_ID \
  --http-method POST \
  --authorization-type NONE

# Connect to Lambda
REGISTER_LAMBDA_ARN=$(aws --endpoint-url=$AWS_ENDPOINT lambda get-function \
  --function-name register-user \
  --query 'Configuration.FunctionArn' \
  --output text)

aws --endpoint-url=$AWS_ENDPOINT apigateway put-integration \
  --rest-api-id $API_ID \
  --resource-id $REGISTER_RESOURCE_ID \
  --http-method POST \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/$REGISTER_LAMBDA_ARN/invocations"

# Deploy
aws --endpoint-url=$AWS_ENDPOINT apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name devStep 3: Test Registration
bashcurl -X POST http://localhost:4566/restapis/$API_ID/dev/_user_request_/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "testuser@example.com",
    "password": "SecurePass123!",
    "name": "Test User"
  }'Expected Output:
json{
    "message": "User registered successfully",
    "email": "testuser@example.com"
}What happened?

API Gateway received registration request
Lambda created user in Cognito
Lambda sent welcome email via SES
User can now log in
Project 2: Order Processing PipelineArchitecture:
API Gateway → Lambda → SQS → Lambda → S3 + SESWhat it does: Customer places order → stored in queue → processed asynchronously → receipt saved to S3 → confirmation email sentStep 1: Create Orders Queue
bashaws --endpoint-url=$AWS_ENDPOINT sqs create-queue --queue-name orders-queue

ORDERS_QUEUE_URL=$(aws --endpoint-url=$AWS_ENDPOINT sqs get-queue-url \
  --queue-name orders-queue \
  --query 'QueueUrl' \
  --output text)

ORDERS_QUEUE_ARN=$(aws --endpoint-url=$AWS_ENDPOINT sqs get-queue-attributes \
  --queue-url $ORDERS_QUEUE_URL \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' \
  --output text)Step 2: Create Order Submission Lambda
bashcat > submit_order.py << 'EOF'
import boto3
import json
import os
from datetime import datetime

sqs = boto3.client('sqs', endpoint_url='http://localhost:4566')

QUEUE_URL = os.environ['QUEUE_URL']

def lambda_handler(event, context):
    body = json.loads(event['body'])
    
    order = {
        'orderId': f"ORD-{datetime.now().strftime('%Y%m%d-%H%M%S')}",
        'customerId': body['customerId'],
        'items': body['items'],
        'total': body['total'],
        'timestamp': datetime.now().isoformat()
    }
    
    # Send to queue
    sqs.send_message(
        QueueUrl=QUEUE_URL,
        MessageBody=json.dumps(order)
    )
    
    return {
        'statusCode': 202,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps({
            'message': 'Order received',
            'orderId': order['orderId']
        })
    }
EOF

zip submit_order.zip submit_order.py

aws --endpoint-url=$AWS_ENDPOINT lambda create-function \
  --function-name submit-order \
  --runtime python3.11 \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --handler submit_order.lambda_handler \
  --zip-file fileb://submit_order.zip \
  --environment "Variables={QUEUE_URL=$ORDERS_QUEUE_URL}"Step 3: Create Order Processor Lambda
bashcat > process_order.py << 'EOF'
import boto3
import json
import os

s3 = boto3.client('s3', endpoint_url='http://localhost:4566')
ses = boto3.client('ses', endpoint_url='http://localhost:4566')

BUCKET_NAME = 'order-receipts'

def lambda_handler(event, context):
    for record in event['Records']:
        order = json.loads(record['body'])
        order_id = order['orderId']
        
        # Save receipt to S3
        receipt = f"Order ID: {order_id}\nTotal: ${order['total']}\n"
        
        s3.put_object(
            Bucket=BUCKET_NAME,
            Key=f"receipts/{order_id}.txt",
            Body=receipt
        )
        
        # Send confirmation email
        # (In real app, get customer email from database)
        customer_email = 'customer@example.com'
        
        ses.send_email(
            Source='orders@example.com',
            Destination={'ToAddresses': [customer_email]},
            Message={
                'Subject': {'Data': f'Order Confirmation: {order_id}'},
                'Body': {
                    'Text': {'Data': f'Your order {order_id} has been processed!\n\nTotal: ${order["total"]}'}
                }
            }
        )
        
        print(f"Processed order: {order_id}")
    
    return {'statusCode': 200}
EOF

# Create bucket
aws --endpoint-url=$AWS_ENDPOINT s3 mb s3://order-receipts

zip process_order.zip process_order.py

aws --endpoint-url=$AWS_ENDPOINT lambda create-function \
  --function-name process-order \
  --runtime python3.11 \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --handler process_order.lambda_handler \
  --zip-file fileb://process_order.zipStep 4: Connect SQS to Processor
bashaws --endpoint-url=$AWS_ENDPOINT lambda create-event-source-mapping \
  --function-name process-order \
  --event-source-arn $ORDERS_QUEUE_ARN \
  --batch-size 10Step 5: Create API Endpoint
bash# Add /orders resource
aws --endpoint-url=$AWS_ENDPOINT apigateway create-resource \
  --rest-api-id $API_ID \
  --parent-id $ROOT_RESOURCE_ID \
  --path-part orders

ORDERS_RESOURCE_ID=$(aws --endpoint-url=$AWS_ENDPOINT apigateway get-resources \
  --rest-api-id $API_ID \
  --query "items[?path=='/orders'].id" \
  --output text)

aws --endpoint-url=$AWS_ENDPOINT apigateway put-method \
  --rest-api-id $API_ID \
  --resource-id $ORDERS_RESOURCE_ID \
  --http-method POST \
  --authorization-type NONE

SUBMIT_ORDER_ARN=$(aws --endpoint-url=$AWS_ENDPOINT lambda get-function \
  --function-name submit-order \
  --query 'Configuration.FunctionArn' \
  --output text)

aws --endpoint-url=$AWS_ENDPOINT apigateway put-integration \
  --rest-api-id $API_ID \
  --resource-id $ORDERS_RESOURCE_ID \
  --http-method POST \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/$SUBMIT_ORDER_ARN/invocations"

aws --endpoint-url=$AWS_ENDPOINT apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name devStep 6: Test End-to-End
bash# Submit order
curl -X POST http://localhost:4566/restapis/$API_ID/dev/_user_request_/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "CUST-123",
    "items": [
      {"name": "Laptop", "price": 999.99}
    ],
    "total": 999.99
  }'

# Wait a few seconds for processing

# Check receipt in S3
aws --endpoint-url=$AWS_ENDPOINT s3 ls s3://order-receipts/receipts/

# Download receipt
aws --endpoint-url=$AWS_ENDPOINT s3 cp s3://order-receipts/receipts/ORD-XXXXXXX.txt order.txt
cat order.txtWhat happened?

Customer submitted order via API
Lambda queued order in SQS (instant response)
Processor Lambda picked up message
Receipt saved to S3
Confirmation email sent
Order processing complete!
Why this architecture?

Decoupled: API responds immediately, processing happens async
Scalable: Multiple processors can handle queue
Reliable: If processor fails, message stays in queue
Auditable: All receipts stored in S3
Project 3: Authenticated File UploadArchitecture:
API Gateway (Cognito Auth) → Lambda → S3 Pre-signed URL → Direct UploadWhat it does: Authenticated users get temporary URLs to upload files directly to S3Step 1: Create Upload Lambda
bashcat > generate_upload_url.py << 'EOF'
import boto3
import json
import os

s3 = boto3.client('s3', endpoint_url='http://localhost:4566')

BUCKET_NAME = 'user-uploads'

def lambda_handler(event, context):
    # Extract user info from Cognito token (in authorizer context)
    user_id = event['requestContext']['authorizer']['claims']['sub']
    
    # Get filename from request
    body = json.loads(event['body'])
    filename = body['filename']
