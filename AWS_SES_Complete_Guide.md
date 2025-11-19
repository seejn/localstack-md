# AWS SES (Simple Email Service) - Complete Practical Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Key Concepts](#key-concepts)
3. [Email Address & Domain Verification](#email-address--domain-verification)
4. [Sending Emails](#sending-emails)
5. [Email Templates](#email-templates)
6. [Configuration Sets](#configuration-sets)
7. [Email Receiving](#email-receiving)
8. [Suppression Lists](#suppression-lists)
9. [Reputation & Deliverability](#reputation--deliverability)
10. [SMTP Credentials](#smtp-credentials)
11. [Event Publishing](#event-publishing)
12. [Testing & Sandbox](#testing--sandbox)
13. [Best Practices](#best-practices)

---

## Introduction

Amazon Simple Email Service (SES) is a cost-effective, flexible, and scalable email sending and receiving service that enables you to send and receive email using your own email addresses and domains.

**Use Cases:**
- Transactional emails (order confirmations, password resets)
- Marketing campaigns
- Automated notifications
- Email receiving and processing
- Bulk email sending

**Benefits:**
- Pay-as-you-go pricing ($0.10 per 1,000 emails)
- High deliverability rates
- Flexible deployment (SMTP, API, SDK)
- Email receiving capabilities
- Real-time sending analytics
- Built-in bounce and complaint handling

---

## Key Concepts

### Sending Limits
- **Sandbox**: 200 emails/day, only verified addresses
- **Production**: Request limit increase (can send millions)
- **Sending Rate**: Max emails per second

### Email Verification
- **Email Address**: Verify individual emails
- **Domain**: Verify entire domain (recommended)
- **DNS Records**: Required for domain verification

### Identities
- Verified email addresses or domains
- Source of outgoing emails (FROM address)

### Reputation
- **Bounce Rate**: Keep below 5%
- **Complaint Rate**: Keep below 0.1%
- High reputation = better deliverability

### Configuration Sets
- Group of rules for email sending
- Event publishing, IP pool management
- Reputation monitoring

---

## Email Address & Domain Verification

### 1. Verify Email Address

```bash
aws ses verify-email-identity --email-address john@example.com
```

**Expected Output:**
```
(No output on success)
```

**Why:** Verify email address to send emails from it.

**When to Use:** Individual email verification, testing, sandbox mode.

**Next Step:** Check email inbox for verification link from AWS.

**Verification Email:**
```
Subject: Amazon SES Email Verification Request
Click this link to verify: https://email-verification.us-east-1.amazonaws.com/...
```

**LocalStack:**
```bash
aws ses verify-email-identity --email-address john@example.com --endpoint-url=http://localhost:4566
```

---

### 2. List Verified Email Addresses

```bash
aws ses list-identities --identity-type EmailAddress
```

**Expected Output:**
```json
{
    "Identities": [
        "john@example.com",
        "noreply@example.com"
    ]
}
```

**Why:** View all verified email addresses.

**When to Use:** Auditing, checking verification status.

---

### 3. Get Identity Verification Status

```bash
aws ses get-identity-verification-attributes --identities john@example.com
```

**Expected Output:**
```json
{
    "VerificationAttributes": {
        "john@example.com": {
            "VerificationStatus": "Success"
        }
    }
}
```

**Why:** Check if email/domain is verified.

**When to Use:** Troubleshooting, confirming verification.

**Verification Statuses:**
- **Pending**: Awaiting verification
- **Success**: Verified
- **Failed**: Verification failed
- **TemporaryFailure**: Retry needed
- **NotStarted**: Not yet initiated

---

### 4. Verify Domain

```bash
aws ses verify-domain-identity --domain example.com
```

**Expected Output:**
```json
{
    "VerificationToken": "abc123def456ghi789jkl012mno345pqr678stu"
}
```

**Why:** Verify entire domain for email sending.

**When to Use:** Production deployments, multiple email addresses.

**Next Step:** Add TXT record to domain DNS.

**DNS Record:**
```
Type: TXT
Name: _amazonses.example.com
Value: abc123def456ghi789jkl012mno345pqr678stu
```

**Verify DNS propagation:**
```bash
dig TXT _amazonses.example.com
```

---

### 5. Set Up Domain DKIM

```bash
aws ses verify-domain-dkim --domain example.com
```

**Expected Output:**
```json
{
    "DkimTokens": [
        "token1abc123",
        "token2def456",
        "token3ghi789"
    ]
}
```

**Why:** Enable DKIM signing for better deliverability and authentication.

**When to Use:** Production deployments, preventing email spoofing.

**DKIM (DomainKeys Identified Mail):** Email authentication method.

**Add CNAME records to DNS:**
```
Name: token1abc123._domainkey.example.com
Value: token1abc123.dkim.amazonses.com

Name: token2def456._domainkey.example.com
Value: token2def456.dkim.amazonses.com

Name: token3ghi789._domainkey.example.com
Value: token3ghi789.dkim.amazonses.com
```

---

### 6. Get DKIM Attributes

```bash
aws ses get-identity-dkim-attributes --identities example.com
```

**Expected Output:**
```json
{
    "DkimAttributes": {
        "example.com": {
            "DkimEnabled": true,
            "DkimVerificationStatus": "Success",
            "DkimTokens": [
                "token1abc123",
                "token2def456",
                "token3ghi789"
            ]
        }
    }
}
```

**Why:** Check DKIM configuration status.

**When to Use:** Verifying DKIM setup, troubleshooting deliverability.

---

### 7. Set Custom MAIL FROM Domain

```bash
aws ses set-identity-mail-from-domain \
    --identity example.com \
    --mail-from-domain mail.example.com \
    --behavior-on-mx-failure UseDefaultValue
```

**Expected Output:**
```
(No output on success)
```

**Why:** Use custom MAIL FROM domain instead of amazonses.com.

**When to Use:** Brand consistency, better deliverability, DMARC alignment.

**Add MX and SPF records to DNS:**
```
Type: MX
Name: mail.example.com
Value: 10 feedback-smtp.us-east-1.amazonses.com

Type: TXT
Name: mail.example.com
Value: v=spf1 include:amazonses.com ~all
```

**Behavior on MX Failure:**
- **UseDefaultValue**: Use amazonses.com
- **RejectMessage**: Reject email

---

### 8. Delete Identity

```bash
aws ses delete-identity --identity john@example.com
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove verified email/domain.

**When to Use:** Cleanup, decommissioning.

---

## Sending Emails

### 1. Send Simple Email

```bash
aws ses send-email \
    --from noreply@example.com \
    --to john@example.com \
    --subject "Test Email" \
    --text "This is a test email from AWS SES."
```

**Expected Output:**
```json
{
    "MessageId": "0100017f8e9a1234-5678-90ab-cdef-1234567890ab-000000"
}
```

**Why:** Send simple text email.

**When to Use:** Notifications, alerts, testing.

---

### 2. Send Email with HTML Body

```bash
aws ses send-email \
    --from noreply@example.com \
    --to john@example.com \
    --subject "Welcome to Our Service" \
    --html file://email-body.html \
    --text "Welcome! This is the plain text version."
```

**email-body.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: Arial, sans-serif; }
        .header { background-color: #4CAF50; color: white; padding: 10px; }
        .content { padding: 20px; }
    </style>
</head>
<body>
    <div class="header">
        <h1>Welcome to Our Service!</h1>
    </div>
    <div class="content">
        <p>Thank you for signing up.</p>
        <a href="https://example.com">Get Started</a>
    </div>
</body>
</html>
```

**Expected Output:**
```json
{
    "MessageId": "0100017f8e9a5678-abcd-ef01-2345-6789abcdef01-000000"
}
```

**Why:** Send formatted HTML emails.

**When to Use:** Marketing, rich content, branded communications.

**Best Practice:** Always include text version for email clients that don't support HTML.

---

### 3. Send Email with Multiple Recipients

```bash
aws ses send-email \
    --from noreply@example.com \
    --to john@example.com jane@example.com \
    --cc manager@example.com \
    --bcc audit@example.com \
    --subject "Team Update" \
    --text "This is a team update."
```

**Expected Output:**
```json
{
    "MessageId": "0100017f8e9a9012-3456-7890-abcd-ef0123456789-000000"
}
```

**Why:** Send to multiple recipients.

**When to Use:** Team communications, distribution lists.

**Note:** Each recipient counts toward sending quota.

---

### 4. Send Email with Attachments (Raw Email)

```bash
aws ses send-raw-email --raw-message file://message.txt
```

**message.txt (MIME format):**
```
From: noreply@example.com
To: john@example.com
Subject: Invoice Attached
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="boundary123"

--boundary123
Content-Type: text/plain; charset=UTF-8

Please find your invoice attached.

--boundary123
Content-Type: application/pdf; name="invoice.pdf"
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="invoice.pdf"

JVBERi0xLjQKJeLjz9MKMyAwIG9iaiA8PC9UeXBlIC9QYWdlIC9QYXJlbnQgM...
--boundary123--
```

**Expected Output:**
```json
{
    "MessageId": "0100017f8e9aabcd-ef01-2345-6789-abcdef012345-000000"
}
```

**Why:** Send emails with attachments or complex MIME content.

**When to Use:** Invoices, reports, documents.

---

### 5. Send Email from File (Raw)

```bash
# Create email file
cat > raw-email.txt << 'EOF'
From: noreply@example.com
To: john@example.com
Subject: Test Raw Email
Content-Type: text/plain; charset=UTF-8

This is a raw email message.
EOF

# Send
aws ses send-raw-email --raw-message file://raw-email.txt
```

**Expected Output:**
```json
{
    "MessageId": "0100017f8e9adef0-1234-5678-90ab-cdef01234567-000000"
}
```

**Why:** Full control over email headers and content.

**When to Use:** Custom headers, advanced email features.

---

### 6. Send Email with Configuration Set

```bash
aws ses send-email \
    --from noreply@example.com \
    --to john@example.com \
    --subject "Test Email" \
    --text "Testing with configuration set" \
    --configuration-set-name my-config-set
```

**Expected Output:**
```json
{
    "MessageId": "0100017f8e9a0123-4567-89ab-cdef-012345678901-000000"
}
```

**Why:** Track email events, manage IP pools.

**When to Use:** Production monitoring, analytics.

---

### 7. Send Bulk Templated Email

```bash
aws ses send-bulk-templated-email \
    --source noreply@example.com \
    --template MyTemplate \
    --destinations file://destinations.json
```

**destinations.json:**
```json
[
    {
        "Destination": {
            "ToAddresses": ["john@example.com"]
        },
        "ReplacementTemplateData": "{\"name\":\"John\",\"orderId\":\"12345\"}"
    },
    {
        "Destination": {
            "ToAddresses": ["jane@example.com"]
        },
        "ReplacementTemplateData": "{\"name\":\"Jane\",\"orderId\":\"12346\"}"
    }
]
```

**Expected Output:**
```json
{
    "Status": [
        {
            "Status": "Success",
            "MessageId": "0100017f8e9a2345-6789-abcd-ef01-234567890123-000000"
        },
        {
            "Status": "Success",
            "MessageId": "0100017f8e9a3456-789a-bcde-f012-345678901234-000000"
        }
    ]
}
```

**Why:** Send personalized emails to many recipients efficiently.

**When to Use:** Marketing campaigns, bulk notifications.

**Advantages:**
- More efficient than individual sends
- Lower cost
- Better performance

---

## Email Templates

### 1. Create Email Template

```bash
aws ses create-template --template file://template.json
```

**template.json:**
```json
{
    "Template": {
        "TemplateName": "OrderConfirmation",
        "SubjectPart": "Order Confirmation - {{orderId}}",
        "TextPart": "Dear {{name}},\n\nYour order {{orderId}} has been confirmed.\n\nTotal: ${{total}}\n\nThank you!",
        "HtmlPart": "<!DOCTYPE html><html><body><h1>Order Confirmation</h1><p>Dear {{name}},</p><p>Your order <strong>{{orderId}}</strong> has been confirmed.</p><p>Total: <strong>${{total}}</strong></p><p>Thank you for your purchase!</p></body></html>"
    }
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Reusable email templates with variable substitution.

**When to Use:** Transactional emails, consistent branding.

**Template Variables:** Use {{variableName}} syntax (Handlebars).

---

### 2. List Email Templates

```bash
aws ses list-templates
```

**Expected Output:**
```json
{
    "TemplatesMetadata": [
        {
            "Name": "OrderConfirmation",
            "CreatedTimestamp": "2025-11-17T10:00:00+00:00"
        },
        {
            "Name": "PasswordReset",
            "CreatedTimestamp": "2025-11-17T09:00:00+00:00"
        }
    ]
}
```

**Why:** View all email templates.

**When to Use:** Template management, auditing.

---

### 3. Get Email Template

```bash
aws ses get-template --template-name OrderConfirmation
```

**Expected Output:**
```json
{
    "Template": {
        "TemplateName": "OrderConfirmation",
        "SubjectPart": "Order Confirmation - {{orderId}}",
        "TextPart": "Dear {{name}},...",
        "HtmlPart": "<!DOCTYPE html>..."
    }
}
```

**Why:** View template content.

**When to Use:** Reviewing templates, copying configuration.

---

### 4. Update Email Template

```bash
aws ses update-template --template file://updated-template.json
```

**updated-template.json:**
```json
{
    "Template": {
        "TemplateName": "OrderConfirmation",
        "SubjectPart": "Order #{{orderId}} Confirmed!",
        "TextPart": "Updated text...",
        "HtmlPart": "Updated HTML..."
    }
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Modify existing template.

**When to Use:** Content updates, design changes.

---

### 5. Send Templated Email

```bash
aws ses send-templated-email \
    --source noreply@example.com \
    --destination ToAddresses=john@example.com \
    --template OrderConfirmation \
    --template-data '{"name":"John Doe","orderId":"12345","total":"99.99"}'
```

**Expected Output:**
```json
{
    "MessageId": "0100017f8e9a4567-89ab-cdef-0123-456789abcdef-000000"
}
```

**Why:** Send email using template with personalized data.

**When to Use:** Transactional emails with dynamic content.

---

### 6. Test Template Rendering

```bash
aws ses test-render-template \
    --template-name OrderConfirmation \
    --template-data '{"name":"John Doe","orderId":"12345","total":"99.99"}'
```

**Expected Output:**
```json
{
    "Subject": "Order Confirmation - 12345",
    "TextPart": "Dear John Doe,\n\nYour order 12345 has been confirmed.\n\nTotal: $99.99\n\nThank you!",
    "HtmlPart": "<!DOCTYPE html><html><body><h1>Order Confirmation</h1><p>Dear John Doe,</p>..."
}
```

**Why:** Preview how template renders with specific data.

**When to Use:** Testing templates before sending, debugging.

---

### 7. Delete Email Template

```bash
aws ses delete-template --template-name OrderConfirmation
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove unused template.

**When to Use:** Cleanup, template deprecation.

---

## Configuration Sets

### 1. Create Configuration Set

```bash
aws ses create-configuration-set --configuration-set Name=my-config-set
```

**Expected Output:**
```
(No output on success)
```

**Why:** Group sending rules and event tracking.

**When to Use:** Production monitoring, IP pools, event publishing.

---

### 2. List Configuration Sets

```bash
aws ses list-configuration-sets
```

**Expected Output:**
```json
{
    "ConfigurationSets": [
        {
            "Name": "my-config-set"
        },
        {
            "Name": "production-config"
        }
    ]
}
```

**Why:** View all configuration sets.

**When to Use:** Managing configurations, auditing.

---

### 3. Describe Configuration Set

```bash
aws ses describe-configuration-set --configuration-set-name my-config-set
```

**Expected Output:**
```json
{
    "ConfigurationSet": {
        "Name": "my-config-set"
    },
    "EventDestinations": [],
    "TrackingOptions": {},
    "ReputationOptions": {
        "ReputationMetricsEnabled": false
    }
}
```

**Why:** Get detailed configuration set information.

**When to Use:** Reviewing settings, troubleshooting.

---

### 4. Add Event Destination (CloudWatch)

```bash
aws ses put-configuration-set-event-destination \
    --configuration-set-name my-config-set \
    --event-destination file://event-destination.json
```

**event-destination.json:**
```json
{
    "Name": "CloudWatchDestination",
    "Enabled": true,
    "MatchingEventTypes": ["send", "bounce", "complaint", "delivery", "open", "click"],
    "CloudWatchDestination": {
        "DimensionConfigurations": [
            {
                "DimensionName": "ses:configuration-set",
                "DimensionValueSource": "messageTag",
                "DefaultDimensionValue": "default"
            }
        ]
    }
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Track email events in CloudWatch.

**When to Use:** Monitoring, analytics, alerting.

**Event Types:**
- **send**: Email sent
- **bounce**: Bounced email
- **complaint**: Spam complaint
- **delivery**: Successfully delivered
- **open**: Email opened (requires tracking)
- **click**: Link clicked (requires tracking)
- **reject**: Rejected by SES
- **renderingFailure**: Template rendering failed

---

### 5. Add Event Destination (SNS)

```bash
aws ses put-configuration-set-event-destination \
    --configuration-set-name my-config-set \
    --event-destination file://sns-destination.json
```

**sns-destination.json:**
```json
{
    "Name": "SNSDestination",
    "Enabled": true,
    "MatchingEventTypes": ["bounce", "complaint"],
    "SNSDestination": {
        "TopicARN": "arn:aws:sns:us-east-1:123456789012:ses-events"
    }
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Receive email events via SNS notifications.

**When to Use:** Real-time event processing, integrations.

---

### 6. Add Event Destination (Kinesis Firehose)

```bash
aws ses put-configuration-set-event-destination \
    --configuration-set-name my-config-set \
    --event-destination file://firehose-destination.json
```

**firehose-destination.json:**
```json
{
    "Name": "FirehoseDestination",
    "Enabled": true,
    "MatchingEventTypes": ["send", "bounce", "complaint", "delivery"],
    "KinesisFirehoseDestination": {
        "IAMRoleARN": "arn:aws:iam::123456789012:role/SESFirehoseRole",
        "DeliveryStreamARN": "arn:aws:firehose:us-east-1:123456789012:deliverystream/ses-events"
    }
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Stream events to S3 for long-term storage and analysis.

**When to Use:** Data warehousing, compliance, analytics.

---

### 7. Enable Reputation Metrics

```bash
aws ses put-configuration-set-reputation-options \
    --configuration-set-name my-config-set \
    --reputation-metrics-enabled
```

**Expected Output:**
```
(No output on success)
```

**Why:** Track bounce and complaint rates.

**When to Use:** Monitoring sender reputation, production deployments.

---

### 8. Enable Open/Click Tracking

```bash
aws ses put-configuration-set-tracking-options \
    --configuration-set-name my-config-set \
    --custom-redirect-domain track.example.com
```

**Expected Output:**
```
(No output on success)
```

**Why:** Track email opens and link clicks.

**When to Use:** Email marketing, engagement analytics.

**Note:** Requires custom domain for tracking links.

---

### 9. Delete Configuration Set

```bash
aws ses delete-configuration-set --configuration-set-name my-config-set
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove configuration set.

**When to Use:** Cleanup, decommissioning.

---

## Email Receiving

### 1. Create Receipt Rule Set

```bash
aws ses create-receipt-rule-set --rule-set-name my-rule-set
```

**Expected Output:**
```
(No output on success)
```

**Why:** Create rule set for incoming email processing.

**When to Use:** Email receiving, automated processing.

---

### 2. Set Active Receipt Rule Set

```bash
aws ses set-active-receipt-rule-set --rule-set-name my-rule-set
```

**Expected Output:**
```
(No output on success)
```

**Why:** Activate rule set for receiving emails.

**When to Use:** Enabling email receiving.

**Note:** Only one active rule set at a time.

---

### 3. Create Receipt Rule (Save to S3)

```bash
aws ses create-receipt-rule \
    --rule-set-name my-rule-set \
    --rule file://receipt-rule.json
```

**receipt-rule.json:**
```json
{
    "Name": "SaveToS3",
    "Enabled": true,
    "Recipients": ["support@example.com"],
    "Actions": [
        {
            "S3Action": {
                "BucketName": "my-email-bucket",
                "ObjectKeyPrefix": "emails/"
            }
        }
    ]
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Save incoming emails to S3.

**When to Use:** Email archiving, processing with Lambda.

---

### 4. Create Receipt Rule (Invoke Lambda)

```bash
aws ses create-receipt-rule \
    --rule-set-name my-rule-set \
    --rule file://lambda-rule.json
```

**lambda-rule.json:**
```json
{
    "Name": "ProcessWithLambda",
    "Enabled": true,
    "Recipients": ["orders@example.com"],
    "Actions": [
        {
            "LambdaAction": {
                "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:ProcessEmail",
                "InvocationType": "Event"
            }
        }
    ]
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Process incoming emails with Lambda.

**When to Use:** Email parsing, order processing, automated responses.

---

### 5. Describe Receipt Rule

```bash
aws ses describe-receipt-rule \
    --rule-set-name my-rule-set \
    --rule-name SaveToS3
```

**Expected Output:**
```json
{
    "Rule": {
        "Name": "SaveToS3",
        "Enabled": true,
        "Recipients": ["support@example.com"],
        "Actions": [
            {
                "S3Action": {
                    "BucketName": "my-email-bucket",
                    "ObjectKeyPrefix": "emails/"
                }
            }
        ]
    }
}
```

**Why:** View receipt rule configuration.

**When to Use:** Reviewing rules, troubleshooting.

---

### 6. Verify Domain for Receiving (MX Record)

```bash
aws ses verify-domain-identity --domain example.com
```

**Add MX record to DNS:**
```
Type: MX
Name: example.com
Value: 10 inbound-smtp.us-east-1.amazonaws.com
Priority: 10
```

**Why:** Configure domain to receive emails via SES.

**When to Use:** Setting up email receiving.

---

## Suppression Lists

### 1. Get Account Suppression Attributes

```bash
aws sesv2 get-account
```

**Expected Output:**
```json
{
    "EnforcementStatus": "ENABLED",
    "ProductionAccessEnabled": false,
    "SendingEnabled": true,
    "SuppressionAttributes": {
        "SuppressedReasons": ["BOUNCE", "COMPLAINT"]
    }
}
```

**Why:** View account-level suppression settings.

**When to Use:** Checking suppression configuration.

---

### 2. Add Email to Suppression List

```bash
aws sesv2 put-suppressed-destination \
    --email-address baduser@example.com \
    --reason COMPLAINT
```

**Expected Output:**
```
(No output on success)
```

**Why:** Prevent sending to specific address.

**When to Use:** Manual suppression, compliance.

**Reasons:**
- **BOUNCE**: Hard bounce
- **COMPLAINT**: Spam complaint

---

### 3. List Suppressed Destinations

```bash
aws sesv2 list-suppressed-destinations
```

**Expected Output:**
```json
{
    "SuppressedDestinationSummaries": [
        {
            "EmailAddress": "baduser@example.com",
            "Reason": "COMPLAINT",
            "LastUpdateTime": "2025-11-17T10:00:00+00:00"
        }
    ]
}
```

**Why:** View all suppressed email addresses.

**When to Use:** Auditing suppression list, cleanup.

---

### 4. Remove Email from Suppression List

```bash
aws sesv2 delete-suppressed-destination --email-address baduser@example.com
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove address from suppression list.

**When to Use:** Re-enabling delivery after resolution.

---

## Reputation & Deliverability

### 1. Get Send Statistics

```bash
aws ses get-send-statistics
```

**Expected Output:**
```json
{
    "SendDataPoints": [
        {
            "Timestamp": "2025-11-17T10:00:00+00:00",
            "DeliveryAttempts": 100,
            "Bounces": 2,
            "Complaints": 0,
            "Rejects": 1
        },
        {
            "Timestamp": "2025-11-17T09:45:00+00:00",
            "DeliveryAttempts": 150,
            "Bounces": 1,
            "Complaints": 0,
            "Rejects": 0
        }
    ]
}
```

**Why:** Monitor sending statistics and reputation metrics.

**When to Use:** Performance monitoring, troubleshooting deliverability.

**Metrics:**
- **DeliveryAttempts**: Total send attempts
- **Bounces**: Failed deliveries
- **Complaints**: Spam complaints
- **Rejects**: Rejected by SES (invalid, suppressed, etc.)

---

### 2. Get Send Quota

```bash
aws ses get-send-quota
```

**Expected Output:**
```json
{
    "Max24HourSend": 200.0,
    "MaxSendRate": 1.0,
    "SentLast24Hours": 45.0
}
```

**Why:** Check sending limits.

**When to Use:** Capacity planning, troubleshooting sending failures.

**Limits:**
- **Max24HourSend**: Daily quota
- **MaxSendRate**: Emails per second
- **SentLast24Hours**: Usage today

---

### 3. Request Production Access

```bash
# Request via AWS Support Console or CLI
aws support create-case \
    --subject "SES Production Access Request" \
    --service-code ses \
    --category-code production-access \
    --severity-code low \
    --communication-body "Please enable production access for SES..."
```

**Why:** Move from sandbox to production.

**When to Use:** Production deployment.

**Requirements:**
- Describe use case
- Bounce/complaint handling process
- Expected sending volume

---

### 4. Enable Dedicated IP Pool (Advanced)

```bash
aws sesv2 create-dedicated-ip-pool --pool-name my-ip-pool
```

**Expected Output:**
```
(No output on success)
```

**Why:** Isolate sending reputation with dedicated IPs.

**When to Use:** High-volume senders, reputation management.

**Note:** Additional cost for dedicated IPs.

---

## SMTP Credentials

### 1. Create SMTP Credentials (IAM User)

```bash
# Create IAM user
aws iam create-user --user-name ses-smtp-user

# Attach policy
aws iam attach-user-policy \
    --user-name ses-smtp-user \
    --policy-arn arn:aws:iam::aws:policy/AmazonSESFullAccess

# Create access key
aws iam create-access-key --user-name ses-smtp-user
```

**Expected Output:**
```json
{
    "AccessKey": {
        "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
        "Status": "Active",
        "CreateDate": "2025-11-17T10:00:00+00:00"
    }
}
```

**Why:** Generate credentials for SMTP sending.

**When to Use:** Integrating applications via SMTP.

**Convert to SMTP password:** Use AWS SES SMTP password calculator or SDK.

**SMTP Endpoints:**
- **us-east-1**: email-smtp.us-east-1.amazonaws.com:587 (TLS)
- **us-west-2**: email-smtp.us-west-2.amazonaws.com:587 (TLS)
- Port 25, 465, 587, 2465, 2587

---

### 2. Send Email via SMTP (Python Example)

```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# SMTP settings
smtp_host = "email-smtp.us-east-1.amazonaws.com"
smtp_port = 587
smtp_username = "AKIAIOSFODNN7EXAMPLE"
smtp_password = "SMTP_PASSWORD_HERE"

# Email content
sender = "noreply@example.com"
recipient = "john@example.com"
subject = "Test Email via SMTP"

msg = MIMEMultipart()
msg['From'] = sender
msg['To'] = recipient
msg['Subject'] = subject
msg.attach(MIMEText("This is a test email sent via SMTP.", 'plain'))

# Send email
try:
    server = smtplib.SMTP(smtp_host, smtp_port)
    server.starttls()
    server.login(smtp_username, smtp_password)
    server.sendmail(sender, recipient, msg.as_string())
    server.quit()
    print("Email sent successfully!")
except Exception as e:
    print(f"Error: {e}")
```

**Why:** Send emails via SMTP protocol.

**When to Use:** Legacy applications, SMTP-only systems.

---

## Event Publishing

### 1. Publish Events to CloudWatch

```bash
# Already covered in Configuration Sets section
# Events automatically published when configured
```

**View metrics in CloudWatch:**
```bash
aws cloudwatch get-metric-statistics \
    --namespace AWS/SES \
    --metric-name Bounce \
    --dimensions Name=ses:configuration-set,Value=my-config-set \
    --start-time 2025-11-17T00:00:00Z \
    --end-time 2025-11-17T23:59:59Z \
    --period 3600 \
    --statistics Sum
```

**Why:** Monitor email events in CloudWatch.

**When to Use:** Centralized monitoring, alerting.

---

### 2. Create CloudWatch Alarm for Bounces

```bash
aws cloudwatch put-metric-alarm \
    --alarm-name high-bounce-rate \
    --alarm-description "Alert when bounce rate exceeds 5%" \
    --metric-name Bounce \
    --namespace AWS/SES \
    --statistic Sum \
    --period 3600 \
    --threshold 50 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:ses-alerts
```

**Expected Output:**
```
(No output on success)
```

**Why:** Get alerted when bounce rate is high.

**When to Use:** Proactive reputation management.

---

## Testing & Sandbox

### 1. Use SES Mailbox Simulator

```bash
# Success
aws ses send-email --from noreply@example.com --to success@simulator.amazonses.com --subject "Test" --text "Test"

# Bounce
aws ses send-email --from noreply@example.com --to bounce@simulator.amazonses.com --subject "Test" --text "Test"

# Complaint
aws ses send-email --from noreply@example.com --to complaint@simulator.amazonses.com --subject "Test" --text "Test"

# Out of office
aws ses send-email --from noreply@example.com --to ooto@simulator.amazonses.com --subject "Test" --text "Test"
```

**Why:** Test without affecting reputation or using quota.

**When to Use:** Development, testing bounce/complaint handling.

**Mailbox Simulator Addresses:**
- **success@simulator.amazonses.com**: Successful delivery
- **bounce@simulator.amazonses.com**: Hard bounce
- **complaint@simulator.amazonses.com**: Spam complaint
- **ooto@simulator.amazonses.com**: Out of office
- **suppressionlist@simulator.amazonses.com**: Suppression list

---

### 2. Test Email Content

```bash
# Send test email to your verified address
aws ses send-email \
    --from noreply@example.com \
    --to your-verified-email@example.com \
    --subject "Content Test" \
    --html file://test-email.html \
    --text file://test-email.txt
```

**Why:** Preview email rendering in actual email client.

**When to Use:** Design testing, QA.

---

## Best Practices

### Deliverability
1. **Verify domains (not just emails)** - Better reputation
2. **Set up DKIM** - Email authentication
3. **Use custom MAIL FROM** - DMARC alignment
4. **Warm up IP addresses** - Gradual volume increase
5. **Monitor bounce/complaint rates** - Keep below thresholds
6. **Handle bounces** - Remove invalid addresses
7. **Double opt-in** - Confirm subscriptions
8. **Clean mailing lists** - Remove inactive users

### Security
1. **Use IAM roles** - Not long-term credentials
2. **Rotate SMTP credentials** - Regular rotation
3. **Enable TLS** - Encrypted transmission
4. **Validate input** - Prevent header injection
5. **Rate limiting** - Prevent abuse
6. **Monitor suspicious activity** - CloudWatch alarms

### Cost Optimization
1. **Use bulk sending** - More efficient
2. **Remove bounced addresses** - Don't waste quota
3. **Use templates** - Reduce rendering costs
4. **Monitor usage** - Set billing alerts
5. **Optimize attachments** - Compress files

### Development
1. **Use configuration sets** - Track everything
2. **Test with simulator** - Don't use real addresses
3. **Implement retry logic** - Handle throttling
4. **Log MessageId** - Track delivery
5. **Handle bounces/complaints** - Automated processing
6. **Use SESv2 API** - Newer features

### Compliance
1. **Include unsubscribe link** - Required for marketing
2. **Honor unsubscribe requests** - Immediately
3. **Provide physical address** - CAN-SPAM compliance
4. **Accurate FROM address** - No deception
5. **Clear subject lines** - Describe content
6. **GDPR compliance** - Data protection

---

## Troubleshooting

### Email Not Delivered
- Check send statistics for bounces/rejects
- Verify recipient address
- Check sending quota
- Review suppression list
- Verify sender identity

### High Bounce Rate
- Clean mailing list
- Use double opt-in
- Monitor bounce types (hard vs. soft)
- Remove invalid addresses
- Check DNS configuration

### Low Open Rates
- Improve subject lines
- Test send times
- Segment audience
- Check spam folder placement
- Review email content/design

### Throttling Errors
- Check sending quota and rate
- Implement exponential backoff
- Request quota increase
- Use bulk sending API

### Domain Not Verified
- Check DNS propagation (can take 72 hours)
- Verify DNS records match exactly
- Use correct record type (TXT for domain, CNAME for DKIM)
- Check domain spelling

---

## Quick Reference

```bash
# Verification
aws ses verify-email-identity --email-address EMAIL
aws ses verify-domain-identity --domain DOMAIN
aws ses verify-domain-dkim --domain DOMAIN

# Sending
aws ses send-email --from FROM --to TO --subject SUBJECT --text TEXT
aws ses send-templated-email --source FROM --destination ToAddresses=TO --template TEMPLATE --template-data DATA

# Templates
aws ses create-template --template file://template.json
aws ses list-templates
aws ses delete-template --template-name NAME

# Configuration Sets
aws ses create-configuration-set --configuration-set Name=NAME
aws ses list-configuration-sets

# Statistics
aws ses get-send-quota
aws ses get-send-statistics

# Suppression
aws sesv2 list-suppressed-destinations
aws sesv2 put-suppressed-destination --email-address EMAIL --reason REASON
```

---

## Summary

SES provides reliable, scalable email sending and receiving capabilities. Master these commands to:
- Send transactional and marketing emails
- Monitor deliverability and reputation
- Implement email receiving and processing
- Track email events and engagement
- Maintain compliance

Practice sending emails, creating templates, and monitoring metrics. SES's high deliverability and low cost make it ideal for production email systems.

For LocalStack: add `--endpoint-url=http://localhost:4566` to commands.

Happy emailing!
