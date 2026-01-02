# AWS IAM (Identity and Access Management) - Study Notes

## Table of Contents
1. [Overview](#overview)
2. [Core Components](#core-components)
3. [IAM Policies](#iam-policies)
4. [IAM Roles](#iam-roles)
5. [Security Best Practices](#security-best-practices)
6. [Advanced Concepts](#advanced-concepts)
7. [Common Use Cases](#common-use-cases)
8. [Troubleshooting](#troubleshooting)

---

## Overview

### What is IAM?
- Global service that manages access to AWS resources
- No additional charge for IAM
- Enables secure control over authentication and authorization
- Foundation of AWS security

### Key Principles
- **Least Privilege**: Grant only necessary permissions
- **Defense in Depth**: Multiple layers of security
- **Separation of Duties**: No single user has excessive permissions
- **Audit and Monitor**: Track all IAM activities

---

## Core Components

### 1. IAM Users
- Represents a person or application
- Has permanent long-term credentials
- **Types of access:**
  - Programmatic access (Access Key ID + Secret Access Key)
  - AWS Management Console access (username + password)
- Maximum 5,000 users per AWS account

**Best Practices:**
- Create individual users (never share credentials)
- Enable MFA for all users
- Rotate credentials regularly
- Use roles for applications instead of embedding access keys

### 2. IAM Groups
- Collection of IAM users
- Makes permission management easier
- Users inherit permissions from groups
- A user can belong to multiple groups (max 10)
- Groups cannot be nested

**Example Structure:**
```
Developers Group → ReadOnly to dev resources
Admins Group → Full access to all resources
DevOps Group → Deployment and infrastructure management
```

### 3. IAM Policies
- JSON documents defining permissions
- Attached to users, groups, or roles
- Define what actions are allowed or denied

**Types:**
- **AWS Managed Policies**: Created and maintained by AWS
- **Customer Managed Policies**: Created by you, reusable
- **Inline Policies**: Embedded directly, 1:1 relationship

### 4. IAM Roles
- Temporary security credentials
- Assumed by users, applications, or services
- No permanent credentials
- More secure than embedding access keys

**Common Role Types:**
- Service roles (for EC2, Lambda, etc.)
- Cross-account roles
- Identity provider roles (SAML, OIDC)

---

## IAM Policies

### Policy Structure
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

### Policy Elements
- **Version**: Policy language version (always use "2012-10-17")
- **Statement**: Array of individual statements
- **Effect**: "Allow" or "Deny"
- **Action**: API operations (e.g., s3:PutObject, ec2:StartInstances)
- **Resource**: ARN of resources the action applies to
- **Condition**: (Optional) Circumstances under which the policy grants permission

### Policy Evaluation Logic
1. By default, all requests are **implicitly denied**
2. **Explicit allow** in policy overrides implicit deny
3. **Explicit deny** overrides any allows
4. Only if there's an explicit allow and no explicit deny, the request is allowed

### Wildcards and Variables
```json
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::${aws:username}/*"
}
```
- `*` matches any characters
- `?` matches a single character
- Variables like `${aws:username}` for dynamic policies

---

## IAM Roles

### Why Use Roles?
- No long-term credentials to manage or rotate
- Temporary security credentials
- Easy to grant and revoke access
- Better for EC2 instances, Lambda functions, etc.

### Role Components
1. **Trust Policy**: Who can assume the role
2. **Permission Policy**: What the role can do

### Example: EC2 Instance Role
```json
// Trust Policy
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "ec2.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}
```

### Cross-Account Access
- Role in Account B with trust policy allowing Account A
- User in Account A assumes role in Account B
- External ID for additional security (prevents confused deputy problem)

### Session Duration
- Default: 1 hour
- Maximum: 12 hours (for roles)
- Configure based on security requirements

---

## Security Best Practices

### 1. Root Account Security
- Enable MFA on root account
- Don't use root account for daily tasks
- Create IAM users with admin permissions instead
- Lock away root credentials securely

### 2. Authentication
- Enforce strong password policy
- Enable MFA for all human users
- Rotate access keys regularly (90 days)
- Remove unnecessary credentials

### 3. Authorization
- Apply least privilege principle
- Use groups to assign permissions
- Review permissions regularly
- Use IAM Access Analyzer

### 4. Monitoring and Auditing
- Enable CloudTrail in all regions
- Review IAM credential reports
- Set up CloudWatch alarms for suspicious activities
- Use AWS Config for compliance

### 5. Key Management
- Never embed access keys in code
- Use roles for applications
- Store secrets in AWS Secrets Manager or Parameter Store
- Use temporary credentials when possible

---

## Advanced Concepts

### 1. Permission Boundaries
- Maximum permissions an IAM entity can have
- Used to delegate permission management safely
- Does NOT grant permissions (only limits them)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:*",
      "ec2:*"
    ],
    "Resource": "*"
  }]
}
```

### 2. Service Control Policies (SCPs)
- Part of AWS Organizations
- Set maximum permissions for accounts
- Applied at organization, OU, or account level
- Affect all users and roles in the account (except root)

### 3. IAM Policy Conditions
Fine-grained access control:

```json
"Condition": {
  "DateGreaterThan": {"aws:CurrentTime": "2024-01-01T00:00:00Z"},
  "DateLessThan": {"aws:CurrentTime": "2024-12-31T23:59:59Z"},
  "IpAddress": {"aws:SourceIp": "10.0.0.0/8"},
  "StringEquals": {"aws:RequestedRegion": "us-east-1"}
}
```

### 4. IAM Identity Center (AWS SSO)
- Centralized access management
- Single sign-on across AWS accounts
- Integration with external identity providers
- Better than creating individual IAM users

### 5. IAM Access Analyzer
- Identifies resources shared with external entities
- Validates policies against best practices
- Generates policies based on CloudTrail logs
- Helps achieve least privilege

### 6. Resource-Based Policies
- Attached to resources (S3 buckets, KMS keys, etc.)
- Define who can access the resource
- Evaluated alongside identity-based policies

### 7. Session Policies
- Passed when assuming a role
- Further restrict permissions for that session
- Useful for temporary additional restrictions

---

## Common Use Cases

### 1. Developer Access to Dev Environment
```
- Create "Developers" group
- Attach policies for EC2, S3, RDS read/write in dev account
- Add condition to restrict to specific regions
- Enable MFA requirement
```

### 2. Cross-Account CI/CD Pipeline
```
- Create role in production account
- Trust policy allows dev account
- CI/CD tool in dev account assumes production role
- Deploys applications with temporary credentials
```

### 3. Third-Party Access
```
- Create role with external ID
- Trust policy allows third-party AWS account
- Provide role ARN and external ID to third party
- Third party assumes role for limited access
```

### 4. Temporary Elevated Permissions
```
- User normally has read-only access
- Create role with elevated permissions
- User assumes role when needed (break-glass scenario)
- All actions logged in CloudTrail
```

---

## Troubleshooting

### Common Issues

**Issue: Access Denied**
- Check policy syntax
- Verify policy is attached correctly
- Look for explicit deny
- Check permission boundaries and SCPs
- Ensure service role has trust relationship

**Issue: Cannot Assume Role**
- Verify trust policy allows the principal
- Check if role requires MFA or external ID
- Ensure user has sts:AssumeRole permission
- Verify session duration hasn't expired

**Issue: Policy Not Working**
- Check policy version is "2012-10-17"
- Verify JSON syntax
- Ensure actions and resources match correctly
- Check conditions are met
- Review policy evaluation order

### Useful AWS CLI Commands
```bash
# List users
aws iam list-users

# Get user details
aws iam get-user --user-name username

# List policies attached to user
aws iam list-attached-user-policies --user-name username

# Simulate policy
aws iam simulate-principal-policy --policy-source-arn arn:aws:iam::123456789012:user/username --action-names s3:GetObject

# Generate credential report
aws iam generate-credential-report
aws iam get-credential-report

# Get role
aws iam get-role --role-name rolename

# Assume role
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/rolename --role-session-name session1
```

---

## Quick Reference

### IAM Limits
- Users per account: 5,000
- Groups per account: 300
- Roles per account: 5,000
- Policies per user/group/role: 10 managed policies
- Policy size: 6,144 characters (managed), 10,240 (inline)
- Groups per user: 10

### Important ARN Formats
```
arn:aws:iam::123456789012:user/username
arn:aws:iam::123456789012:group/groupname
arn:aws:iam::123456789012:role/rolename
arn:aws:iam::123456789012:policy/policyname
arn:aws:iam::aws:policy/policyname  # AWS managed
```

### Policy Variables
- `${aws:username}` - IAM user name
- `${aws:userid}` - Unique ID of the user
- `${aws:PrincipalTag/tag-key}` - Tag attached to principal
- `${aws:SourceIp}` - Source IP address
- `${aws:CurrentTime}` - Current date and time

---

## Study Resources

### Official Documentation
- AWS IAM User Guide
- AWS IAM Policy Reference
- IAM Best Practices
- AWS Security Blog
