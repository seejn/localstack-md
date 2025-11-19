# AWS Cognito - Complete Practical Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Key Concepts](#key-concepts)
3. [User Pools - Management](#user-pools---management)
4. [User Pool Clients](#user-pool-clients)
5. [User Management](#user-management)
6. [Authentication & Sign-In](#authentication--sign-in)
7. [User Attributes & Verification](#user-attributes--verification)
8. [Groups & Permissions](#groups--permissions)
9. [Multi-Factor Authentication (MFA)](#multi-factor-authentication-mfa)
10. [Password Policies](#password-policies)
11. [Identity Pools (Federated Identities)](#identity-pools-federated-identities)
12. [Social & Enterprise Identity Providers](#social--enterprise-identity-providers)
13. [Lambda Triggers](#lambda-triggers)
14. [Token Management](#token-management)
15. [Best Practices](#best-practices)

---

## Introduction

Amazon Cognito provides authentication, authorization, and user management for web and mobile applications. Users can sign in directly or through social identity providers.

**Two Main Components:**
1. **User Pools**: User directory for sign-up/sign-in
2. **Identity Pools**: AWS credentials for accessing AWS services

**Use Cases:**
- User authentication and authorization
- Social identity federation (Google, Facebook, etc.)
- Enterprise identity federation (SAML, OIDC)
- Multi-factor authentication (MFA)
- User profile management
- Serverless authentication

**Benefits:**
- Fully managed service
- Secure and scalable
- Standards-based (OAuth 2.0, OIDC, SAML)
- Integration with API Gateway and ALB
- Customizable with Lambda triggers

---

## Key Concepts

### User Pools
- User directory service
- Sign-up and sign-in functionality
- Built-in UI or custom UI
- Password policies and MFA
- Email/SMS verification
- Social and enterprise IdP federation
- Returns JWT tokens (ID, Access, Refresh)

### Identity Pools
- Grant AWS credentials to users
- Support authenticated and unauthenticated access
- Integrate with User Pools and external IdPs
- Control access to AWS services via IAM roles
- Returns temporary AWS credentials

### Tokens (User Pool)
- **ID Token**: Contains user identity claims
- **Access Token**: Used to authorize API calls
- **Refresh Token**: Used to get new ID/Access tokens

### Attributes
- User profile data (email, name, etc.)
- Standard attributes (predefined)
- Custom attributes (your own fields)
- Required vs. optional
- Mutable vs. immutable

---

## User Pools - Management

### 1. Create User Pool

```bash
aws cognito-idp create-user-pool \
    --pool-name my-user-pool \
    --policies '{
        "PasswordPolicy": {
            "MinimumLength": 8,
            "RequireUppercase": true,
            "RequireLowercase": true,
            "RequireNumbers": true,
            "RequireSymbols": true
        }
    }' \
    --auto-verified-attributes email \
    --username-attributes email \
    --schema '[
        {
            "Name": "email",
            "AttributeDataType": "String",
            "Required": true,
            "Mutable": true
        }
    ]'
```

**Expected Output:**
```json
{
    "UserPool": {
        "Id": "us-east-1_ABC123",
        "Name": "my-user-pool",
        "Policies": {
            "PasswordPolicy": {
                "MinimumLength": 8,
                "RequireUppercase": true,
                "RequireLowercase": true,
                "RequireNumbers": true,
                "RequireSymbols": true
            }
        },
        "AutoVerifiedAttributes": ["email"],
        "UsernameAttributes": ["email"],
        "CreationDate": "2025-11-17T10:00:00+00:00"
    }
}
```

**Why:** Create user directory for authentication.

**When to Use:** Starting new application with user auth, replacing custom auth system.

**Username Options:**
- Default: Username (unique identifier)
- Email: Use email as username
- Phone: Use phone number as username
- Both: Email or phone

**LocalStack:**
```bash
aws cognito-idp create-user-pool --pool-name my-user-pool --endpoint-url=http://localhost:4566
```

---

### 2. List User Pools

```bash
aws cognito-idp list-user-pools --max-results 10
```

**Expected Output:**
```json
{
    "UserPools": [
        {
            "Id": "us-east-1_ABC123",
            "Name": "my-user-pool",
            "CreationDate": "2025-11-17T10:00:00+00:00",
            "LastModifiedDate": "2025-11-17T10:00:00+00:00"
        }
    ]
}
```

**Why:** View all user pools in account/region.

**When to Use:** Finding pool IDs, inventory, auditing.

---

### 3. Describe User Pool

```bash
aws cognito-idp describe-user-pool --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```json
{
    "UserPool": {
        "Id": "us-east-1_ABC123",
        "Name": "my-user-pool",
        "Policies": {
            "PasswordPolicy": {
                "MinimumLength": 8,
                "RequireUppercase": true
            }
        },
        "AutoVerifiedAttributes": ["email"],
        "UsernameAttributes": ["email"],
        "EstimatedNumberOfUsers": 0
    }
}
```

**Why:** Get detailed configuration of user pool.

**When to Use:** Checking settings, troubleshooting, documentation.

---

### 4. Update User Pool

```bash
aws cognito-idp update-user-pool \
    --user-pool-id us-east-1_ABC123 \
    --policies '{
        "PasswordPolicy": {
            "MinimumLength": 12,
            "RequireUppercase": true,
            "RequireLowercase": true,
            "RequireNumbers": true,
            "RequireSymbols": true
        }
    }'
```

**Expected Output:**
```
(No output on success)
```

**Why:** Modify user pool settings.

**When to Use:** Updating policies, changing verification settings.

**Note:** Some settings cannot be changed after creation (e.g., username attributes).

---

### 5. Delete User Pool

```bash
aws cognito-idp delete-user-pool --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove user pool (deletes all users and configuration).

**When to Use:** Cleanup, testing, decommissioning.

**Warning:** Irreversible operation. All users will be deleted.

---

### 6. Add Custom Domain

```bash
aws cognito-idp create-user-pool-domain \
    --domain my-auth-domain \
    --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```json
{
    "CloudFrontDomain": "my-auth-domain.auth.us-east-1.amazoncognito.com"
}
```

**Why:** Custom domain for hosted UI.

**When to Use:** Branding, custom login pages.

**Hosted UI URL:** `https://my-auth-domain.auth.us-east-1.amazoncognito.com/login`

---

### 7. Use Custom Domain (with ACM certificate)

```bash
aws cognito-idp create-user-pool-domain \
    --domain auth.example.com \
    --user-pool-id us-east-1_ABC123 \
    --custom-domain-config CertificateArn=arn:aws:acm:us-east-1:123456789012:certificate/abc123
```

**Expected Output:**
```json
{
    "CloudFrontDomain": "d123abc456def.cloudfront.net"
}
```

**Why:** Use your own domain for hosted UI.

**When to Use:** Professional branding, custom domain requirements.

**Requires:** ACM certificate in us-east-1, DNS CNAME to CloudFront domain.

---

## User Pool Clients

### 1. Create User Pool Client

```bash
aws cognito-idp create-user-pool-client \
    --user-pool-id us-east-1_ABC123 \
    --client-name my-web-app \
    --generate-secret \
    --allowed-o-auth-flows authorization_code implicit \
    --allowed-o-auth-scopes openid email profile \
    --allowed-o-auth-flows-user-pool-client \
    --callback-urls https://example.com/callback \
    --supported-identity-providers COGNITO
```

**Expected Output:**
```json
{
    "UserPoolClient": {
        "ClientId": "1a2b3c4d5e6f7g8h9i0j",
        "ClientName": "my-web-app",
        "UserPoolId": "us-east-1_ABC123",
        "ClientSecret": "secret123abc456def",
        "GenerateSecret": true,
        "AllowedOAuthFlows": ["authorization_code", "implicit"],
        "AllowedOAuthScopes": ["openid", "email", "profile"],
        "CallbackURLs": ["https://example.com/callback"],
        "SupportedIdentityProviders": ["COGNITO"]
    }
}
```

**Why:** Register application to use user pool.

**When to Use:** Each application (web, mobile, API) needs a client.

**OAuth Flows:**
- **authorization_code**: Server-side apps (most secure)
- **implicit**: Client-side apps (SPAs) - less secure
- **client_credentials**: Machine-to-machine

**OAuth Scopes:**
- **openid**: ID token
- **email**: Email access
- **profile**: Profile access
- **aws.cognito.signin.user.admin**: All user attributes

---

### 2. List User Pool Clients

```bash
aws cognito-idp list-user-pool-clients --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```json
{
    "UserPoolClients": [
        {
            "ClientId": "1a2b3c4d5e6f7g8h9i0j",
            "ClientName": "my-web-app",
            "UserPoolId": "us-east-1_ABC123"
        }
    ]
}
```

**Why:** View all registered clients for user pool.

**When to Use:** Finding client IDs, auditing applications.

---

### 3. Describe User Pool Client

```bash
aws cognito-idp describe-user-pool-client \
    --user-pool-id us-east-1_ABC123 \
    --client-id 1a2b3c4d5e6f7g8h9i0j
```

**Expected Output:**
```json
{
    "UserPoolClient": {
        "ClientId": "1a2b3c4d5e6f7g8h9i0j",
        "ClientName": "my-web-app",
        "ClientSecret": "secret123abc456def",
        "AllowedOAuthFlows": ["authorization_code"],
        "CallbackURLs": ["https://example.com/callback"]
    }
}
```

**Why:** Get client configuration (including secret).

**When to Use:** Integration setup, troubleshooting.

---

### 4. Update User Pool Client

```bash
aws cognito-idp update-user-pool-client \
    --user-pool-id us-east-1_ABC123 \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --callback-urls https://example.com/callback https://example.com/auth
```

**Expected Output:**
```json
{
    "UserPoolClient": {
        "ClientId": "1a2b3c4d5e6f7g8h9i0j",
        "CallbackURLs": ["https://example.com/callback", "https://example.com/auth"]
    }
}
```

**Why:** Modify client settings (URLs, scopes, etc.).

**When to Use:** Updating application configuration.

---

### 5. Delete User Pool Client

```bash
aws cognito-idp delete-user-pool-client \
    --user-pool-id us-east-1_ABC123 \
    --client-id 1a2b3c4d5e6f7g8h9i0j
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove application client.

**When to Use:** Decommissioning application, cleanup.

---

## User Management

### 1. Admin Create User

```bash
aws cognito-idp admin-create-user \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com \
    --user-attributes Name=email,Value=john@example.com Name=email_verified,Value=true Name=name,Value="John Doe" \
    --temporary-password TempPass123! \
    --message-action SUPPRESS
```

**Expected Output:**
```json
{
    "User": {
        "Username": "john@example.com",
        "Attributes": [
            {
                "Name": "email",
                "Value": "john@example.com"
            },
            {
                "Name": "email_verified",
                "Value": "true"
            },
            {
                "Name": "name",
                "Value": "John Doe"
            }
        ],
        "UserCreateDate": "2025-11-17T10:30:00+00:00",
        "UserStatus": "FORCE_CHANGE_PASSWORD",
        "Enabled": true
    }
}
```

**Why:** Create user with admin privileges (bypass sign-up flow).

**When to Use:** Bulk user import, administrative user creation.

**User Status:**
- **FORCE_CHANGE_PASSWORD**: Temporary password, must change on first login
- **CONFIRMED**: Account verified and active

**Message Action:**
- **RESEND**: Send welcome email/SMS
- **SUPPRESS**: Don't send message

---

### 2. List Users

```bash
aws cognito-idp list-users --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```json
{
    "Users": [
        {
            "Username": "john@example.com",
            "Attributes": [
                {
                    "Name": "email",
                    "Value": "john@example.com"
                }
            ],
            "UserCreateDate": "2025-11-17T10:30:00+00:00",
            "UserStatus": "CONFIRMED",
            "Enabled": true
        }
    ]
}
```

**Why:** View all users in pool.

**When to Use:** User management, auditing, bulk operations.

**Filter by attribute:**
```bash
aws cognito-idp list-users --user-pool-id us-east-1_ABC123 --filter 'email ^= "john"'
```

---

### 3. Admin Get User

```bash
aws cognito-idp admin-get-user \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com
```

**Expected Output:**
```json
{
    "Username": "john@example.com",
    "UserAttributes": [
        {
            "Name": "sub",
            "Value": "12345678-1234-1234-1234-123456789012"
        },
        {
            "Name": "email",
            "Value": "john@example.com"
        },
        {
            "Name": "email_verified",
            "Value": "true"
        }
    ],
    "UserCreateDate": "2025-11-17T10:30:00+00:00",
    "UserStatus": "CONFIRMED",
    "Enabled": true
}
```

**Why:** Get detailed user information.

**When to Use:** User support, troubleshooting, verification.

---

### 4. Admin Update User Attributes

```bash
aws cognito-idp admin-update-user-attributes \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com \
    --user-attributes Name=name,Value="John Smith" Name=phone_number,Value="+12345678901"
```

**Expected Output:**
```
(No output on success)
```

**Why:** Modify user attributes administratively.

**When to Use:** User support, data corrections, bulk updates.

---

### 5. Admin Delete User

```bash
aws cognito-idp admin-delete-user \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com
```

**Expected Output:**
```
(No output on success)
```

**Why:** Permanently delete user.

**When to Use:** GDPR compliance, user requests, cleanup.

**Warning:** Irreversible operation.

---

### 6. Admin Disable User

```bash
aws cognito-idp admin-disable-user \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com
```

**Expected Output:**
```
(No output on success)
```

**Why:** Temporarily disable user account (can be re-enabled).

**When to Use:** Account suspension, security incidents.

---

### 7. Admin Enable User

```bash
aws cognito-idp admin-enable-user \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com
```

**Expected Output:**
```
(No output on success)
```

**Why:** Re-enable disabled account.

**When to Use:** Reinstating suspended accounts.

---

### 8. Admin Reset User Password

```bash
aws cognito-idp admin-reset-user-password \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com
```

**Expected Output:**
```
(No output on success)
```

**Why:** Force password reset (sends verification code).

**When to Use:** User forgot password, security incidents.

**Result:** User status changes to RESET_REQUIRED, verification code sent.

---

### 9. Admin Set User Password

```bash
aws cognito-idp admin-set-user-password \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com \
    --password NewPass123! \
    --permanent
```

**Expected Output:**
```
(No output on success)
```

**Why:** Set user password without verification flow.

**When to Use:** Administrative password resets, bulk operations.

**Permanent:** If true, password doesn't need change on next login.

---

## Authentication & Sign-In

### 1. User Sign-Up (Self-Registration)

```bash
aws cognito-idp sign-up \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --username john@example.com \
    --password MyPass123! \
    --user-attributes Name=email,Value=john@example.com Name=name,Value="John Doe"
```

**Expected Output:**
```json
{
    "UserConfirmed": false,
    "UserSub": "12345678-1234-1234-1234-123456789012",
    "CodeDeliveryDetails": {
        "Destination": "j***@e***",
        "DeliveryMedium": "EMAIL",
        "AttributeName": "email"
    }
}
```

**Why:** User self-registration.

**When to Use:** Public sign-up flow.

**Result:** Verification code sent to email/phone (if auto-verify enabled).

---

### 2. Confirm Sign-Up

```bash
aws cognito-idp confirm-sign-up \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --username john@example.com \
    --confirmation-code 123456
```

**Expected Output:**
```
(No output on success)
```

**Why:** Verify email/phone with code sent during sign-up.

**When to Use:** Completing registration flow.

**Result:** User status changes to CONFIRMED.

---

### 3. Resend Confirmation Code

```bash
aws cognito-idp resend-confirmation-code \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --username john@example.com
```

**Expected Output:**
```json
{
    "CodeDeliveryDetails": {
        "Destination": "j***@e***",
        "DeliveryMedium": "EMAIL",
        "AttributeName": "email"
    }
}
```

**Why:** Send new verification code.

**When to Use:** User didn't receive code, code expired.

---

### 4. Initiate Auth (Sign-In)

```bash
aws cognito-idp initiate-auth \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --auth-flow USER_PASSWORD_AUTH \
    --auth-parameters USERNAME=john@example.com,PASSWORD=MyPass123!
```

**Expected Output:**
```json
{
    "AuthenticationResult": {
        "AccessToken": "eyJraWQiOiJ...",
        "ExpiresIn": 3600,
        "TokenType": "Bearer",
        "RefreshToken": "eyJjdHkiOiJ...",
        "IdToken": "eyJraWQiOiJ..."
    }
}
```

**Why:** User authentication (get tokens).

**When to Use:** Login flow.

**Auth Flows:**
- **USER_PASSWORD_AUTH**: Username/password
- **REFRESH_TOKEN_AUTH**: Refresh tokens
- **CUSTOM_AUTH**: Custom authentication

**Tokens:**
- **ID Token**: User identity (contains claims)
- **Access Token**: API authorization
- **Refresh Token**: Get new tokens (long-lived)

---

### 5. Admin Initiate Auth

```bash
aws cognito-idp admin-initiate-auth \
    --user-pool-id us-east-1_ABC123 \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --auth-flow ADMIN_NO_SRP_AUTH \
    --auth-parameters USERNAME=john@example.com,PASSWORD=MyPass123!
```

**Expected Output:**
```json
{
    "AuthenticationResult": {
        "AccessToken": "eyJraWQiOiJ...",
        "IdToken": "eyJraWQiOiJ...",
        "RefreshToken": "eyJjdHkiOiJ..."
    }
}
```

**Why:** Server-side authentication (bypasses SRP).

**When to Use:** Backend authentication, testing, server-to-server.

---

### 6. Refresh Tokens

```bash
aws cognito-idp initiate-auth \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --auth-flow REFRESH_TOKEN_AUTH \
    --auth-parameters REFRESH_TOKEN=eyJjdHkiOiJ...
```

**Expected Output:**
```json
{
    "AuthenticationResult": {
        "AccessToken": "eyJraWQiOiJ...",
        "IdToken": "eyJraWQiOiJ...",
        "ExpiresIn": 3600
    }
}
```

**Why:** Get new access/ID tokens without re-authenticating.

**When to Use:** Token expiration (access token expires in 1 hour by default).

---

### 7. Sign Out (Revoke Token)

```bash
aws cognito-idp global-sign-out \
    --access-token eyJraWQiOiJ...
```

**Expected Output:**
```
(No output on success)
```

**Why:** Sign out user from all devices.

**When to Use:** User logout, security incidents.

**Result:** All tokens for user are invalidated.

---

### 8. Forgot Password

```bash
aws cognito-idp forgot-password \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --username john@example.com
```

**Expected Output:**
```json
{
    "CodeDeliveryDetails": {
        "Destination": "j***@e***",
        "DeliveryMedium": "EMAIL",
        "AttributeName": "email"
    }
}
```

**Why:** Initiate password reset flow.

**When to Use:** User forgot password.

**Result:** Verification code sent to verified email/phone.

---

### 9. Confirm Forgot Password

```bash
aws cognito-idp confirm-forgot-password \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --username john@example.com \
    --confirmation-code 123456 \
    --password NewPass456!
```

**Expected Output:**
```
(No output on success)
```

**Why:** Complete password reset with verification code.

**When to Use:** User received code and wants to set new password.

---

### 10. Change Password

```bash
aws cognito-idp change-password \
    --access-token eyJraWQiOiJ... \
    --previous-password MyPass123! \
    --proposed-password NewPass456!
```

**Expected Output:**
```
(No output on success)
```

**Why:** Change password while authenticated.

**When to Use:** User wants to change password (knows current password).

---

## User Attributes & Verification

### 1. Get User Attributes

```bash
aws cognito-idp get-user --access-token eyJraWQiOiJ...
```

**Expected Output:**
```json
{
    "Username": "john@example.com",
    "UserAttributes": [
        {
            "Name": "sub",
            "Value": "12345678-1234-1234-1234-123456789012"
        },
        {
            "Name": "email",
            "Value": "john@example.com"
        },
        {
            "Name": "email_verified",
            "Value": "true"
        }
    ]
}
```

**Why:** Get current user's profile.

**When to Use:** Displaying user profile, checking verification status.

---

### 2. Update User Attributes

```bash
aws cognito-idp update-user-attributes \
    --access-token eyJraWQiOiJ... \
    --user-attributes Name=name,Value="John Smith" Name=phone_number,Value="+12345678901"
```

**Expected Output:**
```json
{
    "CodeDeliveryDetailsList": [
        {
            "AttributeName": "phone_number",
            "DeliveryMedium": "SMS",
            "Destination": "+***901"
        }
    ]
}
```

**Why:** User updates their own profile.

**When to Use:** Profile editing.

**Note:** Changing email/phone triggers verification.

---

### 3. Verify User Attribute

```bash
aws cognito-idp verify-user-attribute \
    --access-token eyJraWQiOiJ... \
    --attribute-name email \
    --code 123456
```

**Expected Output:**
```
(No output on success)
```

**Why:** Verify new email/phone number.

**When to Use:** After updating email/phone attribute.

---

### 4. Get User Attribute Verification Code

```bash
aws cognito-idp get-user-attribute-verification-code \
    --access-token eyJraWQiOiJ... \
    --attribute-name email
```

**Expected Output:**
```json
{
    "CodeDeliveryDetails": {
        "AttributeName": "email",
        "DeliveryMedium": "EMAIL",
        "Destination": "j***@e***"
    }
}
```

**Why:** Request verification code for attribute.

**When to Use:** User needs new verification code.

---

### 5. Delete User Attributes

```bash
aws cognito-idp delete-user-attributes \
    --access-token eyJraWQiOiJ... \
    --user-attribute-names phone_number custom:company
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove user attributes.

**When to Use:** Clearing optional attributes.

**Note:** Cannot delete required attributes.

---

## Groups & Permissions

### 1. Create Group

```bash
aws cognito-idp create-group \
    --group-name Admins \
    --user-pool-id us-east-1_ABC123 \
    --description "Administrator group" \
    --role-arn arn:aws:iam::123456789012:role/CognitoAdminRole \
    --precedence 1
```

**Expected Output:**
```json
{
    "Group": {
        "GroupName": "Admins",
        "UserPoolId": "us-east-1_ABC123",
        "Description": "Administrator group",
        "RoleArn": "arn:aws:iam::123456789012:role/CognitoAdminRole",
        "Precedence": 1,
        "CreationDate": "2025-11-17T11:00:00+00:00"
    }
}
```

**Why:** Create user groups for role-based access control.

**When to Use:** Organizing users, permission management.

**Precedence:** Lower number = higher priority (for IAM role assignment).

---

### 2. List Groups

```bash
aws cognito-idp list-groups --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```json
{
    "Groups": [
        {
            "GroupName": "Admins",
            "UserPoolId": "us-east-1_ABC123",
            "Description": "Administrator group",
            "Precedence": 1
        },
        {
            "GroupName": "Users",
            "UserPoolId": "us-east-1_ABC123",
            "Precedence": 10
        }
    ]
}
```

**Why:** View all groups in user pool.

**When to Use:** Group management, auditing.

---

### 3. Add User to Group

```bash
aws cognito-idp admin-add-user-to-group \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com \
    --group-name Admins
```

**Expected Output:**
```
(No output on success)
```

**Why:** Assign user to group.

**When to Use:** Role assignment, permission management.

**Result:** User inherits group's IAM role (for Identity Pool integration).

---

### 4. List Users in Group

```bash
aws cognito-idp list-users-in-group \
    --user-pool-id us-east-1_ABC123 \
    --group-name Admins
```

**Expected Output:**
```json
{
    "Users": [
        {
            "Username": "john@example.com",
            "Attributes": [
                {
                    "Name": "email",
                    "Value": "john@example.com"
                }
            ],
            "UserStatus": "CONFIRMED"
        }
    ]
}
```

**Why:** View group members.

**When to Use:** Group auditing, membership verification.

---

### 5. List Groups for User

```bash
aws cognito-idp admin-list-groups-for-user \
    --username john@example.com \
    --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```json
{
    "Groups": [
        {
            "GroupName": "Admins",
            "UserPoolId": "us-east-1_ABC123",
            "Precedence": 1
        }
    ]
}
```

**Why:** View user's group memberships.

**When to Use:** Permission auditing, troubleshooting access.

---

### 6. Remove User from Group

```bash
aws cognito-idp admin-remove-user-from-group \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com \
    --group-name Admins
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove user from group.

**When to Use:** Role changes, permission revocation.

---

### 7. Delete Group

```bash
aws cognito-idp delete-group \
    --group-name Admins \
    --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```
(No output on success)
```

**Why:** Remove group.

**When to Use:** Cleanup, restructuring permissions.

**Note:** Users in group are not deleted, just group membership removed.

---

## Multi-Factor Authentication (MFA)

### 1. Set MFA Configuration (User Pool)

```bash
aws cognito-idp set-user-pool-mfa-config \
    --user-pool-id us-east-1_ABC123 \
    --software-token-mfa-configuration Enabled=true \
    --sms-mfa-configuration Enabled=true,SmsConfiguration='{SnsCallerArn=arn:aws:iam::123456789012:role/CognitoSNSRole}' \
    --mfa-configuration OPTIONAL
```

**Expected Output:**
```json
{
    "SoftwareTokenMfaConfiguration": {
        "Enabled": true
    },
    "SmsMfaConfiguration": {
        "Enabled": true
    },
    "MfaConfiguration": "OPTIONAL"
}
```

**Why:** Configure MFA options for user pool.

**When to Use:** Enhanced security requirements.

**MFA Configuration:**
- **OFF**: MFA disabled
- **OPTIONAL**: User chooses
- **ON**: Required for all users

**MFA Methods:**
- **SMS**: Text message codes
- **TOTP**: Time-based OTP (Google Authenticator, Authy, etc.)

---

### 2. Get MFA Configuration

```bash
aws cognito-idp get-user-pool-mfa-config --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```json
{
    "SoftwareTokenMfaConfiguration": {
        "Enabled": true
    },
    "SmsMfaConfiguration": {
        "Enabled": true
    },
    "MfaConfiguration": "OPTIONAL"
}
```

**Why:** View current MFA settings.

**When to Use:** Auditing, troubleshooting.

---

### 3. Associate Software Token (TOTP)

```bash
aws cognito-idp associate-software-token --access-token eyJraWQiOiJ...
```

**Expected Output:**
```json
{
    "SecretCode": "JBSWY3DPEHPK3PXP",
    "Session": "session-token-abc123"
}
```

**Why:** Set up TOTP MFA (generate QR code data).

**When to Use:** User enabling TOTP MFA.

**Next Step:** User scans QR code with authenticator app, then verifies.

---

### 4. Verify Software Token

```bash
aws cognito-idp verify-software-token \
    --access-token eyJraWQiOiJ... \
    --user-code 123456
```

**Expected Output:**
```json
{
    "Status": "SUCCESS"
}
```

**Why:** Confirm TOTP setup with code from authenticator app.

**When to Use:** Completing TOTP setup.

---

### 5. Set User MFA Preference

```bash
aws cognito-idp set-user-mfa-preference \
    --access-token eyJraWQiOiJ... \
    --software-token-mfa-settings Enabled=true,PreferredMfa=true \
    --sms-mfa-settings Enabled=false
```

**Expected Output:**
```
(No output on success)
```

**Why:** User chooses preferred MFA method.

**When to Use:** User has multiple MFA options.

---

### 6. Admin Set User MFA Preference

```bash
aws cognito-idp admin-set-user-mfa-preference \
    --user-pool-id us-east-1_ABC123 \
    --username john@example.com \
    --software-token-mfa-settings Enabled=true,PreferredMfa=true
```

**Expected Output:**
```
(No output on success)
```

**Why:** Administrator sets user's MFA preference.

**When to Use:** Admin configuration, troubleshooting.

---

### 7. Respond to MFA Challenge

```bash
aws cognito-idp admin-respond-to-auth-challenge \
    --user-pool-id us-east-1_ABC123 \
    --client-id 1a2b3c4d5e6f7g8h9i0j \
    --challenge-name SMS_MFA \
    --session session-token-abc123 \
    --challenge-responses USERNAME=john@example.com,SMS_MFA_CODE=123456
```

**Expected Output:**
```json
{
    "AuthenticationResult": {
        "AccessToken": "eyJraWQiOiJ...",
        "IdToken": "eyJraWQiOiJ...",
        "RefreshToken": "eyJjdHkiOiJ..."
    }
}
```

**Why:** Complete MFA challenge during sign-in.

**When to Use:** MFA-enabled user login.

**Challenge Names:**
- **SMS_MFA**: SMS code
- **SOFTWARE_TOKEN_MFA**: TOTP code

---

## Password Policies

### 1. Update Password Policy

```bash
aws cognito-idp update-user-pool \
    --user-pool-id us-east-1_ABC123 \
    --policies '{
        "PasswordPolicy": {
            "MinimumLength": 12,
            "RequireUppercase": true,
            "RequireLowercase": true,
            "RequireNumbers": true,
            "RequireSymbols": true,
            "TemporaryPasswordValidityDays": 7
        }
    }'
```

**Expected Output:**
```
(No output on success)
```

**Why:** Enforce password complexity requirements.

**When to Use:** Security hardening, compliance requirements.

**Settings:**
- **MinimumLength**: 6-99 characters
- **RequireUppercase**: A-Z
- **RequireLowercase**: a-z
- **RequireNumbers**: 0-9
- **RequireSymbols**: Special characters
- **TemporaryPasswordValidityDays**: Temp password expiry

---

## Identity Pools (Federated Identities)

### 1. Create Identity Pool

```bash
aws cognito-identity create-identity-pool \
    --identity-pool-name my-identity-pool \
    --allow-unauthenticated-identities \
    --cognito-identity-providers ProviderName=cognito-idp.us-east-1.amazonaws.com/us-east-1_ABC123,ClientId=1a2b3c4d5e6f7g8h9i0j
```

**Expected Output:**
```json
{
    "IdentityPoolId": "us-east-1:12345678-1234-1234-1234-123456789012",
    "IdentityPoolName": "my-identity-pool",
    "AllowUnauthenticatedIdentities": true,
    "CognitoIdentityProviders": [
        {
            "ProviderName": "cognito-idp.us-east-1.amazonaws.com/us-east-1_ABC123",
            "ClientId": "1a2b3c4d5e6f7g8h9i0j"
        }
    ]
}
```

**Why:** Grant AWS credentials to users.

**When to Use:** Users need to access AWS services (S3, DynamoDB, etc.).

**Unauthenticated Identities:** Allow guest access with limited permissions.

---

### 2. Set Identity Pool Roles

```bash
aws cognito-identity set-identity-pool-roles \
    --identity-pool-id us-east-1:12345678-1234-1234-1234-123456789012 \
    --roles authenticated=arn:aws:iam::123456789012:role/Cognito_AuthRole,unauthenticated=arn:aws:iam::123456789012:role/Cognito_UnauthRole
```

**Expected Output:**
```
(No output on success)
```

**Why:** Assign IAM roles for authenticated/unauthenticated users.

**When to Use:** Controlling AWS resource access permissions.

**Roles:**
- **Authenticated**: Logged-in users
- **Unauthenticated**: Guest users

---

### 3. Get Identity Pool Roles

```bash
aws cognito-identity get-identity-pool-roles --identity-pool-id us-east-1:12345678-1234-1234-1234-123456789012
```

**Expected Output:**
```json
{
    "IdentityPoolId": "us-east-1:12345678-1234-1234-1234-123456789012",
    "Roles": {
        "authenticated": "arn:aws:iam::123456789012:role/Cognito_AuthRole",
        "unauthenticated": "arn:aws:iam::123456789012:role/Cognito_UnauthRole"
    }
}
```

**Why:** View IAM role assignments.

**When to Use:** Auditing, troubleshooting permissions.

---

### 4. Get Credentials for Identity

**Get Identity ID:**
```bash
aws cognito-identity get-id \
    --identity-pool-id us-east-1:12345678-1234-1234-1234-123456789012 \
    --logins cognito-idp.us-east-1.amazonaws.com/us-east-1_ABC123=eyJraWQiOiJ...
```

**Expected Output:**
```json
{
    "IdentityId": "us-east-1:87654321-4321-4321-4321-210987654321"
}
```

**Get Credentials:**
```bash
aws cognito-identity get-credentials-for-identity \
    --identity-id us-east-1:87654321-4321-4321-4321-210987654321 \
    --logins cognito-idp.us-east-1.amazonaws.com/us-east-1_ABC123=eyJraWQiOiJ...
```

**Expected Output:**
```json
{
    "IdentityId": "us-east-1:87654321-4321-4321-4321-210987654321",
    "Credentials": {
        "AccessKeyId": "ASIAXXX",
        "SecretKey": "xxx",
        "SessionToken": "xxx",
        "Expiration": "2025-11-17T12:00:00+00:00"
    }
}
```

**Why:** Get temporary AWS credentials for user.

**When to Use:** Client accessing AWS services directly.

---

## Social & Enterprise Identity Providers

### 1. Create Identity Provider (SAML)

```bash
aws cognito-idp create-identity-provider \
    --user-pool-id us-east-1_ABC123 \
    --provider-name MyEnterpriseIdP \
    --provider-type SAML \
    --provider-details file://saml-metadata.json \
    --attribute-mapping email=http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress
```

**saml-metadata.json:**
```json
{
    "MetadataURL": "https://idp.example.com/metadata.xml"
}
```

**Expected Output:**
```json
{
    "IdentityProvider": {
        "UserPoolId": "us-east-1_ABC123",
        "ProviderName": "MyEnterpriseIdP",
        "ProviderType": "SAML",
        "ProviderDetails": {
            "MetadataURL": "https://idp.example.com/metadata.xml"
        }
    }
}
```

**Why:** Enable enterprise SSO with SAML.

**When to Use:** Corporate identity integration (Okta, Azure AD, etc.).

---

### 2. Create Identity Provider (Social - Google)

```bash
aws cognito-idp create-identity-provider \
    --user-pool-id us-east-1_ABC123 \
    --provider-name Google \
    --provider-type Google \
    --provider-details client_id=YOUR_GOOGLE_CLIENT_ID,client_secret=YOUR_GOOGLE_CLIENT_SECRET,authorize_scopes=email,profile \
    --attribute-mapping email=email,name=name
```

**Expected Output:**
```json
{
    "IdentityProvider": {
        "UserPoolId": "us-east-1_ABC123",
        "ProviderName": "Google",
        "ProviderType": "Google"
    }
}
```

**Why:** Enable Google sign-in.

**When to Use:** Social login for convenience.

**Supported Social Providers:**
- Google
- Facebook
- LoginWithAmazon
- SignInWithApple

---

### 3. List Identity Providers

```bash
aws cognito-idp list-identity-providers --user-pool-id us-east-1_ABC123
```

**Expected Output:**
```json
{
    "Providers": [
        {
            "ProviderName": "Google",
            "ProviderType": "Google",
            "UserPoolId": "us-east-1_ABC123"
        }
    ]
}
```

**Why:** View configured identity providers.

**When to Use:** Auditing, configuration verification.

---

## Lambda Triggers

### 1. Update User Pool with Lambda Triggers

```bash
aws cognito-idp update-user-pool \
    --user-pool-id us-east-1_ABC123 \
    --lambda-config '{
        "PreSignUp": "arn:aws:lambda:us-east-1:123456789012:function:PreSignUp",
        "PostConfirmation": "arn:aws:lambda:us-east-1:123456789012:function:PostConfirmation",
        "PreAuthentication": "arn:aws:lambda:us-east-1:123456789012:function:PreAuth",
        "PostAuthentication": "arn:aws:lambda:us-east-1:123456789012:function:PostAuth",
        "CustomMessage": "arn:aws:lambda:us-east-1:123456789012:function:CustomMessage"
    }'
```

**Expected Output:**
```
(No output on success)
```

**Why:** Customize Cognito workflows with Lambda.

**When to Use:** Custom validation, notifications, integration with other systems.

**Available Triggers:**
- **PreSignUp**: Before user registration
- **PostConfirmation**: After email/phone verification
- **PreAuthentication**: Before sign-in
- **PostAuthentication**: After sign-in
- **CustomMessage**: Customize verification messages
- **DefineAuthChallenge**: Custom auth flow
- **CreateAuthChallenge**: Generate auth challenge
- **VerifyAuthChallengeResponse**: Validate auth challenge
- **PreTokenGeneration**: Modify token claims
- **UserMigration**: Migrate users from legacy system

---

### 2. PreSignUp Trigger Example

```python
def lambda_handler(event, context):
    # Auto-confirm users from specific domain
    if event['request']['userAttributes']['email'].endswith('@example.com'):
        event['response']['autoConfirmUser'] = True
        event['response']['autoVerifyEmail'] = True

    # Block sign-up from specific domains
    if event['request']['userAttributes']['email'].endswith('@blocked.com'):
        raise Exception('Sign-ups from this domain are not allowed')

    return event
```

**Why:** Customize registration logic.

**When to Use:** Auto-verification, domain restrictions, custom validation.

---

### 3. PostConfirmation Trigger Example

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

def lambda_handler(event, context):
    # Create user profile in DynamoDB after confirmation
    table.put_item(Item={
        'userId': event['request']['userAttributes']['sub'],
        'email': event['request']['userAttributes']['email'],
        'createdAt': int(time.time())
    })

    return event
```

**Why:** Perform actions after user confirms account.

**When to Use:** Creating user profiles, sending welcome emails, analytics.

---

## Token Management

### 1. Decode JWT Token (Using Python)

```python
import jwt
import requests

# Get JWKS (JSON Web Key Set)
jwks_url = f'https://cognito-idp.us-east-1.amazonaws.com/us-east-1_ABC123/.well-known/jwks.json'
jwks = requests.get(jwks_url).json()

# Decode token (no verification - for inspection only)
token = "eyJraWQiOiJ..."
decoded = jwt.decode(token, options={"verify_signature": False})
print(decoded)

# Verify and decode
# (requires additional setup with public keys)
```

**Why:** Inspect token contents, verify claims.

**When to Use:** Debugging, custom authorization logic.

---

### 2. Verify Token Signature (Python with python-jose)

```python
from jose import jwt, jwk
from jose.utils import base64url_decode
import requests

def verify_token(token, region, user_pool_id):
    # Get JWKS
    jwks_url = f'https://cognito-idp.{region}.amazonaws.com/{user_pool_id}/.well-known/jwks.json'
    jwks = requests.get(jwks_url).json()

    # Get token header
    headers = jwt.get_unverified_headers(token)
    kid = headers['kid']

    # Find matching key
    key = next(k for k in jwks['keys'] if k['kid'] == kid)

    # Verify
    public_key = jwk.construct(key)
    message, encoded_signature = str(token).rsplit('.', 1)
    decoded_signature = base64url_decode(encoded_signature.encode())

    # Verify signature
    if public_key.verify(message.encode(), decoded_signature):
        claims = jwt.get_unverified_claims(token)
        return claims
    else:
        raise Exception('Invalid signature')

# Usage
claims = verify_token(token, 'us-east-1', 'us-east-1_ABC123')
print(claims['email'])
```

**Why:** Verify token authenticity.

**When to Use:** Custom API authorization, backend validation.

---

## Best Practices

### Security
1. **Enable MFA** - For sensitive applications
2. **Use strong password policies** - Min 12 chars, complexity requirements
3. **Enable advanced security features** - Compromised credentials check
4. **Rotate app client secrets** - Regular rotation
5. **Use HTTPS only** - Redirect URLs must be HTTPS
6. **Limit token lifetime** - Shorter for sensitive apps
7. **Implement rate limiting** - Prevent brute force
8. **Monitor suspicious activity** - CloudWatch alarms

### User Experience
1. **Use hosted UI** - Faster implementation, maintained by AWS
2. **Social sign-in** - Reduce friction
3. **Custom email templates** - Brand consistency
4. **Password-less auth** - Consider TOTP, magic links
5. **Remember device** - Reduce MFA prompts
6. **Clear error messages** - But don't leak information

### Development
1. **Use User Pool Client per app** - Separate web, mobile, API
2. **Test with LocalStack** - Local development
3. **Use Lambda triggers** - Customize workflows
4. **Implement proper token refresh** - Handle expiration gracefully
5. **Store tokens securely** - HttpOnly cookies, secure storage
6. **Use Identity Pools for AWS access** - Not User Pool tokens

### Cost Optimization
1. **Monitor MAUs** - Monthly Active Users pricing tier
2. **Use hosted UI** - Avoid custom implementation costs
3. **Optimize Lambda triggers** - Efficient code, avoid unnecessary invocations
4. **Set appropriate token lifetimes** - Balance UX and token refresh frequency

### Compliance
1. **Enable CloudTrail** - Audit all API calls
2. **Use encryption** - KMS for sensitive attributes
3. **Implement GDPR compliance** - User data deletion, export
4. **Document data retention** - Policy and implementation
5. **Regular security audits** - Review configurations

---

## Troubleshooting

### User Cannot Sign In
- Check user status (CONFIRMED vs FORCE_CHANGE_PASSWORD)
- Verify password meets policy requirements
- Check if account is disabled
- Verify MFA configuration if enabled

### Token Validation Fails
- Check token expiration
- Verify token signature with JWKS
- Check audience (aud) and issuer (iss) claims
- Ensure using correct User Pool ID

### SMS Not Received
- Check phone number format (+country code)
- Verify SNS IAM role permissions
- Check AWS account SMS spending limit
- Verify region supports SMS

### Social Sign-In Fails
- Verify IdP credentials (client ID, secret)
- Check callback URLs match exactly
- Verify IdP is enabled in User Pool Client
- Check attribute mappings

---

## Quick Reference

```bash
# User Pool
aws cognito-idp create-user-pool --pool-name NAME
aws cognito-idp list-user-pools --max-results 10

# User Pool Client
aws cognito-idp create-user-pool-client --user-pool-id POOL_ID --client-name NAME

# Users
aws cognito-idp admin-create-user --user-pool-id POOL_ID --username USER
aws cognito-idp list-users --user-pool-id POOL_ID
aws cognito-idp admin-delete-user --user-pool-id POOL_ID --username USER

# Authentication
aws cognito-idp admin-initiate-auth --user-pool-id POOL_ID --client-id CLIENT_ID --auth-flow ADMIN_NO_SRP_AUTH --auth-parameters USERNAME=USER,PASSWORD=PASS

# Groups
aws cognito-idp create-group --user-pool-id POOL_ID --group-name GROUP
aws cognito-idp admin-add-user-to-group --user-pool-id POOL_ID --username USER --group-name GROUP

# Identity Pool
aws cognito-identity create-identity-pool --identity-pool-name NAME
```

---

## Summary

Cognito provides complete authentication and authorization for applications. Master these commands to:
- Manage user registration and authentication
- Implement MFA and social sign-in
- Control access with groups and IAM roles
- Customize workflows with Lambda triggers
- Integrate with AWS services via Identity Pools

Practice creating user pools, managing users, and implementing authentication flows. Cognito's flexibility and security features make it ideal for modern applications.

For LocalStack: add `--endpoint-url=http://localhost:4566` to commands.

Happy authenticating!
