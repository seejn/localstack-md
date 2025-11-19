# AWS SQS (Simple Queue Service) - Complete Practical Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Key Concepts](#key-concepts)
3. [Queue Management](#queue-management)
4. [Sending Messages](#sending-messages)
5. [Receiving Messages](#receiving-messages)
6. [Message Visibility & Deletion](#message-visibility--deletion)
7. [Dead Letter Queues (DLQ)](#dead-letter-queues-dlq)
8. [FIFO Queues](#fifo-queues)
9. [Queue Attributes & Configuration](#queue-attributes--configuration)
10. [Permissions & Access Control](#permissions--access-control)
11. [Integration with Lambda](#integration-with-lambda)
12. [Monitoring & CloudWatch](#monitoring--cloudwatch)
13. [Best Practices](#best-practices)

---

## Introduction

Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications.

**Use Cases:**
- Decoupling application components
- Asynchronous processing
- Load leveling and buffering
- Batch processing
- Message fanout patterns
- Work queues
- Event-driven architectures

**Benefits:**
- Fully managed (no infrastructure)
- Unlimited throughput and messages
- Low latency (single-digit milliseconds)
- At-least-once delivery (Standard) or exactly-once (FIFO)
- Pay-per-use pricing
- Reliable message storage

---

## Key Concepts

### Queue Types

**Standard Queue:**
- Nearly unlimited throughput
- Best-effort ordering
- At-least-once delivery (may receive duplicates)
- Lowest cost

**FIFO Queue:**
- Up to 3,000 messages/second (with batching: 30,000 msg/sec)
- First-In-First-Out ordering
- Exactly-once processing
- Higher cost than Standard
- Queue name must end with `.fifo`

### Message Attributes
- **Message Body**: Up to 256 KB of text
- **Message Attributes**: Structured metadata (optional)
- **Message ID**: Unique identifier
- **Receipt Handle**: Token for deleting/modifying message

### Visibility Timeout
- Duration message is invisible after being received
- Prevents multiple consumers processing same message
- Default: 30 seconds
- Range: 0 seconds to 12 hours

### Retention Period
- How long messages are kept if not deleted
- Default: 4 days
- Range: 1 minute to 14 days

### Delay Queues
- Postpone delivery of new messages
- Default: 0 seconds
- Range: 0 to 15 minutes

---

## Queue Management

### 1. Create Standard Queue

```bash
aws sqs create-queue --queue-name my-standard-queue
```

**Expected Output:**
```json
{
    "QueueUrl": "https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue"
}
```

**Why:** Create a standard queue for asynchronous message processing.

**When to Use:** Decoupling services, work queues, event processing.

**LocalStack:**
```bash
aws sqs create-queue --queue-name my-standard-queue --endpoint-url=http://localhost:4566
```

---

### 2. Create Queue with Attributes

```bash
aws sqs create-queue \
    --queue-name my-configured-queue \
    --attributes '{
        "DelaySeconds": "5",
        "MaximumMessageSize": "262144",
        "MessageRetentionPeriod": "1209600",
        "VisibilityTimeout": "60",
        "ReceiveMessageWaitTimeSeconds": "10"
    }'
```

**Expected Output:**
```json
{
    "QueueUrl": "https://sqs.us-east-1.amazonaws.com/123456789012/my-configured-queue"
}
```

**Why:** Configure queue parameters at creation.

**When to Use:** Setting up queue with specific requirements.

**Attributes:**
- **DelaySeconds**: Delivery delay (0-900 seconds)
- **MaximumMessageSize**: Message size limit (1024-262144 bytes)
- **MessageRetentionPeriod**: Retention period (60-1209600 seconds)
- **VisibilityTimeout**: Visibility timeout (0-43200 seconds)
- **ReceiveMessageWaitTimeSeconds**: Long polling (0-20 seconds)

---

### 3. List Queues

```bash
aws sqs list-queues
```

**Expected Output:**
```json
{
    "QueueUrls": [
        "https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue",
        "https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-queue.fifo",
        "https://sqs.us-east-1.amazonaws.com/123456789012/my-dlq"
    ]
}
```

**Why:** View all queues in account/region.

**When to Use:** Queue discovery, auditing, inventory.

---

### 4. List Queues with Prefix Filter

```bash
aws sqs list-queues --queue-name-prefix prod-
```

**Expected Output:**
```json
{
    "QueueUrls": [
        "https://sqs.us-east-1.amazonaws.com/123456789012/prod-orders",
        "https://sqs.us-east-1.amazonaws.com/123456789012/prod-notifications"
    ]
}
```

**Why:** Filter queues by name prefix.

**When to Use:** Finding specific queues, environment filtering.

---

### 5. Get Queue URL

```bash
aws sqs get-queue-url --queue-name my-standard-queue
```

**Expected Output:**
```json
{
    "QueueUrl": "https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue"
}
```

**Why:** Get queue URL from queue name.

**When to Use:** Need URL for other operations.

---

### 6. Get Queue Attributes

```bash
aws sqs get-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attribute-names All
```

**Expected Output:**
```json
{
    "Attributes": {
        "QueueArn": "arn:aws:sqs:us-east-1:123456789012:my-standard-queue",
        "ApproximateNumberOfMessages": "5",
        "ApproximateNumberOfMessagesNotVisible": "2",
        "ApproximateNumberOfMessagesDelayed": "0",
        "CreatedTimestamp": "1700227200",
        "DelaySeconds": "0",
        "MaximumMessageSize": "262144",
        "MessageRetentionPeriod": "345600",
        "VisibilityTimeout": "30",
        "ReceiveMessageWaitTimeSeconds": "0"
    }
}
```

**Why:** View queue configuration and statistics.

**When to Use:** Monitoring, troubleshooting, capacity planning.

**Key Metrics:**
- **ApproximateNumberOfMessages**: Messages available
- **ApproximateNumberOfMessagesNotVisible**: In-flight messages
- **ApproximateNumberOfMessagesDelayed**: Delayed messages

---

### 7. Set Queue Attributes

```bash
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attributes '{
        "VisibilityTimeout": "60",
        "MessageRetentionPeriod": "604800",
        "ReceiveMessageWaitTimeSeconds": "20"
    }'
```

**Expected Output:**
```
(No output on success)
```

**Why:** Modify queue configuration.

**When to Use:** Tuning performance, changing retention, adjusting timeouts.

---

### 8. Purge Queue

```bash
aws sqs purge-queue \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue
```

**Expected Output:**
```
(No output on success)
```

**Why:** Delete all messages in queue.

**When to Use:** Testing, clearing old messages, emergency cleanup.

**Warning:** Cannot be undone. Messages deleted immediately.

**Note:** Can only purge once every 60 seconds.

---

### 9. Delete Queue

```bash
aws sqs delete-queue \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove queue permanently.

**When to Use:** Decommissioning, cleanup.

**Warning:** All messages are deleted. Cannot be recovered.

**Note:** Deletion takes up to 60 seconds. Queue URL remains valid during deletion.

---

## Sending Messages

### 1. Send Message

```bash
aws sqs send-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --message-body "This is a test message"
```

**Expected Output:**
```json
{
    "MessageId": "12345678-1234-1234-1234-123456789012",
    "MD5OfMessageBody": "098f6bcd4621d373cade4e832627b4f6"
}
```

**Why:** Send single message to queue.

**When to Use:** Publishing events, sending tasks, notifications.

---

### 2. Send Message with Attributes

```bash
aws sqs send-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --message-body "Order placed" \
    --message-attributes '{
        "OrderId": {
            "DataType": "String",
            "StringValue": "ORD-12345"
        },
        "Priority": {
            "DataType": "Number",
            "StringValue": "1"
        },
        "Timestamp": {
            "DataType": "Number",
            "StringValue": "1700227200"
        }
    }'
```

**Expected Output:**
```json
{
    "MessageId": "23456789-2345-2345-2345-234567890123",
    "MD5OfMessageBody": "d41d8cd98f00b204e9800998ecf8427e",
    "MD5OfMessageAttributes": "3ae8f24a8d1e131f6e2c2c8e2f9c3e4d"
}
```

**Why:** Send structured metadata with message.

**When to Use:** Filtering, routing, providing context without parsing body.

**Data Types:**
- **String**: Text values
- **Number**: Numeric values (stored as string)
- **Binary**: Binary data (base64 encoded)
- **String.Array**, **Number.Array**, **Binary.Array**: Arrays (custom)

---

### 3. Send Message with Delay

```bash
aws sqs send-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --message-body "Delayed message" \
    --delay-seconds 300
```

**Expected Output:**
```json
{
    "MessageId": "34567890-3456-3456-3456-345678901234"
}
```

**Why:** Delay message delivery.

**When to Use:** Scheduled tasks, retry delays, rate limiting.

**Delay Range:** 0-900 seconds (15 minutes)

---

### 4. Send Message Batch

```bash
aws sqs send-message-batch \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --entries file://messages.json
```

**messages.json:**
```json
[
    {
        "Id": "msg1",
        "MessageBody": "First message"
    },
    {
        "Id": "msg2",
        "MessageBody": "Second message",
        "DelaySeconds": 60
    },
    {
        "Id": "msg3",
        "MessageBody": "Third message",
        "MessageAttributes": {
            "Type": {
                "DataType": "String",
                "StringValue": "Order"
            }
        }
    }
]
```

**Expected Output:**
```json
{
    "Successful": [
        {
            "Id": "msg1",
            "MessageId": "45678901-4567-4567-4567-456789012345",
            "MD5OfMessageBody": "..."
        },
        {
            "Id": "msg2",
            "MessageId": "56789012-5678-5678-5678-567890123456",
            "MD5OfMessageBody": "..."
        },
        {
            "Id": "msg3",
            "MessageId": "67890123-6789-6789-6789-678901234567",
            "MD5OfMessageBody": "..."
        }
    ],
    "Failed": []
}
```

**Why:** Send up to 10 messages efficiently in single request.

**When to Use:** Bulk operations, high throughput, cost optimization.

**Advantages:**
- More efficient than individual sends
- Lower cost (billed as 1 request)
- Better performance

**Limits:**
- Max 10 messages per batch
- Max 256 KB total payload

---

### 5. Send JSON Message

```bash
# Create JSON payload
cat > order.json << 'EOF'
{
    "orderId": "ORD-12345",
    "customerId": "CUST-67890",
    "items": [
        {"productId": "PROD-111", "quantity": 2},
        {"productId": "PROD-222", "quantity": 1}
    ],
    "total": 99.99
}
EOF

# Send as message body
aws sqs send-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --message-body file://order.json
```

**Expected Output:**
```json
{
    "MessageId": "78901234-7890-7890-7890-789012345678"
}
```

**Why:** Send structured data as message.

**When to Use:** Complex payloads, JSON APIs, event data.

---

## Receiving Messages

### 1. Receive Message

```bash
aws sqs receive-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue
```

**Expected Output:**
```json
{
    "Messages": [
        {
            "MessageId": "12345678-1234-1234-1234-123456789012",
            "ReceiptHandle": "AQEBwJnKyrHigUMZj6rYigCgxlaS3SLy0a...",
            "MD5OfBody": "098f6bcd4621d373cade4e832627b4f6",
            "Body": "This is a test message"
        }
    ]
}
```

**Why:** Retrieve messages from queue.

**When to Use:** Processing work items, polling for events.

**Important:** Receipt handle is required for deleting message.

---

### 2. Receive Multiple Messages

```bash
aws sqs receive-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --max-number-of-messages 10
```

**Expected Output:**
```json
{
    "Messages": [
        {
            "MessageId": "msg-1",
            "ReceiptHandle": "handle-1...",
            "Body": "Message 1"
        },
        {
            "MessageId": "msg-2",
            "ReceiptHandle": "handle-2...",
            "Body": "Message 2"
        }
    ]
}
```

**Why:** Retrieve up to 10 messages in single call.

**When to Use:** Batch processing, improved throughput.

**Max Messages:** 1-10 (default: 1)

**Note:** May return fewer messages than requested even if queue has more.

---

### 3. Receive Message with Attributes

```bash
aws sqs receive-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attribute-names All \
    --message-attribute-names All
```

**Expected Output:**
```json
{
    "Messages": [
        {
            "MessageId": "23456789-2345-2345-2345-234567890123",
            "ReceiptHandle": "AQEBwJnKyrHigUMZj6rYigCgxlaS3SLy0a...",
            "Body": "Order placed",
            "Attributes": {
                "SenderId": "AIDAIT2UOQQY3AUEKVGXU",
                "SentTimestamp": "1700227200000",
                "ApproximateReceiveCount": "1",
                "ApproximateFirstReceiveTimestamp": "1700227260000"
            },
            "MessageAttributes": {
                "OrderId": {
                    "DataType": "String",
                    "StringValue": "ORD-12345"
                },
                "Priority": {
                    "DataType": "Number",
                    "StringValue": "1"
                }
            }
        }
    ]
}
```

**Why:** Get message metadata and custom attributes.

**When to Use:** Need sender info, timestamps, or custom attributes.

**System Attributes:**
- **SenderId**: AWS account ID or IAM role
- **SentTimestamp**: When message was sent
- **ApproximateReceiveCount**: Receive attempts
- **ApproximateFirstReceiveTimestamp**: First receive time

---

### 4. Long Polling

```bash
aws sqs receive-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --wait-time-seconds 20
```

**Expected Output:**
```json
{
    "Messages": [
        {
            "MessageId": "34567890-3456-3456-3456-345678901234",
            "ReceiptHandle": "AQEBwJnKyrHigUMZj6rYigCgxlaS3SLy0a...",
            "Body": "Message body"
        }
    ]
}
```

**Why:** Wait for messages instead of immediate return (reduces empty responses).

**When to Use:** Reducing API calls, cost optimization, efficient polling.

**Wait Time:** 0-20 seconds

**Benefits:**
- Reduces empty responses (fewer wasted API calls)
- Lower cost
- Messages delivered as soon as available
- Better for most use cases than short polling

**Short Polling (default):** Returns immediately, may return empty even if messages arrive soon.

---

### 5. Receive and Process Loop (Bash)

```bash
#!/bin/bash
QUEUE_URL="https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue"

while true; do
    # Receive message with long polling
    RESPONSE=$(aws sqs receive-message \
        --queue-url "$QUEUE_URL" \
        --max-number-of-messages 10 \
        --wait-time-seconds 20 \
        --attribute-names All)

    # Check if messages received
    if echo "$RESPONSE" | jq -e '.Messages' > /dev/null; then
        # Process each message
        echo "$RESPONSE" | jq -c '.Messages[]' | while read -r message; do
            BODY=$(echo "$message" | jq -r '.Body')
            RECEIPT_HANDLE=$(echo "$message" | jq -r '.ReceiptHandle')

            echo "Processing: $BODY"

            # Process message (your logic here)
            # ...

            # Delete message after successful processing
            aws sqs delete-message \
                --queue-url "$QUEUE_URL" \
                --receipt-handle "$RECEIPT_HANDLE"

            echo "Deleted message"
        done
    else
        echo "No messages available"
    fi
done
```

**Why:** Continuously poll and process messages.

**When to Use:** Worker processes, queue consumers.

---

## Message Visibility & Deletion

### 1. Delete Message

```bash
aws sqs delete-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --receipt-handle "AQEBwJnKyrHigUMZj6rYigCgxlaS3SLy0a..."
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove message after successful processing.

**When to Use:** After processing message, preventing reprocessing.

**Important:** Must delete message or it becomes visible again after timeout.

---

### 2. Delete Message Batch

```bash
aws sqs delete-message-batch \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --entries file://delete-batch.json
```

**delete-batch.json:**
```json
[
    {
        "Id": "msg1",
        "ReceiptHandle": "AQEBwJnKyrHigUMZj6rYigCgxlaS3SLy0a..."
    },
    {
        "Id": "msg2",
        "ReceiptHandle": "AQEBxKoLzsHjhVNZk7sZjhDhymbtT4Uy1b..."
    }
]
```

**Expected Output:**
```json
{
    "Successful": [
        {
            "Id": "msg1"
        },
        {
            "Id": "msg2"
        }
    ],
    "Failed": []
}
```

**Why:** Delete multiple messages efficiently.

**When to Use:** Batch processing, improved performance.

**Max:** 10 messages per batch

---

### 3. Change Message Visibility

```bash
aws sqs change-message-visibility \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --receipt-handle "AQEBwJnKyrHigUMZj6rYigCgxlaS3SLy0a..." \
    --visibility-timeout 300
```

**Expected Output:**
```
(No output on success)
```

**Why:** Extend or shorten processing time for message.

**When to Use:** Long processing, need more time, early release.

**Use Cases:**
- Extend timeout if processing takes longer
- Set to 0 to make message immediately visible again (release back to queue)
- Implement custom retry logic

---

### 4. Change Message Visibility Batch

```bash
aws sqs change-message-visibility-batch \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --entries file://visibility-batch.json
```

**visibility-batch.json:**
```json
[
    {
        "Id": "msg1",
        "ReceiptHandle": "AQEBwJnKyrHigUMZj6rYigCgxlaS3SLy0a...",
        "VisibilityTimeout": 300
    },
    {
        "Id": "msg2",
        "ReceiptHandle": "AQEBxKoLzsHjhVNZk7sZjhDhymbtT4Uy1b...",
        "VisibilityTimeout": 600
    }
]
```

**Expected Output:**
```json
{
    "Successful": [
        {
            "Id": "msg1"
        },
        {
            "Id": "msg2"
        }
    ],
    "Failed": []
}
```

**Why:** Change visibility for multiple messages.

**When to Use:** Batch processing with variable processing times.

---

## Dead Letter Queues (DLQ)

### 1. Create Dead Letter Queue

```bash
aws sqs create-queue --queue-name my-dlq
```

**Expected Output:**
```json
{
    "QueueUrl": "https://sqs.us-east-1.amazonaws.com/123456789012/my-dlq"
}
```

**Why:** Create queue for failed messages.

**When to Use:** Error handling, debugging, manual intervention.

---

### 2. Get DLQ ARN

```bash
aws sqs get-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-dlq \
    --attribute-names QueueArn
```

**Expected Output:**
```json
{
    "Attributes": {
        "QueueArn": "arn:aws:sqs:us-east-1:123456789012:my-dlq"
    }
}
```

**Why:** Get ARN needed for redrive policy.

**When to Use:** Configuring source queue.

---

### 3. Configure Redrive Policy

```bash
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attributes '{
        "RedrivePolicy": "{\"deadLetterTargetArn\":\"arn:aws:sqs:us-east-1:123456789012:my-dlq\",\"maxReceiveCount\":\"3\"}"
    }'
```

**Expected Output:**
```
(No output on success)
```

**Why:** Route failed messages to DLQ after max receive attempts.

**When to Use:** Error handling, preventing poison messages.

**maxReceiveCount:** How many times message can be received before moving to DLQ.

**Flow:**
1. Consumer receives message
2. Processing fails, message returns to queue
3. After maxReceiveCount attempts, moved to DLQ
4. Investigate and manually process or discard

---

### 4. Start DLQ Redrive

```bash
aws sqs start-message-move-task \
    --source-arn arn:aws:sqs:us-east-1:123456789012:my-dlq
```

**Expected Output:**
```json
{
    "TaskHandle": "AQEB6nR4..."
}
```

**Why:** Move messages back from DLQ to source queue (after fixing issue).

**When to Use:** Reprocessing after bug fix, recovering from failures.

---

### 5. List DLQ Redrive Tasks

```bash
aws sqs list-message-move-tasks \
    --source-arn arn:aws:sqs:us-east-1:123456789012:my-dlq
```

**Expected Output:**
```json
{
    "Results": [
        {
            "TaskHandle": "AQEB6nR4...",
            "Status": "RUNNING",
            "SourceArn": "arn:aws:sqs:us-east-1:123456789012:my-dlq",
            "ApproximateNumberOfMessagesMoved": 150,
            "ApproximateNumberOfMessagesToMove": 200
        }
    ]
}
```

**Why:** Monitor redrive progress.

**When to Use:** Tracking message movement from DLQ.

---

## FIFO Queues

### 1. Create FIFO Queue

```bash
aws sqs create-queue \
    --queue-name my-fifo-queue.fifo \
    --attributes '{
        "FifoQueue": "true",
        "ContentBasedDeduplication": "true"
    }'
```

**Expected Output:**
```json
{
    "QueueUrl": "https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-queue.fifo"
}
```

**Why:** Create queue with guaranteed ordering and exactly-once processing.

**When to Use:** Order-critical operations, financial transactions, no duplicates.

**Requirements:**
- Queue name must end with `.fifo`
- Must set FifoQueue attribute to "true"

**ContentBasedDeduplication:** Automatically deduplicate based on message body hash.

---

### 2. Send Message to FIFO Queue

```bash
aws sqs send-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-queue.fifo \
    --message-body "Order processing step 1" \
    --message-group-id "order-12345" \
    --message-deduplication-id "dedup-001"
```

**Expected Output:**
```json
{
    "MessageId": "12345678-1234-1234-1234-123456789012",
    "SequenceNumber": "123456789012345678901"
}
```

**Why:** Send ordered message with deduplication.

**When to Use:** Sequential operations, preventing duplicates.

**Message Group ID:** Messages with same group ID processed in order.
**Message Deduplication ID:** Prevents duplicates within 5-minute window.

---

### 3. Send Message with Content-Based Deduplication

```bash
aws sqs send-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-queue.fifo \
    --message-body "Unique message content" \
    --message-group-id "group-1"
```

**Expected Output:**
```json
{
    "MessageId": "23456789-2345-2345-2345-234567890123",
    "SequenceNumber": "123456789012345678902"
}
```

**Why:** Automatic deduplication based on message body (no deduplication ID needed).

**When to Use:** Queue has ContentBasedDeduplication enabled, simpler code.

---

### 4. Send Message Batch to FIFO Queue

```bash
aws sqs send-message-batch \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-queue.fifo \
    --entries file://fifo-batch.json
```

**fifo-batch.json:**
```json
[
    {
        "Id": "msg1",
        "MessageBody": "Step 1",
        "MessageGroupId": "order-12345",
        "MessageDeduplicationId": "dedup-001"
    },
    {
        "Id": "msg2",
        "MessageBody": "Step 2",
        "MessageGroupId": "order-12345",
        "MessageDeduplicationId": "dedup-002"
    },
    {
        "Id": "msg3",
        "MessageBody": "Step 1",
        "MessageGroupId": "order-67890",
        "MessageDeduplicationId": "dedup-003"
    }
]
```

**Expected Output:**
```json
{
    "Successful": [
        {
            "Id": "msg1",
            "MessageId": "34567890-3456-3456-3456-345678901234",
            "SequenceNumber": "123456789012345678903"
        },
        {
            "Id": "msg2",
            "MessageId": "45678901-4567-4567-4567-456789012345",
            "SequenceNumber": "123456789012345678904"
        },
        {
            "Id": "msg3",
            "MessageId": "56789012-5678-5678-5678-567890123456",
            "SequenceNumber": "123456789012345678905"
        }
    ]
}
```

**Why:** Send multiple ordered messages efficiently.

**When to Use:** Batch operations requiring order, high throughput.

**Note:** Messages with same MessageGroupId processed in order. Different groups can be processed in parallel.

---

### 5. Create FIFO DLQ

```bash
# Create FIFO DLQ
aws sqs create-queue --queue-name my-fifo-dlq.fifo --attributes FifoQueue=true

# Get DLQ ARN
DLQ_ARN=$(aws sqs get-queue-attributes --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-dlq.fifo --attribute-names QueueArn --query 'Attributes.QueueArn' --output text)

# Configure redrive policy
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-queue.fifo \
    --attributes '{
        "RedrivePolicy": "{\"deadLetterTargetArn\":\"'"$DLQ_ARN"'\",\"maxReceiveCount\":\"3\"}"
    }'
```

**Why:** Error handling for FIFO queues.

**When to Use:** Production FIFO queues.

**Note:** FIFO queue DLQ must also be FIFO.

---

## Queue Attributes & Configuration

### 1. Set Delay Queue

```bash
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attributes DelaySeconds=300
```

**Expected Output:**
```
(No output on success)
```

**Why:** Delay all messages in queue by default.

**When to Use:** Rate limiting, scheduled processing.

**Range:** 0-900 seconds (15 minutes)

**Note:** Per-message delay (in send-message) overrides queue default.

---

### 2. Set Message Retention Period

```bash
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attributes MessageRetentionPeriod=604800
```

**Expected Output:**
```
(No output on success)
```

**Why:** Control how long messages are kept.

**When to Use:** Compliance, storage optimization.

**Range:** 60 seconds (1 minute) to 1,209,600 seconds (14 days)
**Default:** 345,600 seconds (4 days)

---

### 3. Set Maximum Message Size

```bash
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attributes MaximumMessageSize=65536
```

**Expected Output:**
```
(No output on success)
```

**Why:** Limit message size.

**When to Use:** Controlling costs, preventing large payloads.

**Range:** 1,024 bytes (1 KB) to 262,144 bytes (256 KB)
**Default:** 262,144 bytes

**Note:** For larger payloads, store in S3 and send S3 reference in message.

---

### 4. Enable Server-Side Encryption (SSE)

```bash
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attributes '{
        "KmsMasterKeyId": "alias/aws/sqs",
        "KmsDataKeyReusePeriodSeconds": "300"
    }'
```

**Expected Output:**
```
(No output on success)
```

**Why:** Encrypt messages at rest.

**When to Use:** Security compliance, sensitive data.

**KMS Key Options:**
- **alias/aws/sqs**: AWS managed key (free)
- **Custom KMS key ARN**: Customer managed key (better control, costs apply)

**Data Key Reuse Period:** 60-86,400 seconds (1 minute to 24 hours)

---

### 5. Add Tags to Queue

```bash
aws sqs tag-queue \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --tags Environment=Production,Application=Orders,CostCenter=Engineering
```

**Expected Output:**
```
(No output on success)
```

**Why:** Organize queues, cost allocation, automation.

**When to Use:** Resource management, billing, filtering.

---

### 6. List Queue Tags

```bash
aws sqs list-queue-tags \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue
```

**Expected Output:**
```json
{
    "Tags": {
        "Environment": "Production",
        "Application": "Orders",
        "CostCenter": "Engineering"
    }
}
```

**Why:** View queue tags.

**When to Use:** Auditing, cost analysis.

---

### 7. Remove Tags from Queue

```bash
aws sqs untag-queue \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --tag-keys Environment CostCenter
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove specific tags.

**When to Use:** Tag cleanup, reorganization.

---

## Permissions & Access Control

### 1. Add Queue Policy

```bash
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attributes file://queue-policy.json
```

**queue-policy.json:**
```json
{
    "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"AWS\":\"arn:aws:iam::123456789012:role/MyLambdaRole\"},\"Action\":\"sqs:*\",\"Resource\":\"arn:aws:sqs:us-east-1:123456789012:my-standard-queue\"}]}"
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Grant permissions to other AWS accounts or services.

**When to Use:** Cross-account access, service integrations.

---

### 2. Allow SNS to Send to Queue

```bash
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attributes file://sns-policy.json
```

**sns-policy.json:**
```json
{
    "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"sns.amazonaws.com\"},\"Action\":\"sqs:SendMessage\",\"Resource\":\"arn:aws:sqs:us-east-1:123456789012:my-standard-queue\",\"Condition\":{\"ArnEquals\":{\"aws:SourceArn\":\"arn:aws:sns:us-east-1:123456789012:my-topic\"}}}]}"
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Allow SNS topic to publish to queue (fanout pattern).

**When to Use:** SNS to SQS integration, event fanout.

---

### 3. Allow S3 to Send to Queue

```bash
aws sqs set-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue \
    --attributes file://s3-policy.json
```

**s3-policy.json:**
```json
{
    "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"s3.amazonaws.com\"},\"Action\":\"sqs:SendMessage\",\"Resource\":\"arn:aws:sqs:us-east-1:123456789012:my-standard-queue\",\"Condition\":{\"ArnEquals\":{\"aws:SourceArn\":\"arn:aws:s3:::my-bucket\"}}}]}"
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Allow S3 bucket to send notifications to queue.

**When to Use:** S3 event processing.

---

## Integration with Lambda

### 1. Create Lambda Event Source Mapping

```bash
aws lambda create-event-source-mapping \
    --function-name my-queue-processor \
    --event-source-arn arn:aws:sqs:us-east-1:123456789012:my-standard-queue \
    --batch-size 10 \
    --maximum-batching-window-in-seconds 5
```

**Expected Output:**
```json
{
    "UUID": "12345678-1234-1234-1234-123456789012",
    "BatchSize": 10,
    "MaximumBatchingWindowInSeconds": 5,
    "EventSourceArn": "arn:aws:sqs:us-east-1:123456789012:my-standard-queue",
    "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-queue-processor",
    "State": "Creating"
}
```

**Why:** Automatically invoke Lambda when messages arrive.

**When to Use:** Serverless queue processing, event-driven architecture.

**Batch Size:** 1-10 messages per invocation (max 10,000 for FIFO)
**Batching Window:** 0-300 seconds (wait for batch to fill)

**Lambda automatically:**
- Polls queue
- Invokes function with batch of messages
- Deletes messages if function succeeds
- Returns messages to queue if function fails

---

### 2. Lambda Function for SQS Processing (Python)

```python
import json

def lambda_handler(event, context):
    # Process each message in batch
    for record in event['Records']:
        # Extract message data
        message_id = record['messageId']
        receipt_handle = record['receiptHandle']
        body = record['body']
        attributes = record.get('messageAttributes', {})

        print(f"Processing message: {message_id}")
        print(f"Body: {body}")

        # Parse JSON body if needed
        try:
            data = json.loads(body)
            # Process data...
            print(f"Processed order: {data.get('orderId')}")
        except json.JSONDecodeError:
            print(f"Invalid JSON: {body}")

    return {
        'statusCode': 200,
        'body': json.dumps(f'Processed {len(event["Records"])} messages')
    }
```

**Why:** Process SQS messages with Lambda.

**When to Use:** Serverless architectures, event processing.

---

### 3. Configure Partial Batch Failure

```bash
aws lambda update-event-source-mapping \
    --uuid 12345678-1234-1234-1234-123456789012 \
    --function-response-types ReportBatchItemFailures
```

**Expected Output:**
```json
{
    "UUID": "12345678-1234-1234-1234-123456789012",
    "FunctionResponseTypes": ["ReportBatchItemFailures"]
}
```

**Lambda function with partial failure handling:**
```python
def lambda_handler(event, context):
    failed_items = []

    for record in event['Records']:
        try:
            # Process message
            process_message(record)
        except Exception as e:
            print(f"Failed to process {record['messageId']}: {e}")
            failed_items.append({
                'itemIdentifier': record['messageId']
            })

    return {
        'batchItemFailures': failed_items
    }
```

**Why:** Only return failed messages to queue, delete successful ones.

**When to Use:** Improving efficiency, faster processing of successful messages.

---

## Monitoring & CloudWatch

### 1. Get Queue Metrics

```bash
aws cloudwatch get-metric-statistics \
    --namespace AWS/SQS \
    --metric-name ApproximateNumberOfMessagesVisible \
    --dimensions Name=QueueName,Value=my-standard-queue \
    --start-time 2025-11-17T00:00:00Z \
    --end-time 2025-11-17T23:59:59Z \
    --period 3600 \
    --statistics Average,Maximum
```

**Expected Output:**
```json
{
    "Datapoints": [
        {
            "Timestamp": "2025-11-17T10:00:00+00:00",
            "Average": 150.0,
            "Maximum": 250.0,
            "Unit": "Count"
        }
    ]
}
```

**Why:** Monitor queue depth and performance.

**When to Use:** Capacity planning, alerting, performance analysis.

**Key Metrics:**
- **ApproximateNumberOfMessagesVisible**: Available messages
- **ApproximateNumberOfMessagesNotVisible**: In-flight messages
- **ApproximateNumberOfMessagesDelayed**: Delayed messages
- **NumberOfMessagesSent**: Messages sent
- **NumberOfMessagesReceived**: Messages received
- **NumberOfMessagesDeleted**: Messages deleted
- **ApproximateAgeOfOldestMessage**: Age of oldest message (seconds)

---

### 2. Create CloudWatch Alarm

```bash
aws cloudwatch put-metric-alarm \
    --alarm-name high-queue-depth \
    --alarm-description "Alert when queue depth exceeds threshold" \
    --metric-name ApproximateNumberOfMessagesVisible \
    --namespace AWS/SQS \
    --statistic Average \
    --period 300 \
    --threshold 1000 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2 \
    --dimensions Name=QueueName,Value=my-standard-queue \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

**Expected Output:**
```
(No output on success)
```

**Why:** Get alerted when queue depth is high.

**When to Use:** Proactive monitoring, preventing backlogs.

---

### 3. Monitor DLQ

```bash
aws cloudwatch put-metric-alarm \
    --alarm-name dlq-messages \
    --alarm-description "Alert when messages in DLQ" \
    --metric-name ApproximateNumberOfMessagesVisible \
    --namespace AWS/SQS \
    --statistic Sum \
    --period 300 \
    --threshold 1 \
    --comparison-operator GreaterThanOrEqualToThreshold \
    --evaluation-periods 1 \
    --dimensions Name=QueueName,Value=my-dlq \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

**Expected Output:**
```
(No output on success)
```

**Why:** Get alerted when messages enter DLQ (indicating failures).

**When to Use:** Error monitoring, production systems.

---

## Best Practices

### Architecture
1. **Use DLQ** - Essential for production
2. **Set appropriate visibility timeout** - Based on processing time
3. **Implement idempotency** - Handle duplicate messages
4. **Use FIFO only when needed** - Standard is cheaper and faster
5. **Separate queues by priority** - High/low priority queues
6. **Use message attributes** - Metadata without parsing body
7. **Batch operations** - Send/receive/delete in batches

### Performance
1. **Long polling** - Reduce empty responses, lower cost
2. **Batch size tuning** - Balance latency and throughput
3. **Multiple consumers** - Scale horizontally
4. **Connection pooling** - Reuse connections
5. **Parallel processing** - Process messages concurrently

### Reliability
1. **Handle poison messages** - Use DLQ and maxReceiveCount
2. **Implement retry logic** - Exponential backoff
3. **Monitor age of oldest message** - Detect processing bottlenecks
4. **Set message retention** - Appropriate for use case
5. **Delete after successful processing** - Prevent reprocessing

### Security
1. **Enable encryption** - SSE-KMS for sensitive data
2. **Use IAM roles** - Not access keys
3. **Least privilege** - Minimal permissions
4. **VPC endpoints** - Private access to SQS
5. **Queue policies** - Control cross-account access

### Cost Optimization
1. **Use batching** - Fewer API calls
2. **Long polling** - Reduce empty receives
3. **Right-size retention** - Don't keep messages longer than needed
4. **Monitor unused queues** - Delete unused resources
5. **Use Standard when possible** - FIFO is more expensive

### Monitoring
1. **CloudWatch alarms** - Queue depth, age, DLQ
2. **Track metrics** - Sent, received, deleted
3. **Log processing errors** - CloudWatch Logs
4. **Monitor Lambda integrations** - Failures, throttling
5. **Regular DLQ reviews** - Investigate failures

---

## Troubleshooting

### Messages Not Being Received
- Check visibility timeout (messages might be invisible)
- Verify queue has messages (get-queue-attributes)
- Check long polling configuration
- Verify IAM permissions

### Messages Not Being Deleted
- Verify receipt handle is correct
- Check visibility timeout hasn't expired
- Ensure delete called after successful processing

### High DLQ Volume
- Review processing errors in logs
- Check visibility timeout (too short?)
- Investigate application bugs
- Monitor maxReceiveCount setting

### FIFO Queue Throughput Issues
- Check message group ID distribution
- Consider multiple queues
- Use batching (up to 30,000 msg/sec)
- Review consumer count

### Duplicate Messages (Standard Queue)
- This is expected behavior for Standard queues
- Implement idempotency in consumer
- Use FIFO queue if duplicates unacceptable
- Use deduplication IDs

---

## Quick Reference

```bash
# Queue management
aws sqs create-queue --queue-name NAME
aws sqs list-queues
aws sqs delete-queue --queue-url URL
aws sqs purge-queue --queue-url URL

# Sending
aws sqs send-message --queue-url URL --message-body BODY
aws sqs send-message-batch --queue-url URL --entries file://batch.json

# Receiving
aws sqs receive-message --queue-url URL --max-number-of-messages 10 --wait-time-seconds 20

# Deleting
aws sqs delete-message --queue-url URL --receipt-handle HANDLE
aws sqs delete-message-batch --queue-url URL --entries file://delete.json

# Attributes
aws sqs get-queue-attributes --queue-url URL --attribute-names All
aws sqs set-queue-attributes --queue-url URL --attributes KEY=VALUE

# FIFO
aws sqs create-queue --queue-name NAME.fifo --attributes FifoQueue=true
aws sqs send-message --queue-url URL --message-body BODY --message-group-id GROUP --message-deduplication-id DEDUP
```

---

## Summary

SQS provides reliable, scalable message queuing for distributed applications. Master these commands to:
- Decouple application components
- Build asynchronous workflows
- Implement reliable message processing
- Scale processing independently
- Handle failures gracefully

Practice sending and receiving messages, configuring queues, and integrating with Lambda. SQS's simplicity and reliability make it ideal for event-driven architectures.

For LocalStack: add `--endpoint-url=http://localhost:4566` to commands.

Happy queueing!
