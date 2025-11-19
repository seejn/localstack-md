# LocalStack Setup Guide - Complete Docker Installation

This comprehensive guide will walk you through every step required to install and set up LocalStack on your machine using Docker.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installing Docker](#installing-docker)
3. [Installing LocalStack](#installing-localstack)
4. [Configuration](#configuration)
5. [Starting LocalStack](#starting-localstack)
6. [Verifying Installation](#verifying-installation)
7. [Common Commands](#common-commands)
8. [Troubleshooting](#troubleshooting)
9. [Advanced Configuration](#advanced-configuration)

---

## Prerequisites

### System Requirements
- **Operating System**: Linux, macOS, or Windows 10/11
- **RAM**: Minimum 4GB, recommended 8GB or more
- **Disk Space**: At least 10GB free space
- **Internet Connection**: Required for initial setup

### Required Software
- Docker Engine (version 20.10 or later)
- Docker Compose (optional but recommended)
- AWS CLI (optional, for testing)

---

## Installing Docker

### For Linux (Ubuntu/Debian)

#### Step 1: Update Package Index
```bash
sudo apt-get update
```

#### Step 2: Install Prerequisites
```bash
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

#### Step 3: Add Docker's Official GPG Key
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

#### Step 4: Set Up Docker Repository
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Step 5: Install Docker Engine
```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Step 6: Verify Docker Installation
```bash
sudo docker --version
```

#### Step 7: Add Your User to Docker Group (Optional)
This allows running Docker without `sudo`:
```bash
sudo usermod -aG docker $USER
```

**Important**: Log out and log back in for this change to take effect.

#### Step 8: Test Docker Installation
```bash
docker run hello-world
```

### For macOS

#### Step 1: Download Docker Desktop
- Visit: https://www.docker.com/products/docker-desktop
- Download Docker Desktop for Mac (Intel or Apple Silicon)

#### Step 2: Install Docker Desktop
- Open the downloaded `.dmg` file
- Drag Docker to Applications folder
- Launch Docker from Applications

#### Step 3: Verify Installation
Open Terminal and run:
```bash
docker --version
docker-compose --version
```

### For Windows

#### Step 1: Enable WSL 2
Open PowerShell as Administrator:
```powershell
wsl --install
```

#### Step 2: Download Docker Desktop
- Visit: https://www.docker.com/products/docker-desktop
- Download Docker Desktop for Windows

#### Step 3: Install Docker Desktop
- Run the installer
- Follow installation wizard
- Restart computer when prompted

#### Step 4: Verify Installation
Open PowerShell or Command Prompt:
```bash
docker --version
docker-compose --version
```

---

## Installing LocalStack

### Method 1: Using Docker Run (Quick Start)

#### Step 1: Pull LocalStack Docker Image
```bash
docker pull localstack/localstack:latest
```

This downloads the latest LocalStack image (approximately 1-2GB).

#### Step 2: Create LocalStack Data Directory
```bash
mkdir -p $HOME/.localstack
```

This directory will persist LocalStack data between restarts.

#### Step 3: Run LocalStack Container
```bash
docker run -d \
  --name localstack \
  -p 4566:4566 \
  -p 4510-4559:4510-4559 \
  -e SERVICES=s3,lambda,dynamodb,sqs,sns,apigateway,cloudformation,iam,sts \
  -e DEBUG=1 \
  -e DATA_DIR=/tmp/localstack/data \
  -v $HOME/.localstack:/var/lib/localstack \
  -v /var/run/docker.sock:/var/run/docker.sock \
  localstack/localstack:latest
```

**Explanation of flags:**
- `-d`: Run container in detached mode (background)
- `--name localstack`: Name the container "localstack"
- `-p 4566:4566`: Expose main LocalStack port
- `-p 4510-4559:4510-4559`: Expose additional service ports
- `-e SERVICES`: Specify which AWS services to enable
- `-e DEBUG=1`: Enable debug logging
- `-v $HOME/.localstack`: Mount volume for persistence
- `-v /var/run/docker.sock`: Allow LocalStack to spawn Docker containers

### Method 2: Using Docker Compose (Recommended)

#### Step 1: Create Project Directory
```bash
mkdir -p ~/localstack-setup
cd ~/localstack-setup
```

#### Step 2: Create docker-compose.yml File
```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  localstack:
    image: localstack/localstack:latest
    container_name: localstack
    ports:
      - "4566:4566"            # LocalStack Gateway
      - "4510-4559:4510-4559"  # External services port range
    environment:
      # Core Configuration
      - DEBUG=1
      - DOCKER_HOST=unix:///var/run/docker.sock

      # Service Configuration (comma-separated list)
      - SERVICES=s3,lambda,dynamodb,sqs,sns,apigateway,cloudformation,iam,sts,secretsmanager,ssm,cloudwatch,logs,events,kinesis,ec2,rds,redshift,es,elasticache

      # Data Persistence
      - DATA_DIR=/tmp/localstack/data
      - PERSISTENCE=1

      # Lambda Configuration
      - LAMBDA_EXECUTOR=docker
      - LAMBDA_REMOTE_DOCKER=0

      # Additional Settings
      - EDGE_PORT=4566
      - DEFAULT_REGION=us-east-1

    volumes:
      # Persist LocalStack data
      - ./volume:/var/lib/localstack

      # Mount Docker socket for Lambda execution
      - /var/run/docker.sock:/var/run/docker.sock

      # Optional: Mount init scripts
      # - ./init:/etc/localstack/init

    networks:
      - localstack-network

networks:
  localstack-network:
    driver: bridge
EOF
```

#### Step 3: Create Volume Directory
```bash
mkdir -p volume
```

#### Step 4: Start LocalStack
```bash
docker-compose up -d
```

#### Step 5: Verify Container is Running
```bash
docker-compose ps
```

You should see output similar to:
```
NAME          IMAGE                         STATUS        PORTS
localstack    localstack/localstack:latest  Up X seconds  0.0.0.0:4566->4566/tcp
```

---

## Configuration

### Environment Variables

Create a `.env` file in your project directory for custom configuration:

```bash
cat > .env << 'EOF'
# Core Settings
DEBUG=1
DOCKER_HOST=unix:///var/run/docker.sock

# AWS Services (comma-separated)
SERVICES=s3,lambda,dynamodb,sqs,sns,apigateway

# Region Configuration
DEFAULT_REGION=us-east-1
AWS_DEFAULT_REGION=us-east-1

# Data Persistence
DATA_DIR=/tmp/localstack/data
PERSISTENCE=1

# Lambda Settings
LAMBDA_EXECUTOR=docker
LAMBDA_DOCKER_NETWORK=localstack-network
LAMBDA_REMOVE_CONTAINERS=1

# Network Settings
EDGE_PORT=4566
GATEWAY_LISTEN=0.0.0.0:4566

# Optional: Pro Features (requires license)
# LOCALSTACK_API_KEY=your-api-key-here
EOF
```

Update your `docker-compose.yml` to use the `.env` file:
```yaml
services:
  localstack:
    env_file:
      - .env
```

### Service-Specific Configuration

#### Enable All Services
```bash
SERVICES=
```
(Empty value enables all services)

#### Enable Specific Services Only
```bash
SERVICES=s3,lambda,dynamodb,sqs,sns
```

#### Common Service Names
- `s3` - Simple Storage Service
- `lambda` - Lambda Functions
- `dynamodb` - DynamoDB
- `sqs` - Simple Queue Service
- `sns` - Simple Notification Service
- `apigateway` - API Gateway
- `cloudformation` - CloudFormation
- `iam` - Identity and Access Management
- `sts` - Security Token Service
- `secretsmanager` - Secrets Manager
- `ssm` - Systems Manager
- `cloudwatch` - CloudWatch
- `logs` - CloudWatch Logs
- `events` - EventBridge
- `kinesis` - Kinesis Streams
- `ec2` - Elastic Compute Cloud
- `rds` - Relational Database Service

---

## Starting LocalStack

### Using Docker Run

#### Start LocalStack
```bash
docker start localstack
```

#### Stop LocalStack
```bash
docker stop localstack
```

#### Restart LocalStack
```bash
docker restart localstack
```

#### Remove LocalStack Container
```bash
docker stop localstack
docker rm localstack
```

### Using Docker Compose

#### Start LocalStack (Foreground)
```bash
docker-compose up
```

#### Start LocalStack (Background)
```bash
docker-compose up -d
```

#### Stop LocalStack
```bash
docker-compose down
```

#### Stop and Remove Volumes
```bash
docker-compose down -v
```

#### Restart LocalStack
```bash
docker-compose restart
```

#### View Logs
```bash
docker-compose logs -f
```

#### View Logs for Last 100 Lines
```bash
docker-compose logs --tail=100 -f
```

---

## Verifying Installation

### Step 1: Check Container Status
```bash
docker ps | grep localstack
```

Expected output shows container running with ports exposed.

### Step 2: Check LocalStack Health Endpoint
```bash
curl http://localhost:4566/_localstack/health
```

Expected output (JSON):
```json
{
  "services": {
    "s3": "running",
    "lambda": "running",
    "dynamodb": "running",
    ...
  },
  "version": "3.x.x"
}
```

### Step 3: Install AWS CLI (If Not Installed)

#### Linux/macOS
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

#### Verify AWS CLI Installation
```bash
aws --version
```

### Step 4: Configure AWS CLI for LocalStack

Create AWS credentials file:
```bash
mkdir -p ~/.aws

cat > ~/.aws/credentials << 'EOF'
[localstack]
aws_access_key_id = test
aws_secret_access_key = test
EOF

cat > ~/.aws/config << 'EOF'
[profile localstack]
region = us-east-1
output = json
EOF
```

### Step 5: Test S3 Service

#### Create S3 Bucket
```bash
aws --endpoint-url=http://localhost:4566 s3 mb s3://test-bucket --profile localstack
```

#### List S3 Buckets
```bash
aws --endpoint-url=http://localhost:4566 s3 ls --profile localstack
```

#### Upload File to S3
```bash
echo "Hello LocalStack" > test.txt
aws --endpoint-url=http://localhost:4566 s3 cp test.txt s3://test-bucket/ --profile localstack
```

#### List Files in Bucket
```bash
aws --endpoint-url=http://localhost:4566 s3 ls s3://test-bucket/ --profile localstack
```

### Step 6: Test DynamoDB Service

#### Create DynamoDB Table
```bash
aws --endpoint-url=http://localhost:4566 dynamodb create-table \
  --table-name TestTable \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --profile localstack
```

#### List DynamoDB Tables
```bash
aws --endpoint-url=http://localhost:4566 dynamodb list-tables --profile localstack
```

### Step 7: Test SQS Service

#### Create SQS Queue
```bash
aws --endpoint-url=http://localhost:4566 sqs create-queue \
  --queue-name test-queue \
  --profile localstack
```

#### List SQS Queues
```bash
aws --endpoint-url=http://localhost:4566 sqs list-queues --profile localstack
```

---

## Common Commands

### Container Management

#### View Container Logs
```bash
docker logs localstack -f
```

#### View Last 50 Log Lines
```bash
docker logs localstack --tail=50
```

#### Execute Command Inside Container
```bash
docker exec -it localstack bash
```

#### Check Container Resource Usage
```bash
docker stats localstack
```

#### Inspect Container Details
```bash
docker inspect localstack
```

### LocalStack CLI Commands

#### Install LocalStack CLI (Optional)
```bash
pip install localstack
```

#### Start LocalStack via CLI
```bash
localstack start -d
```

#### Stop LocalStack via CLI
```bash
localstack stop
```

#### Check LocalStack Status
```bash
localstack status
```

### AWS Service Shortcuts

Create an alias for easier AWS CLI usage with LocalStack:

```bash
# Add to ~/.bashrc or ~/.zshrc
alias awslocal="aws --endpoint-url=http://localhost:4566 --profile localstack"
```

Then reload your shell:
```bash
source ~/.bashrc  # or source ~/.zshrc
```

Now you can use:
```bash
awslocal s3 ls
awslocal dynamodb list-tables
awslocal lambda list-functions
```

---

## Troubleshooting

### Issue 1: Container Won't Start

#### Check Docker Service Status
```bash
sudo systemctl status docker
```

#### Start Docker Service
```bash
sudo systemctl start docker
```

#### Check Docker Logs
```bash
journalctl -u docker.service -f
```

### Issue 2: Port Already in Use

#### Find Process Using Port 4566
```bash
sudo lsof -i :4566
```

#### Kill Process Using Port
```bash
sudo kill -9 <PID>
```

#### Or Change LocalStack Port
Edit `docker-compose.yml`:
```yaml
ports:
  - "4567:4566"  # Use different host port
```

### Issue 3: Permission Denied Errors

#### Fix Docker Socket Permissions
```bash
sudo chmod 666 /var/run/docker.sock
```

#### Or Add User to Docker Group
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Issue 4: Services Not Starting

#### Enable Debug Logging
```yaml
environment:
  - DEBUG=1
  - LS_LOG=trace
```

#### Check LocalStack Logs
```bash
docker logs localstack | grep ERROR
```

### Issue 5: Lambda Functions Failing

#### Ensure Docker Socket is Mounted
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```

#### Check Lambda Executor Setting
```yaml
environment:
  - LAMBDA_EXECUTOR=docker
```

#### Verify Docker Containers Can Be Created
```bash
docker run --rm hello-world
```

### Issue 6: Data Not Persisting

#### Ensure Volume is Mounted
```bash
docker inspect localstack | grep Mounts -A 10
```

#### Check Persistence Setting
```yaml
environment:
  - PERSISTENCE=1
```

#### Verify Volume Directory Permissions
```bash
ls -la ./volume
chmod -R 755 ./volume
```

### Issue 7: Network Issues

#### Check Container Network
```bash
docker network ls
docker network inspect localstack-network
```

#### Test Connectivity
```bash
curl -v http://localhost:4566/_localstack/health
```

#### Check Firewall Rules
```bash
sudo ufw status
sudo ufw allow 4566/tcp
```

---

## Advanced Configuration

### Init Scripts

Create initialization scripts that run when LocalStack starts:

#### Step 1: Create Init Directory
```bash
mkdir -p init/ready.d
```

#### Step 2: Create Init Script
```bash
cat > init/ready.d/init-resources.sh << 'EOF'
#!/bin/bash

echo "Creating S3 buckets..."
awslocal s3 mb s3://my-app-bucket

echo "Creating DynamoDB tables..."
awslocal dynamodb create-table \
  --table-name Users \
  --attribute-definitions AttributeName=userId,AttributeType=S \
  --key-schema AttributeName=userId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

echo "Creating SQS queues..."
awslocal sqs create-queue --queue-name notifications-queue

echo "Initialization complete!"
EOF

chmod +x init/ready.d/init-resources.sh
```

#### Step 3: Mount Init Directory
Update `docker-compose.yml`:
```yaml
volumes:
  - ./init:/etc/localstack/init/ready.d
```

### Using LocalStack with Custom Domain

#### Step 1: Add Entry to Hosts File
```bash
sudo sh -c 'echo "127.0.0.1 localstack.local" >> /etc/hosts'
```

#### Step 2: Access via Custom Domain
```bash
curl http://localstack.local:4566/_localstack/health
```

### Resource Limits

Limit container resources in `docker-compose.yml`:
```yaml
services:
  localstack:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          memory: 2G
```

### Multiple LocalStack Instances

Run multiple isolated LocalStack instances:

```yaml
version: '3.8'

services:
  localstack-dev:
    image: localstack/localstack:latest
    container_name: localstack-dev
    ports:
      - "4566:4566"
    environment:
      - SERVICES=s3,dynamodb
    volumes:
      - ./volume-dev:/var/lib/localstack

  localstack-test:
    image: localstack/localstack:latest
    container_name: localstack-test
    ports:
      - "4567:4566"
    environment:
      - SERVICES=s3,dynamodb
    volumes:
      - ./volume-test:/var/lib/localstack
```

### Using LocalStack Pro

If you have a LocalStack Pro license:

```yaml
environment:
  - LOCALSTACK_API_KEY=your-api-key-here
```

Pro features include:
- IAM security enforcement
- Advanced Lambda features
- Extended service coverage
- Cloud pods for state management
- Performance improvements

---

## Quick Reference Card

### Essential Commands Cheat Sheet

```bash
# Start LocalStack
docker-compose up -d

# Stop LocalStack
docker-compose down

# View logs
docker-compose logs -f

# Check health
curl http://localhost:4566/_localstack/health

# Create S3 bucket
awslocal s3 mb s3://my-bucket

# List S3 buckets
awslocal s3 ls

# Create DynamoDB table
awslocal dynamodb create-table --table-name MyTable \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# List DynamoDB tables
awslocal dynamodb list-tables

# Create SQS queue
awslocal sqs create-queue --queue-name my-queue

# List SQS queues
awslocal sqs list-queues

# Restart LocalStack
docker-compose restart
```

### Default Endpoints

- Main Gateway: `http://localhost:4566`
- Health Check: `http://localhost:4566/_localstack/health`
- S3: `http://localhost:4566` or `http://s3.localhost.localstack.cloud:4566`

### Default Credentials

```
AWS Access Key ID: test
AWS Secret Access Key: test
Region: us-east-1
```

---

## Next Steps

1. **Explore Services**: Test different AWS services available in LocalStack
2. **Integrate with Your Application**: Update your app configuration to use LocalStack endpoints
3. **Set Up CI/CD**: Integrate LocalStack into your testing pipeline
4. **Learn Advanced Features**: Explore CloudFormation, SAM, and Terraform with LocalStack
5. **Join Community**: Visit https://discuss.localstack.cloud for support

---

## Additional Resources

- Official Documentation: https://docs.localstack.cloud
- GitHub Repository: https://github.com/localstack/localstack
- Community Slack: https://localstack.cloud/slack
- AWS CLI Reference: https://docs.aws.amazon.com/cli/latest/reference/

---

## Summary

You now have LocalStack fully installed and configured on your machine! You can:
- Develop and test AWS applications locally
- Avoid AWS charges during development
- Work offline without internet connectivity
- Run fast integration tests
- Experiment with AWS services risk-free

Happy local cloud development!
