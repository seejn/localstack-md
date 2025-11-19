# AWS S3 (Simple Storage Service) - Complete Practical Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Key Concepts](#key-concepts)
3. [Bucket Operations](#bucket-operations)
4. [Object Operations](#object-operations)
5. [Access Control & Permissions](#access-control--permissions)
6. [Versioning](#versioning)
7. [Lifecycle Management](#lifecycle-management)
8. [Static Website Hosting](#static-website-hosting)
9. [CORS Configuration](#cors-configuration)
10. [Encryption](#encryption)
11. [Logging & Monitoring](#logging--monitoring)
12. [Replication](#replication)
13. [Advanced Features](#advanced-features)
14. [Best Practices](#best-practices)

---

## Introduction

Amazon S3 is an object storage service offering industry-leading scalability, data availability, security, and performance. It's designed to store and retrieve any amount of data from anywhere.

**Use Cases:**
- Backup and disaster recovery
- Data lakes and big data analytics
- Static website hosting
- Content distribution
- Application data storage
- Archive and compliance

---

## Key Concepts

### Buckets
- Containers for objects stored in S3
- Globally unique names (across all AWS accounts)
- Regional resources (data stays in chosen region)
- Can contain unlimited objects

### Objects
- Files stored in S3
- Consist of: Key (name), Value (data), Metadata, Version ID
- Size: 0 bytes to 5TB
- Key is the full path within bucket

### Storage Classes
- **Standard**: Frequent access, low latency
- **Intelligent-Tiering**: Automatic cost optimization
- **Standard-IA**: Infrequent access, lower cost
- **One Zone-IA**: Single AZ, infrequent access
- **Glacier**: Long-term archive, minutes to hours retrieval
- **Glacier Deep Archive**: Lowest cost, 12-hour retrieval

---

## Bucket Operations

### 1. Create a Bucket

```bash
aws s3 mb s3://my-learning-bucket --region us-east-1
```

**Expected Output:**
```
make_bucket: my-learning-bucket
```

**Why:** Creates a new S3 bucket. Buckets are the top-level containers for your data.

**When to Use:** Starting a new project, creating isolated storage for different applications/environments.

**LocalStack:**
```bash
aws s3 mb s3://my-learning-bucket --endpoint-url=http://localhost:4566
```

---

### 2. List All Buckets

```bash
aws s3 ls
```

**Expected Output:**
```
2025-11-17 10:30:45 my-learning-bucket
2025-11-15 08:20:12 another-bucket
2025-11-10 14:15:30 data-backup-bucket
```

**Why:** View all buckets in your account to manage resources and verify bucket creation.

**When to Use:** Auditing resources, checking bucket existence, general overview of storage.

---

### 3. List Bucket Contents

```bash
aws s3 ls s3://my-learning-bucket
```

**Expected Output:**
```
2025-11-17 10:35:20    1024 document.txt
2025-11-17 10:36:45   51200 image.png
                       PRE logs/
```

**Why:** See what objects are stored in a bucket. PRE indicates a prefix/folder.

**When to Use:** Verifying uploads, browsing content, checking file existence.

**List with Prefix (folder):**
```bash
aws s3 ls s3://my-learning-bucket/logs/
```

**Recursive Listing:**
```bash
aws s3 ls s3://my-learning-bucket --recursive
```

**Output:**
```
2025-11-17 10:35:20    1024 document.txt
2025-11-17 10:36:45   51200 image.png
2025-11-17 10:37:10    2048 logs/app.log
2025-11-17 10:37:25    4096 logs/error.log
```

---

### 4. Get Bucket Location

```bash
aws s3api get-bucket-location --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "LocationConstraint": "us-east-1"
}
```

**Why:** Verify which region your bucket is in (affects latency and compliance).

**When to Use:** Checking data residency, troubleshooting region-specific issues.

---

### 5. Delete a Bucket

```bash
aws s3 rb s3://my-learning-bucket
```

**Expected Output:**
```
remove_bucket: my-learning-bucket
```

**Why:** Remove an empty bucket. Bucket must be empty before deletion.

**When to Use:** Cleaning up unused resources, project decommissioning.

**Force Delete (with objects):**
```bash
aws s3 rb s3://my-learning-bucket --force
```

**Note:** This deletes all objects first, then the bucket.

---

## Object Operations

### 1. Upload a File (Put Object)

```bash
aws s3 cp myfile.txt s3://my-learning-bucket/
```

**Expected Output:**
```
upload: ./myfile.txt to s3://my-learning-bucket/myfile.txt
```

**Why:** Store data in S3. Basic upload operation.

**When to Use:** Backing up files, storing application data, sharing files.

**Upload to Specific Path:**
```bash
aws s3 cp myfile.txt s3://my-learning-bucket/documents/myfile.txt
```

**Upload with Storage Class:**
```bash
aws s3 cp myfile.txt s3://my-learning-bucket/ --storage-class STANDARD_IA
```

---

### 2. Upload Multiple Files (Sync)

```bash
aws s3 sync ./local-folder s3://my-learning-bucket/remote-folder/
```

**Expected Output:**
```
upload: local-folder/file1.txt to s3://my-learning-bucket/remote-folder/file1.txt
upload: local-folder/file2.txt to s3://my-learning-bucket/remote-folder/file2.txt
upload: local-folder/subfolder/file3.txt to s3://my-learning-bucket/remote-folder/subfolder/file3.txt
```

**Why:** Efficiently upload entire directories. Only uploads new/changed files.

**When to Use:** Deploying static websites, backing up directories, batch uploads.

**Sync with Delete (mirror):**
```bash
aws s3 sync ./local-folder s3://my-learning-bucket/remote-folder/ --delete
```

**Note:** --delete removes files in S3 that don't exist locally.

---

### 3. Download a File (Get Object)

```bash
aws s3 cp s3://my-learning-bucket/myfile.txt ./downloaded-file.txt
```

**Expected Output:**
```
download: s3://my-learning-bucket/myfile.txt to ./downloaded-file.txt
```

**Why:** Retrieve data from S3 to local storage.

**When to Use:** Restoring backups, downloading user uploads, accessing stored data.

**Download Entire Folder:**
```bash
aws s3 cp s3://my-learning-bucket/documents/ ./local-docs/ --recursive
```

---

### 4. Download with Sync

```bash
aws s3 sync s3://my-learning-bucket/remote-folder/ ./local-folder/
```

**Expected Output:**
```
download: s3://my-learning-bucket/remote-folder/file1.txt to local-folder/file1.txt
download: s3://my-learning-bucket/remote-folder/file2.txt to local-folder/file2.txt
```

**Why:** Download directories efficiently. Only downloads new/changed files.

**When to Use:** Keeping local copies synchronized, distributed systems.

---

### 5. Move Files

```bash
aws s3 mv s3://my-learning-bucket/oldpath/file.txt s3://my-learning-bucket/newpath/file.txt
```

**Expected Output:**
```
move: s3://my-learning-bucket/oldpath/file.txt to s3://my-learning-bucket/newpath/file.txt
```

**Why:** Relocate objects within S3 (copy + delete original).

**When to Use:** Reorganizing data, renaming files, moving to different storage class.

**Move Local to S3:**
```bash
aws s3 mv myfile.txt s3://my-learning-bucket/
```

---

### 6. Delete an Object

```bash
aws s3 rm s3://my-learning-bucket/myfile.txt
```

**Expected Output:**
```
delete: s3://my-learning-bucket/myfile.txt
```

**Why:** Remove unwanted objects to free space and reduce costs.

**When to Use:** Cleaning up old data, removing temporary files.

**Delete Multiple Objects:**
```bash
aws s3 rm s3://my-learning-bucket/documents/ --recursive
```

---

### 7. Get Object Metadata

```bash
aws s3api head-object --bucket my-learning-bucket --key myfile.txt
```

**Expected Output:**
```json
{
    "AcceptRanges": "bytes",
    "LastModified": "2025-11-17T10:35:20+00:00",
    "ContentLength": 1024,
    "ETag": "\"5d41402abc4b2a76b9719d911017c592\"",
    "ContentType": "text/plain",
    "ServerSideEncryption": "AES256",
    "Metadata": {}
}
```

**Why:** Get object details without downloading the entire file.

**When to Use:** Checking file size, verifying existence, reading custom metadata.

---

### 8. Put Object with Metadata

```bash
aws s3api put-object --bucket my-learning-bucket --key myfile.txt --body myfile.txt --metadata author=john,department=engineering
```

**Expected Output:**
```json
{
    "ETag": "\"5d41402abc4b2a76b9719d911017c592\"",
    "ServerSideEncryption": "AES256"
}
```

**Why:** Store custom metadata with objects for categorization and management.

**When to Use:** Tagging user uploads, storing processing status, audit trails.

---

### 9. Generate Presigned URL

```bash
aws s3 presign s3://my-learning-bucket/myfile.txt --expires-in 3600
```

**Expected Output:**
```
https://my-learning-bucket.s3.amazonaws.com/myfile.txt?AWSAccessKeyId=AKIAIOSFODNN7EXAMPLE&Expires=1700227200&Signature=ABC123...
```

**Why:** Create temporary URLs for secure access without AWS credentials.

**When to Use:** Sharing private files, temporary downloads, client-side uploads.

**Custom Expiry (7 days = 604800 seconds):**
```bash
aws s3 presign s3://my-learning-bucket/myfile.txt --expires-in 604800
```

---

### 10. Copy Objects Between Buckets

```bash
aws s3 cp s3://source-bucket/file.txt s3://destination-bucket/file.txt
```

**Expected Output:**
```
copy: s3://source-bucket/file.txt to s3://destination-bucket/file.txt
```

**Why:** Duplicate data across buckets for backups or multi-region deployments.

**When to Use:** Cross-region replication (manual), backups, data migration.

---

## Access Control & Permissions

### 1. Set Bucket ACL (Canned ACL)

```bash
aws s3api put-bucket-acl --bucket my-learning-bucket --acl private
```

**Expected Output:**
```
(No output on success)
```

**Why:** Control bucket-level access. Options: private, public-read, public-read-write, authenticated-read.

**When to Use:** Making buckets public/private, basic access control.

**Public Read:**
```bash
aws s3api put-bucket-acl --bucket my-learning-bucket --acl public-read
```

---

### 2. Get Bucket ACL

```bash
aws s3api get-bucket-acl --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "Owner": {
        "DisplayName": "account-owner",
        "ID": "75aa57f09aa0c8caeab4f8c24e99d10f8e7faeebf76c078efc7c6caea54ba06a"
    },
    "Grants": [
        {
            "Grantee": {
                "Type": "CanonicalUser",
                "ID": "75aa57f09aa0c8caeab4f8c24e99d10f8e7faeebf76c078efc7c6caea54ba06a"
            },
            "Permission": "FULL_CONTROL"
        }
    ]
}
```

**Why:** View current access control settings for auditing and verification.

**When to Use:** Troubleshooting access issues, security audits.

---

### 3. Set Object ACL

```bash
aws s3api put-object-acl --bucket my-learning-bucket --key myfile.txt --acl public-read
```

**Expected Output:**
```
(No output on success)
```

**Why:** Control individual object access, independent of bucket ACL.

**When to Use:** Making specific files public (images, downloads) while keeping bucket private.

---

### 4. Apply Bucket Policy

```bash
aws s3api put-bucket-policy --bucket my-learning-bucket --policy file://bucket-policy.json
```

**bucket-policy.json:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-learning-bucket/*"
        }
    ]
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Fine-grained access control using JSON policies. More powerful than ACLs.

**When to Use:** Complex access patterns, IP restrictions, conditional access, public websites.

---

### 5. Get Bucket Policy

```bash
aws s3api get-bucket-policy --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Sid\":\"PublicReadGetObject\",\"Effect\":\"Allow\",\"Principal\":\"*\",\"Action\":\"s3:GetObject\",\"Resource\":\"arn:aws:s3:::my-learning-bucket/*\"}]}"
}
```

**Why:** View current bucket policy for review and auditing.

**When to Use:** Debugging access issues, security audits, policy documentation.

---

### 6. Delete Bucket Policy

```bash
aws s3api delete-bucket-policy --bucket my-learning-bucket
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove bucket policy, reverting to default (ACL-based) access control.

**When to Use:** Simplifying access control, troubleshooting.

---

### 7. Block Public Access

```bash
aws s3api put-public-access-block --bucket my-learning-bucket --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

**Expected Output:**
```
(No output on success)
```

**Why:** Security feature to prevent accidental public exposure.

**When to Use:** Securing sensitive data buckets, compliance requirements.

---

### 8. Get Public Access Block Configuration

```bash
aws s3api get-public-access-block --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "PublicAccessBlockConfiguration": {
        "BlockPublicAcls": true,
        "IgnorePublicAcls": true,
        "BlockPublicPolicy": true,
        "RestrictPublicBuckets": true
    }
}
```

**Why:** Verify public access restrictions for security compliance.

**When to Use:** Security audits, verifying configurations.

---

## Versioning

### 1. Enable Versioning

```bash
aws s3api put-bucket-versioning --bucket my-learning-bucket --versioning-configuration Status=Enabled
```

**Expected Output:**
```
(No output on success)
```

**Why:** Keep multiple versions of objects. Protects against accidental deletion and overwrites.

**When to Use:** Important data, compliance requirements, rollback capability.

---

### 2. Get Versioning Status

```bash
aws s3api get-bucket-versioning --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "Status": "Enabled"
}
```

**Why:** Verify versioning configuration.

**When to Use:** Auditing, troubleshooting, confirming settings.

---

### 3. List Object Versions

```bash
aws s3api list-object-versions --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "Versions": [
        {
            "Key": "myfile.txt",
            "VersionId": "3/L4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY+MTRCxf3vjVBH40Nr8X8gdRQBpUMLUo",
            "IsLatest": true,
            "LastModified": "2025-11-17T10:40:00+00:00",
            "Size": 1024
        },
        {
            "Key": "myfile.txt",
            "VersionId": "QUpfdndhfd8438MNFDN4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY",
            "IsLatest": false,
            "LastModified": "2025-11-17T10:35:00+00:00",
            "Size": 512
        }
    ]
}
```

**Why:** See all versions of objects for recovery or audit purposes.

**When to Use:** Recovering old versions, auditing changes.

---

### 4. Download Specific Version

```bash
aws s3api get-object --bucket my-learning-bucket --key myfile.txt --version-id QUpfdndhfd8438MNFDN4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY downloaded-version.txt
```

**Expected Output:**
```json
{
    "AcceptRanges": "bytes",
    "LastModified": "2025-11-17T10:35:00+00:00",
    "ContentLength": 512,
    "VersionId": "QUpfdndhfd8438MNFDN4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY"
}
```

**Why:** Retrieve specific version of an object.

**When to Use:** Recovering old data, comparing versions.

---

### 5. Delete Specific Version

```bash
aws s3api delete-object --bucket my-learning-bucket --key myfile.txt --version-id QUpfdndhfd8438MNFDN4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY
```

**Expected Output:**
```json
{
    "DeleteMarker": false,
    "VersionId": "QUpfdndhfd8438MNFDN4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY"
}
```

**Why:** Permanently remove specific version (doesn't affect other versions).

**When to Use:** Cleanup, removing sensitive data from history.

---

### 6. Suspend Versioning

```bash
aws s3api put-bucket-versioning --bucket my-learning-bucket --versioning-configuration Status=Suspended
```

**Expected Output:**
```
(No output on success)
```

**Why:** Stop creating new versions. Existing versions remain.

**When to Use:** Reducing costs, temporary suspension during bulk operations.

---

## Lifecycle Management

### 1. Put Lifecycle Configuration

```bash
aws s3api put-bucket-lifecycle-configuration --bucket my-learning-bucket --lifecycle-configuration file://lifecycle-policy.json
```

**lifecycle-policy.json:**
```json
{
    "Rules": [
        {
            "Id": "Move to IA after 30 days",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "documents/"
            },
            "Transitions": [
                {
                    "Days": 30,
                    "StorageClass": "STANDARD_IA"
                },
                {
                    "Days": 90,
                    "StorageClass": "GLACIER"
                }
            ]
        },
        {
            "Id": "Delete old logs",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "logs/"
            },
            "Expiration": {
                "Days": 365
            }
        }
    ]
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Automate storage class transitions and deletions to optimize costs.

**When to Use:** Cost optimization, automatic cleanup, compliance retention.

---

### 2. Get Lifecycle Configuration

```bash
aws s3api get-bucket-lifecycle-configuration --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "Rules": [
        {
            "Id": "Move to IA after 30 days",
            "Status": "Enabled",
            "Filter": {
                "Prefix": "documents/"
            },
            "Transitions": [
                {
                    "Days": 30,
                    "StorageClass": "STANDARD_IA"
                }
            ]
        }
    ]
}
```

**Why:** Review current lifecycle rules.

**When to Use:** Auditing, troubleshooting, documentation.

---

### 3. Delete Lifecycle Configuration

```bash
aws s3api delete-bucket-lifecycle --bucket my-learning-bucket
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove all lifecycle rules.

**When to Use:** Changing strategy, troubleshooting.

---

## Static Website Hosting

### 1. Enable Website Hosting

```bash
aws s3api put-bucket-website --bucket my-learning-bucket --website-configuration file://website-config.json
```

**website-config.json:**
```json
{
    "IndexDocument": {
        "Suffix": "index.html"
    },
    "ErrorDocument": {
        "Key": "error.html"
    }
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Host static websites directly from S3.

**When to Use:** Simple websites, SPAs (Single Page Apps), documentation sites.

**Website URL format:** `http://my-learning-bucket.s3-website-us-east-1.amazonaws.com`

---

### 2. Get Website Configuration

```bash
aws s3api get-bucket-website --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "IndexDocument": {
        "Suffix": "index.html"
    },
    "ErrorDocument": {
        "Key": "error.html"
    }
}
```

**Why:** Verify website configuration.

**When to Use:** Troubleshooting, auditing.

---

### 3. Delete Website Configuration

```bash
aws s3api delete-bucket-website --bucket my-learning-bucket
```

**Expected Output:**
```
(No output on success)
```

**Why:** Disable website hosting.

**When to Use:** Migrating to CloudFront, decommissioning site.

---

## CORS Configuration

### 1. Put CORS Configuration

```bash
aws s3api put-bucket-cors --bucket my-learning-bucket --cors-configuration file://cors-config.json
```

**cors-config.json:**
```json
{
    "CORSRules": [
        {
            "AllowedOrigins": ["https://example.com"],
            "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
            "AllowedHeaders": ["*"],
            "ExposeHeaders": ["ETag"],
            "MaxAgeSeconds": 3000
        }
    ]
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Allow cross-origin requests from web browsers.

**When to Use:** Web applications accessing S3 from different domains.

---

### 2. Get CORS Configuration

```bash
aws s3api get-bucket-cors --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "CORSRules": [
        {
            "AllowedOrigins": ["https://example.com"],
            "AllowedMethods": ["GET", "PUT", "POST"],
            "AllowedHeaders": ["*"]
        }
    ]
}
```

**Why:** View current CORS rules.

**When to Use:** Troubleshooting CORS errors, auditing.

---

### 3. Delete CORS Configuration

```bash
aws s3api delete-bucket-cors --bucket my-learning-bucket
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove CORS configuration.

**When to Use:** Changing security posture, troubleshooting.

---

## Encryption

### 1. Enable Default Encryption (AES256)

```bash
aws s3api put-bucket-encryption --bucket my-learning-bucket --server-side-encryption-configuration file://encryption-config.json
```

**encryption-config.json:**
```json
{
    "Rules": [
        {
            "ApplyServerSideEncryptionByDefault": {
                "SSEAlgorithm": "AES256"
            },
            "BucketKeyEnabled": false
        }
    ]
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Encrypt all objects at rest automatically.

**When to Use:** Security compliance, protecting sensitive data.

---

### 2. Enable KMS Encryption

```bash
aws s3api put-bucket-encryption --bucket my-learning-bucket --server-side-encryption-configuration file://kms-encryption-config.json
```

**kms-encryption-config.json:**
```json
{
    "Rules": [
        {
            "ApplyServerSideEncryptionByDefault": {
                "SSEAlgorithm": "aws:kms",
                "KMSMasterKeyID": "arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012"
            },
            "BucketKeyEnabled": true
        }
    ]
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Use AWS KMS for encryption with key management and audit trails.

**When to Use:** Enhanced security, compliance, key rotation requirements.

---

### 3. Get Encryption Configuration

```bash
aws s3api get-bucket-encryption --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "ServerSideEncryptionConfiguration": {
        "Rules": [
            {
                "ApplyServerSideEncryptionByDefault": {
                    "SSEAlgorithm": "AES256"
                }
            }
        ]
    }
}
```

**Why:** Verify encryption settings.

**When to Use:** Security audits, compliance checks.

---

### 4. Upload with Server-Side Encryption

```bash
aws s3 cp myfile.txt s3://my-learning-bucket/ --server-side-encryption AES256
```

**Expected Output:**
```
upload: ./myfile.txt to s3://my-learning-bucket/myfile.txt
```

**Why:** Explicitly encrypt individual objects.

**When to Use:** When bucket default encryption isn't configured.

---

## Logging & Monitoring

### 1. Enable Server Access Logging

```bash
aws s3api put-bucket-logging --bucket my-learning-bucket --bucket-logging-status file://logging-config.json
```

**logging-config.json:**
```json
{
    "LoggingEnabled": {
        "TargetBucket": "my-logs-bucket",
        "TargetPrefix": "my-learning-bucket-logs/"
    }
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Track all requests made to bucket for security and analysis.

**When to Use:** Security monitoring, usage analysis, compliance.

---

### 2. Get Bucket Logging

```bash
aws s3api get-bucket-logging --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "LoggingEnabled": {
        "TargetBucket": "my-logs-bucket",
        "TargetPrefix": "my-learning-bucket-logs/"
    }
}
```

**Why:** Verify logging configuration.

**When to Use:** Auditing, troubleshooting.

---

### 3. Add Bucket Tags

```bash
aws s3api put-bucket-tagging --bucket my-learning-bucket --tagging file://tags.json
```

**tags.json:**
```json
{
    "TagSet": [
        {
            "Key": "Environment",
            "Value": "Development"
        },
        {
            "Key": "Project",
            "Value": "LearningAWS"
        },
        {
            "Key": "CostCenter",
            "Value": "Engineering"
        }
    ]
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Organize resources, cost allocation, automation.

**When to Use:** Cost tracking, resource management, filtering.

---

### 4. Get Bucket Tags

```bash
aws s3api get-bucket-tagging --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "TagSet": [
        {
            "Key": "Environment",
            "Value": "Development"
        },
        {
            "Key": "Project",
            "Value": "LearningAWS"
        }
    ]
}
```

**Why:** View tags for cost analysis and resource management.

**When to Use:** Cost reports, auditing, automation.

---

## Replication

### 1. Enable Cross-Region Replication (CRR)

```bash
aws s3api put-bucket-replication --bucket my-learning-bucket --replication-configuration file://replication-config.json
```

**replication-config.json:**
```json
{
    "Role": "arn:aws:iam::123456789012:role/s3-replication-role",
    "Rules": [
        {
            "Status": "Enabled",
            "Priority": 1,
            "DeleteMarkerReplication": {
                "Status": "Enabled"
            },
            "Filter": {
                "Prefix": ""
            },
            "Destination": {
                "Bucket": "arn:aws:s3:::my-replica-bucket",
                "ReplicationTime": {
                    "Status": "Enabled",
                    "Time": {
                        "Minutes": 15
                    }
                },
                "Metrics": {
                    "Status": "Enabled"
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

**Why:** Automatically replicate objects to another region for disaster recovery.

**When to Use:** Multi-region redundancy, compliance, lower latency access.

**Prerequisites:**
- Versioning must be enabled on both source and destination
- IAM role with replication permissions

---

### 2. Get Replication Configuration

```bash
aws s3api get-bucket-replication --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "ReplicationConfiguration": {
        "Role": "arn:aws:iam::123456789012:role/s3-replication-role",
        "Rules": [
            {
                "Status": "Enabled",
                "Priority": 1,
                "Filter": {},
                "Destination": {
                    "Bucket": "arn:aws:s3:::my-replica-bucket"
                }
            }
        ]
    }
}
```

**Why:** Verify replication configuration.

**When to Use:** Auditing, troubleshooting replication issues.

---

## Advanced Features

### 1. Enable Requester Pays

```bash
aws s3api put-bucket-request-payment --bucket my-learning-bucket --request-payment-configuration Payer=Requester
```

**Expected Output:**
```
(No output on success)
```

**Why:** Requester pays data transfer and request costs (not storage).

**When to Use:** Sharing large datasets, distributing costs to consumers.

---

### 2. Enable Transfer Acceleration

```bash
aws s3api put-bucket-accelerate-configuration --bucket my-learning-bucket --accelerate-configuration Status=Enabled
```

**Expected Output:**
```
(No output on success)
```

**Why:** Speed up long-distance transfers using CloudFront edge locations.

**When to Use:** Global users, large file uploads, performance optimization.

**Upload with Acceleration:**
```bash
aws s3 cp largefile.zip s3://my-learning-bucket/ --endpoint-url https://s3-accelerate.amazonaws.com
```

---

### 3. Get Accelerate Configuration

```bash
aws s3api get-bucket-accelerate-configuration --bucket my-learning-bucket
```

**Expected Output:**
```json
{
    "Status": "Enabled"
}
```

**Why:** Verify transfer acceleration status.

**When to Use:** Troubleshooting, auditing.

---

### 4. Enable Object Lock (WORM)

```bash
aws s3api put-object-lock-configuration --bucket my-learning-bucket --object-lock-configuration file://object-lock-config.json
```

**object-lock-config.json:**
```json
{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
        "DefaultRetention": {
            "Mode": "COMPLIANCE",
            "Days": 365
        }
    }
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Prevent object deletion for compliance (WORM - Write Once Read Many).

**When to Use:** Regulatory compliance (SEC, HIPAA), legal holds.

**Note:** Must be enabled at bucket creation.

---

### 5. Enable Inventory

```bash
aws s3api put-bucket-inventory-configuration --bucket my-learning-bucket --id daily-inventory --inventory-configuration file://inventory-config.json
```

**inventory-config.json:**
```json
{
    "Destination": {
        "S3BucketDestination": {
            "Bucket": "arn:aws:s3:::my-inventory-bucket",
            "Format": "CSV",
            "Prefix": "inventory-reports/"
        }
    },
    "IsEnabled": true,
    "Filter": {
        "Prefix": ""
    },
    "Id": "daily-inventory",
    "IncludedObjectVersions": "Current",
    "Schedule": {
        "Frequency": "Daily"
    }
}
```

**Expected Output:**
```
(No output on success)
```

**Why:** Generate reports of objects for analytics and lifecycle management.

**When to Use:** Large bucket analysis, cost optimization, compliance auditing.

---

### 6. Multipart Upload (Large Files)

**Initiate Multipart Upload:**
```bash
aws s3api create-multipart-upload --bucket my-learning-bucket --key largefile.zip
```

**Expected Output:**
```json
{
    "Bucket": "my-learning-bucket",
    "Key": "largefile.zip",
    "UploadId": "VXBsb2FkIElEIGZvciA2aWWpbmcncyBteS1tb3ZpZS5tMnRzIHVwbG9hZA"
}
```

**Upload Part:**
```bash
aws s3api upload-part --bucket my-learning-bucket --key largefile.zip --part-number 1 --body part1.bin --upload-id VXBsb2FkIElEIGZvciA2aWWpbmcncyBteS1tb3ZpZS5tMnRzIHVwbG9hZA
```

**Expected Output:**
```json
{
    "ETag": "\"b54357faf0632cce46e942fa68356b38\""
}
```

**Complete Multipart Upload:**
```bash
aws s3api complete-multipart-upload --bucket my-learning-bucket --key largefile.zip --upload-id VXBsb2FkIElEIGZvciA2aWWpbmcncyBteS1tb3ZpZS5tMnRzIHVwbG9hZA --multipart-upload file://parts.json
```

**Why:** Upload files >5GB, resume interrupted uploads, parallel uploads.

**When to Use:** Large files (>100MB recommended), unreliable networks.

---

### 7. Select Object Content (S3 Select)

```bash
aws s3api select-object-content --bucket my-learning-bucket --key data.csv --expression "SELECT * FROM S3Object WHERE age > 30" --expression-type SQL --input-serialization '{"CSV": {"FileHeaderInfo": "USE"}}' --output-serialization '{"CSV": {}}' output.csv
```

**Expected Output:**
```
(Results written to output.csv)
```

**Why:** Query data directly without downloading entire file. Reduces cost and latency.

**When to Use:** Large CSV/JSON files, filtering data, analytics.

---

### 8. Batch Operations

**Create Job:**
```bash
aws s3control create-job --account-id 123456789012 --operation file://operation.json --manifest file://manifest.json --report file://report.json --priority 10 --role-arn arn:aws:iam::123456789012:role/BatchOperationsRole
```

**Why:** Perform bulk operations (copy, tag, restore) on millions of objects.

**When to Use:** Large-scale migrations, bulk tagging, mass deletions.

---

## Best Practices

### Security
1. Enable versioning for critical data
2. Use bucket policies and IAM roles (not IAM users)
3. Enable default encryption (AES256 or KMS)
4. Block public access unless explicitly needed
5. Enable MFA Delete for critical buckets
6. Use VPC endpoints for private access
7. Enable CloudTrail logging
8. Regular access audits

### Cost Optimization
1. Use lifecycle policies to transition to cheaper storage classes
2. Delete incomplete multipart uploads (lifecycle policy)
3. Enable Intelligent-Tiering for unpredictable access patterns
4. Use S3 Select instead of downloading entire files
5. Monitor storage metrics with CloudWatch
6. Compress data before upload
7. Use S3 Inventory to analyze storage

### Performance
1. Use multipart uploads for files >100MB
2. Enable Transfer Acceleration for global users
3. Use CloudFront for content distribution
4. Parallelize requests (use multiple threads/processes)
5. Use byte-range fetches for large objects
6. Consider S3 prefix design for high request rates
7. Use caching headers

### Data Management
1. Use meaningful bucket and object naming conventions
2. Tag buckets and objects for organization
3. Enable cross-region replication for disaster recovery
4. Regular backup testing
5. Document retention policies
6. Use S3 Inventory for large-scale analysis
7. Monitor with CloudWatch metrics

### Development with LocalStack
```bash
# Set endpoint for all commands
export AWS_ENDPOINT_URL=http://localhost:4566

# Or use per-command
aws s3 ls --endpoint-url=http://localhost:4566

# LocalStack specific features
aws s3api list-buckets --endpoint-url=http://localhost:4566
```

---

## Common Use Case Examples

### 1. Static Website Deployment
```bash
# Create bucket
aws s3 mb s3://my-static-website

# Upload website files
aws s3 sync ./website/ s3://my-static-website/

# Enable website hosting
aws s3 website s3://my-static-website/ --index-document index.html --error-document error.html

# Make public
aws s3api put-bucket-policy --bucket my-static-website --policy file://public-policy.json
```

### 2. Backup Solution
```bash
# Sync local directory to S3
aws s3 sync /data/backups/ s3://my-backup-bucket/daily/ --delete

# Add lifecycle policy for cost optimization
aws s3api put-bucket-lifecycle-configuration --bucket my-backup-bucket --lifecycle-configuration file://backup-lifecycle.json
```

### 3. Data Lake
```bash
# Create bucket with encryption
aws s3 mb s3://my-data-lake
aws s3api put-bucket-encryption --bucket my-data-lake --server-side-encryption-configuration file://encryption.json

# Enable versioning
aws s3api put-bucket-versioning --bucket my-data-lake --versioning-configuration Status=Enabled

# Upload data with metadata
aws s3 cp dataset.parquet s3://my-data-lake/raw/2025/11/ --metadata source=analytics,date=2025-11-17
```

### 4. Cross-Region Disaster Recovery
```bash
# Enable versioning (both buckets)
aws s3api put-bucket-versioning --bucket my-primary-bucket --versioning-configuration Status=Enabled
aws s3api put-bucket-versioning --bucket my-replica-bucket --versioning-configuration Status=Enabled --region eu-west-1

# Configure replication
aws s3api put-bucket-replication --bucket my-primary-bucket --replication-configuration file://replication.json
```

---

## Troubleshooting Common Issues

### Access Denied Errors
1. Check bucket policy and IAM permissions
2. Verify public access block settings
3. Check ACLs
4. Verify KMS key permissions (if using encryption)

### Slow Upload/Download
1. Enable Transfer Acceleration
2. Use multipart uploads
3. Check network connectivity
4. Parallelize operations

### Versioning Issues
1. List all versions to see delete markers
2. Check lifecycle policies
3. Verify MFA Delete isn't blocking operations

### CORS Errors
1. Verify CORS configuration
2. Check AllowedOrigins matches exactly
3. Ensure bucket is publicly accessible (or presigned URLs used)
4. Check browser console for specific error

---

## Quick Reference Commands

```bash
# Bucket operations
aws s3 mb s3://bucket-name                          # Create bucket
aws s3 rb s3://bucket-name                          # Delete empty bucket
aws s3 rb s3://bucket-name --force                  # Delete bucket with contents
aws s3 ls                                           # List all buckets
aws s3 ls s3://bucket-name                          # List bucket contents

# Object operations
aws s3 cp file.txt s3://bucket-name/                # Upload file
aws s3 cp s3://bucket-name/file.txt .               # Download file
aws s3 sync ./local/ s3://bucket-name/              # Sync directory
aws s3 rm s3://bucket-name/file.txt                 # Delete file
aws s3 mv s3://bucket/old.txt s3://bucket/new.txt   # Move/rename

# Presigned URLs
aws s3 presign s3://bucket-name/file.txt --expires-in 3600

# Advanced
aws s3api head-object --bucket name --key file.txt  # Get metadata
aws s3api list-object-versions --bucket name        # List versions
```

---

## Summary

S3 is AWS's foundational storage service with virtually unlimited scalability. Master these commands to:
- Store and retrieve any amount of data
- Host static websites
- Build data lakes
- Implement backup solutions
- Distribute content globally
- Comply with regulatory requirements

Practice these commands regularly, experiment with different configurations, and integrate S3 into your applications. S3's durability (99.999999999%) and availability make it ideal for production workloads.

For LocalStack testing, always add `--endpoint-url=http://localhost:4566` to commands.

Happy learning!
