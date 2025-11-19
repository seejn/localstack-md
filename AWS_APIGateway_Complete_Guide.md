# AWS API Gateway - Complete Practical Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Key Concepts](#key-concepts)
3. [REST API Management](#rest-api-management)
4. [HTTP API Management](#http-api-management)
5. [Resources & Methods](#resources--methods)
6. [Integrations](#integrations)
7. [Request/Response Transformations](#requestresponse-transformations)
8. [Authorization](#authorization)
9. [Stages & Deployments](#stages--deployments)
10. [Custom Domains](#custom-domains)
11. [Usage Plans & API Keys](#usage-plans--api-keys)
12. [CORS Configuration](#cors-configuration)
13. [WebSocket APIs](#websocket-apis)
14. [Monitoring & Logging](#monitoring--logging)
15. [Best Practices](#best-practices)

---

## Introduction

Amazon API Gateway is a fully managed service for creating, publishing, maintaining, monitoring, and securing REST, HTTP, and WebSocket APIs at any scale.

**Use Cases:**
- RESTful APIs for CRUD operations
- Real-time two-way communication (WebSockets)
- Serverless API backends
- Mobile and web application backends
- Microservices architecture
- API monetization

**Benefits:**
- Serverless and scalable
- Pay-per-use pricing
- Built-in security features
- Multiple API types (REST, HTTP, WebSocket)
- Request/response transformation
- API versioning and staging

---

## Key Concepts

### API Types

**REST API:**
- Full API management features
- Request/response transformation
- API keys, usage plans, and throttling
- Custom authorizers
- More expensive than HTTP API

**HTTP API:**
- Lower cost (up to 71% cheaper)
- Lower latency
- OIDC and OAuth 2.0 support
- Simpler configuration
- Fewer features than REST API

**WebSocket API:**
- Two-way communication
- Real-time applications
- Persistent connections
- Chat apps, gaming, trading platforms

### Core Components

**Resources:** URL paths (e.g., /users, /products)
**Methods:** HTTP verbs (GET, POST, PUT, DELETE, etc.)
**Integrations:** Backend services (Lambda, HTTP, AWS services)
**Stages:** Different versions (dev, staging, prod)
**Deployments:** Snapshots of API configuration
**Authorizers:** Authentication and authorization mechanisms

---

## REST API Management

### 1. Create REST API

```bash
aws apigateway create-rest-api \
    --name my-rest-api \
    --description "My REST API" \
    --endpoint-configuration types=REGIONAL
```

**Expected Output:**
```json
{
    "id": "abc123xyz",
    "name": "my-rest-api",
    "description": "My REST API",
    "createdDate": "2025-11-17T10:00:00+00:00",
    "endpointConfiguration": {
        "types": ["REGIONAL"]
    }
}
```

**Why:** Create a new REST API Gateway.

**When to Use:** Building RESTful APIs with full features and control.

**Endpoint Types:**
- **REGIONAL**: Deployed in specific region, lower latency for regional users
- **EDGE**: CloudFront distribution, lower latency globally
- **PRIVATE**: Accessible only from VPC

**LocalStack:**
```bash
aws apigateway create-rest-api --name my-rest-api --endpoint-url=http://localhost:4566
```

---

### 2. List REST APIs

```bash
aws apigateway get-rest-apis
```

**Expected Output:**
```json
{
    "items": [
        {
            "id": "abc123xyz",
            "name": "my-rest-api",
            "description": "My REST API",
            "createdDate": "2025-11-17T10:00:00+00:00",
            "endpointConfiguration": {
                "types": ["REGIONAL"]
            }
        }
    ]
}
```

**Why:** View all REST APIs in your account.

**When to Use:** Finding API IDs, auditing, inventory management.

---

### 3. Get REST API Details

```bash
aws apigateway get-rest-api --rest-api-id abc123xyz
```

**Expected Output:**
```json
{
    "id": "abc123xyz",
    "name": "my-rest-api",
    "description": "My REST API",
    "createdDate": "2025-11-17T10:00:00+00:00",
    "endpointConfiguration": {
        "types": ["REGIONAL"]
    }
}
```

**Why:** Get details of specific API.

**When to Use:** Checking configuration, documentation, troubleshooting.

---

### 4. Update REST API

```bash
aws apigateway update-rest-api \
    --rest-api-id abc123xyz \
    --patch-operations op=replace,path=/description,value="Updated description"
```

**Expected Output:**
```json
{
    "id": "abc123xyz",
    "name": "my-rest-api",
    "description": "Updated description"
}
```

**Why:** Modify API settings.

**When to Use:** Updating configuration, changing endpoint type.

---

### 5. Delete REST API

```bash
aws apigateway delete-rest-api --rest-api-id abc123xyz
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove API and all resources/methods/stages.

**When to Use:** Cleanup, decommissioning.

---

## HTTP API Management

### 1. Create HTTP API

```bash
aws apigatewayv2 create-api \
    --name my-http-api \
    --protocol-type HTTP \
    --description "My HTTP API"
```

**Expected Output:**
```json
{
    "ApiId": "xyz789abc",
    "Name": "my-http-api",
    "ProtocolType": "HTTP",
    "RouteSelectionExpression": "$request.method $request.path",
    "Description": "My HTTP API",
    "CreatedDate": "2025-11-17T10:00:00+00:00"
}
```

**Why:** Create HTTP API (simpler and cheaper than REST API).

**When to Use:** Simpler APIs, cost optimization, lower latency requirements.

---

### 2. Create HTTP API with Quick Create

```bash
aws apigatewayv2 create-api \
    --name my-quick-api \
    --protocol-type HTTP \
    --target arn:aws:lambda:us-east-1:123456789012:function:my-function
```

**Expected Output:**
```json
{
    "ApiId": "quick123",
    "Name": "my-quick-api",
    "ProtocolType": "HTTP",
    "ApiEndpoint": "https://quick123.execute-api.us-east-1.amazonaws.com"
}
```

**Why:** Quickly create HTTP API with default Lambda integration.

**When to Use:** Rapid prototyping, simple Lambda-backed APIs.

---

### 3. List HTTP APIs

```bash
aws apigatewayv2 get-apis
```

**Expected Output:**
```json
{
    "Items": [
        {
            "ApiId": "xyz789abc",
            "Name": "my-http-api",
            "ProtocolType": "HTTP",
            "ApiEndpoint": "https://xyz789abc.execute-api.us-east-1.amazonaws.com",
            "CreatedDate": "2025-11-17T10:00:00+00:00"
        }
    ]
}
```

**Why:** View all HTTP APIs.

**When to Use:** Inventory, finding API IDs.

---

### 4. Get HTTP API Details

```bash
aws apigatewayv2 get-api --api-id xyz789abc
```

**Expected Output:**
```json
{
    "ApiId": "xyz789abc",
    "Name": "my-http-api",
    "ProtocolType": "HTTP",
    "RouteSelectionExpression": "$request.method $request.path",
    "ApiEndpoint": "https://xyz789abc.execute-api.us-east-1.amazonaws.com"
}
```

**Why:** Get details of specific HTTP API.

**When to Use:** Checking configuration, documentation.

---

### 5. Delete HTTP API

```bash
aws apigatewayv2 delete-api --api-id xyz789abc
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove HTTP API.

**When to Use:** Cleanup, decommissioning.

---

## Resources & Methods

### 1. Get Root Resource (REST API)

```bash
aws apigateway get-resources --rest-api-id abc123xyz
```

**Expected Output:**
```json
{
    "items": [
        {
            "id": "root123",
            "path": "/"
        }
    ]
}
```

**Why:** Get root resource ID (needed for creating child resources).

**When to Use:** Starting to build API structure.

---

### 2. Create Resource (REST API)

```bash
aws apigateway create-resource \
    --rest-api-id abc123xyz \
    --parent-id root123 \
    --path-part users
```

**Expected Output:**
```json
{
    "id": "res456",
    "parentId": "root123",
    "pathPart": "users",
    "path": "/users"
}
```

**Why:** Create URL path (e.g., /users, /products).

**When to Use:** Building API structure, adding endpoints.

**Create nested resource:**
```bash
# /users/{id}
aws apigateway create-resource \
    --rest-api-id abc123xyz \
    --parent-id res456 \
    --path-part '{id}'
```

---

### 3. Create Method (REST API)

```bash
aws apigateway put-method \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --authorization-type NONE \
    --request-parameters method.request.querystring.limit=false
```

**Expected Output:**
```json
{
    "httpMethod": "GET",
    "authorizationType": "NONE",
    "apiKeyRequired": false,
    "requestParameters": {
        "method.request.querystring.limit": false
    }
}
```

**Why:** Add HTTP method to resource (GET, POST, etc.).

**When to Use:** Defining API operations.

**Authorization Types:**
- **NONE**: No authorization
- **AWS_IAM**: IAM authorization
- **CUSTOM**: Lambda authorizer
- **COGNITO_USER_POOLS**: Cognito User Pool

---

### 4. Create Route (HTTP API)

```bash
aws apigatewayv2 create-route \
    --api-id xyz789abc \
    --route-key "GET /users" \
    --target integrations/integ123
```

**Expected Output:**
```json
{
    "RouteId": "route456",
    "RouteKey": "GET /users",
    "Target": "integrations/integ123"
}
```

**Why:** Define routes in HTTP API (simpler than REST API resources).

**When to Use:** Building HTTP API structure.

**Route Key Examples:**
- `GET /users`
- `POST /users`
- `GET /users/{id}`
- `$default` (catch-all route)

---

### 5. List Routes (HTTP API)

```bash
aws apigatewayv2 get-routes --api-id xyz789abc
```

**Expected Output:**
```json
{
    "Items": [
        {
            "RouteId": "route456",
            "RouteKey": "GET /users",
            "Target": "integrations/integ123"
        }
    ]
}
```

**Why:** View all routes in HTTP API.

**When to Use:** Auditing API structure, documentation.

---

### 6. Delete Method (REST API)

```bash
aws apigateway delete-method \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove HTTP method from resource.

**When to Use:** API refactoring, removing endpoints.

---

### 7. Delete Resource (REST API)

```bash
aws apigateway delete-resource \
    --rest-api-id abc123xyz \
    --resource-id res456
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove resource (deletes all child resources and methods).

**When to Use:** API restructuring, cleanup.

---

## Integrations

### 1. Create Lambda Integration (REST API)

```bash
aws apigateway put-integration \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --type AWS_PROXY \
    --integration-http-method POST \
    --uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:123456789012:function:my-function/invocations
```

**Expected Output:**
```json
{
    "type": "AWS_PROXY",
    "httpMethod": "POST",
    "uri": "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:123456789012:function:my-function/invocations",
    "passthroughBehavior": "WHEN_NO_MATCH",
    "timeoutInMillis": 29000
}
```

**Why:** Connect API method to Lambda function.

**When to Use:** Serverless API backends.

**Integration Types:**
- **AWS_PROXY**: Lambda proxy integration (recommended)
- **AWS**: AWS service integration with transformation
- **HTTP_PROXY**: HTTP proxy to backend
- **HTTP**: HTTP integration with transformation
- **MOCK**: Return response without backend

**Note:** integration-http-method is always POST for Lambda.

---

### 2. Grant Lambda Invoke Permission

```bash
aws lambda add-permission \
    --function-name my-function \
    --statement-id apigateway-invoke \
    --action lambda:InvokeFunction \
    --principal apigateway.amazonaws.com \
    --source-arn "arn:aws:execute-api:us-east-1:123456789012:abc123xyz/*/*/*"
```

**Expected Output:**
```json
{
    "Statement": "{\"Sid\":\"apigateway-invoke\",\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"apigateway.amazonaws.com\"},\"Action\":\"lambda:InvokeFunction\",\"Resource\":\"arn:aws:lambda:us-east-1:123456789012:function:my-function\",\"Condition\":{\"ArnLike\":{\"AWS:SourceArn\":\"arn:aws:execute-api:us-east-1:123456789012:abc123xyz/*/*/*\"}}}"
}
```

**Why:** Allow API Gateway to invoke Lambda function.

**When to Use:** Required for Lambda integrations.

**Source ARN Format:** `arn:aws:execute-api:region:account-id:api-id/stage/method/resource`

---

### 3. Create Lambda Integration (HTTP API)

```bash
# Create integration
aws apigatewayv2 create-integration \
    --api-id xyz789abc \
    --integration-type AWS_PROXY \
    --integration-uri arn:aws:lambda:us-east-1:123456789012:function:my-function \
    --payload-format-version 2.0
```

**Expected Output:**
```json
{
    "IntegrationId": "integ123",
    "IntegrationType": "AWS_PROXY",
    "IntegrationUri": "arn:aws:lambda:us-east-1:123456789012:function:my-function",
    "PayloadFormatVersion": "2.0"
}
```

**Why:** Connect HTTP API route to Lambda.

**When to Use:** HTTP API backends.

**Payload Format Versions:**
- **1.0**: Same as REST API format
- **2.0**: Simplified format (recommended for HTTP APIs)

---

### 4. Create HTTP Integration (REST API)

```bash
aws apigateway put-integration \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --type HTTP_PROXY \
    --integration-http-method GET \
    --uri https://api.example.com/users
```

**Expected Output:**
```json
{
    "type": "HTTP_PROXY",
    "httpMethod": "GET",
    "uri": "https://api.example.com/users"
}
```

**Why:** Proxy requests to external HTTP endpoint.

**When to Use:** Integrating existing APIs, microservices.

---

### 5. Create Mock Integration (REST API)

```bash
aws apigateway put-integration \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --type MOCK \
    --request-templates '{"application/json": "{\"statusCode\": 200}"}'
```

**Expected Output:**
```json
{
    "type": "MOCK",
    "requestTemplates": {
        "application/json": "{\"statusCode\": 200}"
    }
}
```

**Why:** Return mock responses without backend.

**When to Use:** API prototyping, testing, static responses.

---

### 6. Configure Integration Response (REST API)

```bash
aws apigateway put-integration-response \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --status-code 200 \
    --response-templates '{"application/json": ""}'
```

**Expected Output:**
```json
{
    "statusCode": "200",
    "responseTemplates": {
        "application/json": ""
    }
}
```

**Why:** Configure how integration response is transformed to method response.

**When to Use:** Custom response transformations, error handling.

---

### 7. Configure Method Response (REST API)

```bash
aws apigateway put-method-response \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --status-code 200 \
    --response-models '{"application/json": "Empty"}'
```

**Expected Output:**
```json
{
    "statusCode": "200",
    "responseModels": {
        "application/json": "Empty"
    }
}
```

**Why:** Define expected response structure.

**When to Use:** API documentation, validation, SDK generation.

---

## Request/Response Transformations

### 1. Request Template (Velocity Template)

```bash
aws apigateway put-integration \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method POST \
    --type AWS \
    --integration-http-method POST \
    --uri arn:aws:apigateway:us-east-1:dynamodb:action/PutItem \
    --request-templates file://request-template.json
```

**request-template.json:**
```json
{
    "application/json": "{\"TableName\": \"Users\", \"Item\": {\"id\": {\"S\": \"$input.path('$.userId')\"}, \"name\": {\"S\": \"$input.path('$.userName')\"}}}"
}
```

**Why:** Transform incoming requests before sending to backend.

**When to Use:** Mapping API Gateway format to backend format (DynamoDB, etc.).

**Velocity Template Variables:**
- `$input.path('$.field')` - Extract from JSON body
- `$input.params('param')` - Get query/path parameter
- `$context.requestId` - Request ID
- `$util.urlEncode()` - URL encoding

---

### 2. Response Template (Velocity Template)

```bash
aws apigateway put-integration-response \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --status-code 200 \
    --response-templates file://response-template.json
```

**response-template.json:**
```json
{
    "application/json": "#set($items = $input.path('$.Items'))\n[\n#foreach($item in $items)\n  {\n    \"id\": \"$item.id.S\",\n    \"name\": \"$item.name.S\"\n  }#if($foreach.hasNext),#end\n#end\n]"
}
```

**Why:** Transform backend responses to desired format.

**When to Use:** Formatting DynamoDB responses, cleaning up data.

---

### 3. Request Validation (REST API)

**Create validator:**
```bash
aws apigateway create-request-validator \
    --rest-api-id abc123xyz \
    --name my-validator \
    --validate-request-body true \
    --validate-request-parameters true
```

**Expected Output:**
```json
{
    "id": "validator123",
    "name": "my-validator",
    "validateRequestBody": true,
    "validateRequestParameters": true
}
```

**Apply to method:**
```bash
aws apigateway update-method \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method POST \
    --patch-operations op=replace,path=/requestValidatorId,value=validator123
```

**Why:** Validate requests before sending to backend.

**When to Use:** Input validation, reducing backend load, security.

---

### 4. Create Request Model (REST API)

```bash
aws apigateway create-model \
    --rest-api-id abc123xyz \
    --name UserModel \
    --content-type application/json \
    --schema file://user-schema.json
```

**user-schema.json (JSON Schema):**
```json
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "User",
    "type": "object",
    "properties": {
        "userId": {
            "type": "string"
        },
        "userName": {
            "type": "string"
        },
        "email": {
            "type": "string",
            "format": "email"
        }
    },
    "required": ["userId", "userName"]
}
```

**Expected Output:**
```json
{
    "id": "model123",
    "name": "UserModel",
    "contentType": "application/json",
    "schema": "..."
}
```

**Why:** Define request/response structure for validation and documentation.

**When to Use:** API validation, SDK generation, documentation.

---

## Authorization

### 1. Create Lambda Authorizer (REST API)

```bash
aws apigateway create-authorizer \
    --rest-api-id abc123xyz \
    --name my-authorizer \
    --type TOKEN \
    --authorizer-uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:123456789012:function:my-auth-function/invocations \
    --identity-source method.request.header.Authorization \
    --authorizer-result-ttl-in-seconds 300
```

**Expected Output:**
```json
{
    "id": "auth123",
    "name": "my-authorizer",
    "type": "TOKEN",
    "authorizerUri": "...",
    "identitySource": "method.request.header.Authorization",
    "authorizerResultTtlInSeconds": 300
}
```

**Why:** Custom authentication/authorization logic using Lambda.

**When to Use:** Custom auth schemes, validating JWT tokens, third-party auth.

**Authorizer Types:**
- **TOKEN**: Token-based (e.g., JWT in Authorization header)
- **REQUEST**: Request parameter-based

**Lambda Authorizer Function (Python):**
```python
def lambda_handler(event, context):
    token = event['authorizationToken']

    # Validate token (your logic)
    if token == 'valid-token':
        return generate_policy('user123', 'Allow', event['methodArn'])
    else:
        return generate_policy('user123', 'Deny', event['methodArn'])

def generate_policy(principal_id, effect, resource):
    return {
        'principalId': principal_id,
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [{
                'Action': 'execute-api:Invoke',
                'Effect': effect,
                'Resource': resource
            }]
        }
    }
```

---

### 2. Create Cognito Authorizer (REST API)

```bash
aws apigateway create-authorizer \
    --rest-api-id abc123xyz \
    --name cognito-authorizer \
    --type COGNITO_USER_POOLS \
    --provider-arns arn:aws:cognito-idp:us-east-1:123456789012:userpool/us-east-1_ABC123 \
    --identity-source method.request.header.Authorization
```

**Expected Output:**
```json
{
    "id": "authCognito123",
    "name": "cognito-authorizer",
    "type": "COGNITO_USER_POOLS",
    "providerARNs": ["arn:aws:cognito-idp:us-east-1:123456789012:userpool/us-east-1_ABC123"],
    "identitySource": "method.request.header.Authorization"
}
```

**Why:** Use Cognito User Pool for authentication.

**When to Use:** User authentication with Cognito, OAuth 2.0/OIDC.

---

### 3. Attach Authorizer to Method

```bash
aws apigateway update-method \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --patch-operations op=replace,path=/authorizationType,value=CUSTOM op=replace,path=/authorizerId,value=auth123
```

**Expected Output:**
```json
{
    "httpMethod": "GET",
    "authorizationType": "CUSTOM",
    "authorizerId": "auth123"
}
```

**Why:** Protect method with authorizer.

**When to Use:** Securing API endpoints.

---

### 4. Create JWT Authorizer (HTTP API)

```bash
aws apigatewayv2 create-authorizer \
    --api-id xyz789abc \
    --authorizer-type JWT \
    --identity-source '$request.header.Authorization' \
    --name my-jwt-authorizer \
    --jwt-configuration Audience=my-api,Issuer=https://cognito-idp.us-east-1.amazonaws.com/us-east-1_ABC123
```

**Expected Output:**
```json
{
    "AuthorizerId": "authHttp123",
    "AuthorizerType": "JWT",
    "IdentitySource": ["$request.header.Authorization"],
    "Name": "my-jwt-authorizer",
    "JwtConfiguration": {
        "Audience": ["my-api"],
        "Issuer": "https://cognito-idp.us-east-1.amazonaws.com/us-east-1_ABC123"
    }
}
```

**Why:** Validate JWT tokens (Cognito, Auth0, etc.).

**When to Use:** HTTP API authentication with OAuth 2.0/OIDC.

---

### 5. Enable IAM Authorization

```bash
aws apigateway update-method \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --patch-operations op=replace,path=/authorizationType,value=AWS_IAM
```

**Expected Output:**
```json
{
    "httpMethod": "GET",
    "authorizationType": "AWS_IAM"
}
```

**Why:** Require AWS signature (SigV4) for requests.

**When to Use:** Internal APIs, AWS service integrations, high security.

---

## Stages & Deployments

### 1. Create Deployment (REST API)

```bash
aws apigateway create-deployment \
    --rest-api-id abc123xyz \
    --stage-name dev \
    --stage-description "Development stage" \
    --description "Initial deployment"
```

**Expected Output:**
```json
{
    "id": "deploy123",
    "description": "Initial deployment",
    "createdDate": "2025-11-17T15:00:00+00:00"
}
```

**Why:** Make API changes live. Creates immutable snapshot.

**When to Use:** After creating/modifying resources, methods, integrations.

**Note:** Changes to API aren't live until deployed.

---

### 2. Create Stage (REST API)

```bash
aws apigateway create-stage \
    --rest-api-id abc123xyz \
    --stage-name production \
    --deployment-id deploy123 \
    --description "Production stage"
```

**Expected Output:**
```json
{
    "stageName": "production",
    "deploymentId": "deploy123",
    "description": "Production stage",
    "createdDate": "2025-11-17T15:05:00+00:00",
    "methodSettings": {}
}
```

**Why:** Create environment for API (dev, staging, prod).

**When to Use:** Separating environments, versioning.

**API Endpoint:** `https://abc123xyz.execute-api.us-east-1.amazonaws.com/production`

---

### 3. Deploy to Existing Stage

```bash
aws apigateway create-deployment \
    --rest-api-id abc123xyz \
    --stage-name production \
    --description "Bug fixes and new features"
```

**Expected Output:**
```json
{
    "id": "deploy456",
    "description": "Bug fixes and new features",
    "createdDate": "2025-11-17T15:10:00+00:00"
}
```

**Why:** Update existing stage with new deployment.

**When to Use:** Deploying changes to production/staging.

---

### 4. List Stages

```bash
aws apigateway get-stages --rest-api-id abc123xyz
```

**Expected Output:**
```json
{
    "item": [
        {
            "stageName": "production",
            "deploymentId": "deploy456",
            "description": "Production stage",
            "createdDate": "2025-11-17T15:05:00+00:00"
        },
        {
            "stageName": "dev",
            "deploymentId": "deploy123",
            "createdDate": "2025-11-17T15:00:00+00:00"
        }
    ]
}
```

**Why:** View all stages for API.

**When to Use:** Managing environments, finding stage names.

---

### 5. Update Stage Settings

```bash
aws apigateway update-stage \
    --rest-api-id abc123xyz \
    --stage-name production \
    --patch-operations \
        op=replace,path=/description,value="Updated production stage" \
        op=replace,path=/cacheClusterEnabled,value=true \
        op=replace,path=/cacheClusterSize,value=0.5
```

**Expected Output:**
```json
{
    "stageName": "production",
    "description": "Updated production stage",
    "cacheClusterEnabled": true,
    "cacheClusterSize": "0.5"
}
```

**Why:** Configure stage-specific settings (caching, throttling, logging).

**When to Use:** Performance optimization, monitoring configuration.

---

### 6. Enable Caching

```bash
aws apigateway update-stage \
    --rest-api-id abc123xyz \
    --stage-name production \
    --patch-operations \
        op=replace,path=/cacheClusterEnabled,value=true \
        op=replace,path=/cacheClusterSize,value=0.5
```

**Why:** Cache responses to reduce backend calls and latency.

**When to Use:** Frequently accessed data, reducing costs, improving performance.

**Cache Sizes:** 0.5GB, 1.6GB, 6.1GB, 13.5GB, 28.4GB, 58.2GB, 118GB, 237GB

---

### 7. Configure Throttling

```bash
aws apigateway update-stage \
    --rest-api-id abc123xyz \
    --stage-name production \
    --patch-operations \
        op=replace,path=/throttle/rateLimit,value=1000 \
        op=replace,path=/throttle/burstLimit,value=2000
```

**Expected Output:**
```json
{
    "stageName": "production",
    "throttle": {
        "rateLimit": 1000.0,
        "burstLimit": 2000
    }
}
```

**Why:** Protect backend from overload, control costs.

**When to Use:** Production APIs, preventing abuse.

**Limits:**
- **Rate Limit**: Steady-state requests per second
- **Burst Limit**: Maximum concurrent requests

---

### 8. Deploy HTTP API

```bash
aws apigatewayv2 create-deployment --api-id xyz789abc
```

**Expected Output:**
```json
{
    "DeploymentId": "deployHttp123",
    "DeploymentStatus": "DEPLOYED",
    "CreatedDate": "2025-11-17T15:20:00+00:00"
}
```

**Why:** Deploy HTTP API changes.

**When to Use:** After modifying routes/integrations.

**Note:** HTTP APIs have automatic deployments if auto-deploy is enabled.

---

### 9. Create Stage (HTTP API)

```bash
aws apigatewayv2 create-stage \
    --api-id xyz789abc \
    --stage-name production \
    --auto-deploy
```

**Expected Output:**
```json
{
    "StageName": "production",
    "AutoDeploy": true,
    "CreatedDate": "2025-11-17T15:25:00+00:00"
}
```

**Why:** Create stage with auto-deployment.

**When to Use:** Simplified deployment workflow.

**API Endpoint:** `https://xyz789abc.execute-api.us-east-1.amazonaws.com/production`

---

### 10. Enable Stage Variables

```bash
aws apigateway update-stage \
    --rest-api-id abc123xyz \
    --stage-name production \
    --patch-operations \
        op=replace,path=/variables/lambdaAlias,value=prod \
        op=replace,path=/variables/environment,value=production
```

**Expected Output:**
```json
{
    "stageName": "production",
    "variables": {
        "lambdaAlias": "prod",
        "environment": "production"
    }
}
```

**Why:** Pass stage-specific configuration to integrations.

**When to Use:** Different backends per stage, environment configuration.

**Usage in Integration:**
```
arn:aws:lambda:us-east-1:123456789012:function:my-function:${stageVariables.lambdaAlias}
```

---

## Custom Domains

### 1. Create Custom Domain

```bash
aws apigateway create-domain-name \
    --domain-name api.example.com \
    --certificate-arn arn:aws:acm:us-east-1:123456789012:certificate/abc123 \
    --endpoint-configuration types=REGIONAL \
    --security-policy TLS_1_2
```

**Expected Output:**
```json
{
    "domainName": "api.example.com",
    "certificateArn": "arn:aws:acm:us-east-1:123456789012:certificate/abc123",
    "regionalDomainName": "d-abc123xyz.execute-api.us-east-1.amazonaws.com",
    "regionalHostedZoneId": "Z2FDTNDATAQYW2",
    "endpointConfiguration": {
        "types": ["REGIONAL"]
    },
    "securityPolicy": "TLS_1_2"
}
```

**Why:** Use custom domain instead of default API Gateway domain.

**When to Use:** Professional APIs, branding, easier to remember.

**Prerequisites:**
- ACM certificate for domain
- DNS provider (Route 53, etc.)

---

### 2. Create Base Path Mapping

```bash
aws apigateway create-base-path-mapping \
    --domain-name api.example.com \
    --rest-api-id abc123xyz \
    --stage production \
    --base-path v1
```

**Expected Output:**
```json
{
    "basePath": "v1",
    "restApiId": "abc123xyz",
    "stage": "production"
}
```

**Why:** Map API and stage to custom domain path.

**When to Use:** API versioning, multiple APIs on one domain.

**Result:** `https://api.example.com/v1/users`

---

### 3. Configure DNS (Route 53)

```bash
aws route53 change-resource-record-sets \
    --hosted-zone-id Z123456ABCDEFG \
    --change-batch file://dns-change.json
```

**dns-change.json:**
```json
{
    "Changes": [{
        "Action": "CREATE",
        "ResourceRecordSet": {
            "Name": "api.example.com",
            "Type": "A",
            "AliasTarget": {
                "HostedZoneId": "Z2FDTNDATAQYW2",
                "DNSName": "d-abc123xyz.execute-api.us-east-1.amazonaws.com",
                "EvaluateTargetHealth": false
            }
        }
    }]
}
```

**Why:** Point custom domain to API Gateway endpoint.

**When to Use:** Completing custom domain setup.

---

## Usage Plans & API Keys

### 1. Create API Key

```bash
aws apigateway create-api-key \
    --name my-api-key \
    --description "API key for partner integration" \
    --enabled
```

**Expected Output:**
```json
{
    "id": "apikey123",
    "name": "my-api-key",
    "description": "API key for partner integration",
    "enabled": true,
    "createdDate": "2025-11-17T16:00:00+00:00",
    "value": "a1b2c3d4e5f6g7h8i9j0klmnopqrst"
}
```

**Why:** Create API key for client identification.

**When to Use:** Tracking API usage per client, monetization.

---

### 2. Create Usage Plan

```bash
aws apigateway create-usage-plan \
    --name basic-plan \
    --description "Basic usage plan" \
    --throttle rateLimit=100,burstLimit=200 \
    --quota limit=10000,period=MONTH \
    --api-stages apiId=abc123xyz,stage=production
```

**Expected Output:**
```json
{
    "id": "plan123",
    "name": "basic-plan",
    "description": "Basic usage plan",
    "throttle": {
        "rateLimit": 100.0,
        "burstLimit": 200
    },
    "quota": {
        "limit": 10000,
        "period": "MONTH"
    },
    "apiStages": [
        {
            "apiId": "abc123xyz",
            "stage": "production"
        }
    ]
}
```

**Why:** Define throttling and quota limits for API keys.

**When to Use:** Rate limiting, API monetization, tiered access.

---

### 3. Associate API Key with Usage Plan

```bash
aws apigateway create-usage-plan-key \
    --usage-plan-id plan123 \
    --key-id apikey123 \
    --key-type API_KEY
```

**Expected Output:**
```json
{
    "id": "apikey123",
    "type": "API_KEY",
    "name": "my-api-key"
}
```

**Why:** Link API key to usage plan limits.

**When to Use:** Enforcing limits per key.

---

### 4. Require API Key for Method

```bash
aws apigateway update-method \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method GET \
    --patch-operations op=replace,path=/apiKeyRequired,value=true
```

**Expected Output:**
```json
{
    "httpMethod": "GET",
    "apiKeyRequired": true
}
```

**Why:** Require API key in request header.

**When to Use:** Protecting endpoints, tracking usage.

**Client Usage:**
```bash
curl -H "x-api-key: a1b2c3d4e5f6g7h8i9j0klmnopqrst" https://api.example.com/users
```

---

### 5. Get API Key Usage

```bash
aws apigateway get-usage \
    --usage-plan-id plan123 \
    --key-id apikey123 \
    --start-date 2025-11-01 \
    --end-date 2025-11-30
```

**Expected Output:**
```json
{
    "usagePlanId": "plan123",
    "startDate": "2025-11-01",
    "endDate": "2025-11-30",
    "items": {
        "abc123xyz:production": [
            [0, 150],
            [0, 200],
            ...
        ]
    }
}
```

**Why:** Monitor API key usage.

**When to Use:** Billing, monitoring, quota management.

---

## CORS Configuration

### 1. Enable CORS (REST API Manual)

```bash
# Add OPTIONS method
aws apigateway put-method \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method OPTIONS \
    --authorization-type NONE

# Mock integration for OPTIONS
aws apigateway put-integration \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method OPTIONS \
    --type MOCK \
    --request-templates '{"application/json": "{\"statusCode\": 200}"}'

# Method response for OPTIONS
aws apigateway put-method-response \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method OPTIONS \
    --status-code 200 \
    --response-parameters \
        method.response.header.Access-Control-Allow-Headers=true \
        method.response.header.Access-Control-Allow-Methods=true \
        method.response.header.Access-Control-Allow-Origin=true

# Integration response for OPTIONS
aws apigateway put-integration-response \
    --rest-api-id abc123xyz \
    --resource-id res456 \
    --http-method OPTIONS \
    --status-code 200 \
    --response-parameters \
        method.response.header.Access-Control-Allow-Headers="'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'" \
        method.response.header.Access-Control-Allow-Methods="'GET,POST,PUT,DELETE,OPTIONS'" \
        method.response.header.Access-Control-Allow-Origin="'*'"
```

**Why:** Allow cross-origin requests from browsers.

**When to Use:** Web applications calling API from different domain.

---

### 2. Enable CORS (HTTP API - Simple)

```bash
aws apigatewayv2 update-api \
    --api-id xyz789abc \
    --cors-configuration AllowOrigins="*",AllowMethods="GET,POST,PUT,DELETE",AllowHeaders="Content-Type,Authorization"
```

**Expected Output:**
```json
{
    "ApiId": "xyz789abc",
    "CorsConfiguration": {
        "AllowOrigins": ["*"],
        "AllowMethods": ["GET", "POST", "PUT", "DELETE"],
        "AllowHeaders": ["Content-Type", "Authorization"]
    }
}
```

**Why:** Easily enable CORS on HTTP API (much simpler than REST API).

**When to Use:** HTTP APIs with browser clients.

---

## WebSocket APIs

### 1. Create WebSocket API

```bash
aws apigatewayv2 create-api \
    --name my-websocket-api \
    --protocol-type WEBSOCKET \
    --route-selection-expression '$request.body.action'
```

**Expected Output:**
```json
{
    "ApiId": "ws123abc",
    "Name": "my-websocket-api",
    "ProtocolType": "WEBSOCKET",
    "RouteSelectionExpression": "$request.body.action",
    "ApiEndpoint": "wss://ws123abc.execute-api.us-east-1.amazonaws.com/production"
}
```

**Why:** Create real-time bidirectional API.

**When to Use:** Chat apps, gaming, live dashboards, notifications.

**Route Selection:** Expression to determine which route handles message.

---

### 2. Create WebSocket Route

```bash
aws apigatewayv2 create-route \
    --api-id ws123abc \
    --route-key sendMessage \
    --target integrations/integWs123
```

**Expected Output:**
```json
{
    "RouteId": "routeWs456",
    "RouteKey": "sendMessage",
    "Target": "integrations/integWs123"
}
```

**Why:** Define route for handling specific messages.

**When to Use:** Different actions in WebSocket protocol.

**Special Routes:**
- **$connect**: Client connects
- **$disconnect**: Client disconnects
- **$default**: Default route for unmatched messages

---

### 3. Create WebSocket Integration

```bash
aws apigatewayv2 create-integration \
    --api-id ws123abc \
    --integration-type AWS_PROXY \
    --integration-uri arn:aws:lambda:us-east-1:123456789012:function:ws-handler
```

**Expected Output:**
```json
{
    "IntegrationId": "integWs123",
    "IntegrationType": "AWS_PROXY",
    "IntegrationUri": "arn:aws:lambda:us-east-1:123456789012:function:ws-handler"
}
```

**Why:** Connect route to Lambda backend.

**When to Use:** Implementing WebSocket logic.

---

### 4. Send Message to Connection (from Lambda)

**Python Lambda Function:**
```python
import boto3

apigateway_management = boto3.client(
    'apigatewaymanagementapi',
    endpoint_url='https://ws123abc.execute-api.us-east-1.amazonaws.com/production'
)

def lambda_handler(event, context):
    connection_id = event['requestContext']['connectionId']

    # Send message to client
    apigateway_management.post_to_connection(
        ConnectionId=connection_id,
        Data='Hello from server!'
    )

    return {'statusCode': 200}
```

**Why:** Send messages from server to clients.

**When to Use:** Broadcasting, notifications, real-time updates.

---

### 5. Store Connection IDs (DynamoDB)

**$connect route handler:**
```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('WebSocketConnections')

def lambda_handler(event, context):
    connection_id = event['requestContext']['connectionId']

    # Store connection
    table.put_item(Item={'connectionId': connection_id})

    return {'statusCode': 200}
```

**$disconnect route handler:**
```python
def lambda_handler(event, context):
    connection_id = event['requestContext']['connectionId']

    # Remove connection
    table.delete_item(Key={'connectionId': connection_id})

    return {'statusCode': 200}
```

**Why:** Track active connections for broadcasting.

**When to Use:** Multi-client chat, notifications.

---

## Monitoring & Logging

### 1. Enable CloudWatch Logs (REST API)

```bash
# Create role for API Gateway logging (one-time setup)
aws apigateway update-account \
    --patch-operations op=replace,path=/cloudwatchRoleArn,value=arn:aws:iam::123456789012:role/APIGatewayCloudWatchRole

# Enable for stage
aws apigateway update-stage \
    --rest-api-id abc123xyz \
    --stage-name production \
    --patch-operations \
        op=replace,path=/logging/loglevel,value=INFO \
        op=replace,path=/logging/dataTrace,value=true
```

**Expected Output:**
```json
{
    "stageName": "production",
    "logging": {
        "loglevel": "INFO",
        "dataTrace": true
    }
}
```

**Why:** Log all requests/responses for debugging and monitoring.

**When to Use:** Troubleshooting, security auditing, compliance.

**Log Levels:** OFF, ERROR, INFO

---

### 2. Enable Access Logging

```bash
aws apigateway update-stage \
    --rest-api-id abc123xyz \
    --stage-name production \
    --patch-operations \
        op=replace,path=/accessLogSettings/destinationArn,value=arn:aws:logs:us-east-1:123456789012:log-group:/aws/apigateway/my-api \
        op=replace,path=/accessLogSettings/format,value='$context.requestId $context.error.message $context.error.messageString'
```

**Expected Output:**
```json
{
    "accessLogSettings": {
        "destinationArn": "arn:aws:logs:us-east-1:123456789012:log-group:/aws/apigateway/my-api",
        "format": "$context.requestId $context.error.message $context.error.messageString"
    }
}
```

**Why:** Custom log format for access logs.

**When to Use:** Custom analytics, compliance, monitoring.

**Format Variables:**
- `$context.requestId`
- `$context.error.message`
- `$context.integration.latency`
- `$context.status`

---

### 3. Enable X-Ray Tracing

```bash
aws apigateway update-stage \
    --rest-api-id abc123xyz \
    --stage-name production \
    --patch-operations op=replace,path=/tracingEnabled,value=true
```

**Expected Output:**
```json
{
    "stageName": "production",
    "tracingEnabled": true
}
```

**Why:** Distributed tracing across API Gateway and downstream services.

**When to Use:** Performance analysis, debugging complex workflows.

---

### 4. View API Gateway Metrics

```bash
aws cloudwatch get-metric-statistics \
    --namespace AWS/ApiGateway \
    --metric-name Count \
    --dimensions Name=ApiName,Value=my-rest-api Name=Stage,Value=production \
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
            "Sum": 1250.0,
            "Unit": "Count"
        }
    ]
}
```

**Why:** Monitor API usage, performance, errors.

**When to Use:** Performance analysis, capacity planning, alerting.

**Key Metrics:**
- **Count**: Total API requests
- **4XXError**: Client-side errors
- **5XXError**: Server-side errors
- **Latency**: Request latency
- **IntegrationLatency**: Backend latency
- **CacheHitCount**: Cache hits
- **CacheMissCount**: Cache misses

---

## Best Practices

### Security
1. **Use authorizers** - Don't rely on API keys alone
2. **Enable throttling** - Prevent abuse and DDoS
3. **HTTPS only** - Never use HTTP
4. **Validate requests** - Use request validators
5. **Enable CORS properly** - Don't use wildcard (*) in production
6. **Use WAF** - Web Application Firewall for advanced protection
7. **Rotate API keys** - Regular rotation
8. **CloudWatch Logs** - Monitor for suspicious activity

### Performance
1. **Enable caching** - Reduce backend load
2. **Use HTTP APIs** - Lower latency than REST APIs
3. **Optimize Lambda cold starts** - Use provisioned concurrency
4. **Enable compression** - Reduce payload size
5. **Use regional endpoints** - Lower latency for regional users
6. **Monitor integration latency** - Optimize slow backends

### Cost Optimization
1. **Use HTTP APIs** - 71% cheaper than REST APIs
2. **Enable caching** - Reduce Lambda invocations
3. **Set appropriate quotas** - Prevent runaway costs
4. **Monitor usage** - Set billing alarms
5. **Use REST API only when needed** - HTTP API for simple use cases

### Development
1. **Use stages** - Separate dev/staging/prod
2. **Version APIs** - Use base path for versioning (v1, v2)
3. **Document APIs** - Export OpenAPI/Swagger
4. **Use stage variables** - Environment-specific config
5. **Test before deploy** - Use test invoke
6. **Blue/green deployments** - Use canary deployments

### Reliability
1. **Set timeouts** - Prevent hanging requests
2. **Handle errors gracefully** - Custom error responses
3. **Use DLQ for async** - Capture failed events
4. **Monitor error rates** - Set CloudWatch alarms
5. **Multi-region** - High availability

---

## Common Patterns

### 1. Lambda Proxy Integration (Recommended)

**Advantages:**
- Simplest integration
- Full control in Lambda
- Easy to test
- Less configuration

**Lambda Response Format:**
```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps({'message': 'Success'})
    }
```

### 2. CRUD API with DynamoDB

```bash
# Create API
aws apigateway create-rest-api --name crud-api

# Create resource /items
aws apigateway create-resource --rest-api-id API_ID --parent-id ROOT_ID --path-part items

# Create resource /items/{id}
aws apigateway create-resource --rest-api-id API_ID --parent-id ITEMS_ID --path-part '{id}'

# Add methods: GET (list), POST (create), GET (read), PUT (update), DELETE
# Each method integrated with separate Lambda function
```

### 3. Authentication Flow

```
1. User logs in → Cognito User Pool
2. Receives JWT token
3. Calls API with Authorization header
4. API Gateway validates token (Cognito authorizer)
5. If valid, forwards to Lambda
6. Lambda processes request
7. Response returned to user
```

---

## Troubleshooting

### Integration Timeout
- Check Lambda execution time
- Increase integration timeout (max 29 seconds)
- Optimize Lambda performance

### CORS Errors
- Verify OPTIONS method configured
- Check response headers
- Ensure Lambda returns CORS headers (proxy integration)

### Authorization Errors
- Verify authorizer configuration
- Check IAM policies
- Test token/signature validity

### 500 Internal Server Error
- Check Lambda errors in CloudWatch Logs
- Verify IAM execution role
- Check integration configuration

---

## Quick Reference

```bash
# REST API
aws apigateway create-rest-api --name API_NAME
aws apigateway get-resources --rest-api-id API_ID
aws apigateway create-deployment --rest-api-id API_ID --stage-name STAGE

# HTTP API
aws apigatewayv2 create-api --name API_NAME --protocol-type HTTP
aws apigatewayv2 create-route --api-id API_ID --route-key "GET /path"
aws apigatewayv2 create-deployment --api-id API_ID

# Test
curl https://API_ID.execute-api.REGION.amazonaws.com/STAGE/path
```

---

## Summary

API Gateway provides a complete platform for building, deploying, and managing APIs. Master these commands to:
- Build RESTful APIs
- Create real-time WebSocket applications
- Secure APIs with multiple auth methods
- Scale automatically
- Monitor and optimize performance

Practice creating APIs, integrating with Lambda, and deploying to stages. API Gateway's flexibility makes it ideal for serverless architectures.

For LocalStack: add `--endpoint-url=http://localhost:4566` to commands.

Happy API building!
