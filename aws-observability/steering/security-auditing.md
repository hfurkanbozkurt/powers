# CloudTrail Security Auditing Steering

## Purpose
This steering file provides guidance for using AWS CloudTrail to perform security auditing, compliance monitoring, and governance analysis.

## When to Load This Steering
Load this when the user needs to:
- Investigate security incidents
- Track API activity and resource changes
- Perform compliance audits
- Monitor IAM changes
- Detect unauthorized access attempts
- Generate audit reports

## CloudTrail Overview

CloudTrail provides governance, compliance, and audit capabilities by logging all API calls in your AWS account:

- **Event History**: 90 days of management events in the console
- **Trails**: Long-term storage of events in S3
- **CloudWatch Integration**: Real-time event analysis
- **Multi-Region Support**: Capture events across all regions
- **Multi-Account Support**: Aggregate logs from organization

## Core Concepts

### Event Types

**1. Management Events**
- Control plane operations
- Resource creation, modification, deletion
- IAM changes
- Security group modifications
- Examples: CreateBucket, RunInstances, CreateUser

**2. Data Events**
- Data plane operations
- S3 object-level operations
- Lambda function invocations
- DynamoDB table operations
- Examples: GetObject, PutObject, Invoke

**3. Insights Events**
- Unusual API activity detected by ML
- Rate-based anomalies
- Error rate anomalies
- Requires CloudTrail Insights enabled

### Event Structure

```json
{
  "eventTime": "2024-12-08T10:30:00Z",
  "eventName": "CreateBucket",
  "eventSource": "s3.amazonaws.com",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAI...",
    "arn": "arn:aws:iam::123456789012:user/alice",
    "userName": "alice"
  },
  "sourceIPAddress": "203.0.113.0",
  "userAgent": "aws-cli/2.0.0",
  "requestParameters": {
    "bucketName": "my-new-bucket"
  },
  "responseElements": {
    "location": "https://my-new-bucket.s3.amazonaws.com/"
  },
  "errorCode": null,
  "errorMessage": null
}
```

## CloudTrail MCP Server Tools

### Available Operations

1. **lookup_events**: Search CloudTrail events
2. **get_event_selectors**: View trail configuration
3. **get_trail_status**: Check trail status
4. **list_trails**: List all trails in account

### Tool Usage Patterns

#### Pattern 1: Basic Event Lookup
```
lookup_events:
  - Time range: Last 24 hours
  - Filters:
    - Event name: [specific API call]
    - Username: [IAM user]
    - Resource type: [AWS service]
  - Max results: 50
```

#### Pattern 2: Security Incident Investigation
```
1. Identify suspicious activity:
   lookup_events
   - Event name: [DeleteBucket, TerminateInstances, etc.]
   - Time range: Incident window
   - Username: [if known]

2. Trace user activity:
   lookup_events
   - Username: [suspect user]
   - Time range: Expanded window
   - All event names

3. Check IAM changes:
   lookup_events
   - Event source: iam.amazonaws.com
   - Time range: Before incident
```

#### Pattern 3: Compliance Audit
```
1. List all IAM changes:
   lookup_events
   - Event source: iam.amazonaws.com
   - Time range: Audit period (30 days)
   - Event names: Create*, Delete*, Update*, Attach*, Detach*

2. Track privileged actions:
   lookup_events
   - Event names: AssumeRole, GetFederationToken
   - Time range: Audit period

3. Review resource deletions:
   lookup_events
   - Event name pattern: Delete*
   - Time range: Audit period
```

#### Pattern 4: Resource Change Tracking
```
1. Find specific resource changes:
   lookup_events
   - Resource name: [resource ARN or ID]
   - Time range: Last 90 days

2. Track configuration changes:
   lookup_events
   - Event source: [service].amazonaws.com
   - Event names: Update*, Modify*, Put*
```

## Security Monitoring Use Cases

### Use Case 1: Unauthorized Access Detection

**Indicators**:
- Failed authentication attempts
- Access from unusual IP addresses
- Access outside business hours
- Privilege escalation attempts

**CloudTrail Query**:
```
lookup_events:
  - Event names: ConsoleLogin, AssumeRole
  - Time range: Last 24 hours
  - Filter by: Error codes (AccessDenied, AuthFailure)

Follow-up Logs Insights Query (CloudWatch Logs):
fields eventTime, userIdentity.userName as user, sourceIPAddress, eventName, errorCode, errorMessage
| filter eventName in ["ConsoleLogin", "AssumeRole"]
| filter errorCode in ["AccessDenied", "AuthFailure"]
| sort eventTime desc
| limit 100
```

### Use Case 2: IAM Changes Audit

**Monitor for**:
- User/role creation and deletion
- Policy modifications
- Permission boundary changes
- Access key creation
- MFA changes

**CloudTrail Query**:
```
lookup_events:
  - Event source: iam.amazonaws.com
  - Event names: 
    - CreateUser, DeleteUser
    - CreateRole, DeleteRole
    - PutUserPolicy, PutRolePolicy
    - AttachUserPolicy, AttachRolePolicy
    - CreateAccessKey
    - DeactivateMFADevice
```

**Logs Insights Analysis**:
```
# IAM changes audit
fields eventTime, eventName, userIdentity.userName as actor, requestParameters.userName as targetUser, requestParameters.policyName as policyName, sourceIPAddress
| filter eventSource = "iam.amazonaws.com"
| filter eventName in ["CreateUser", "DeleteUser", "AttachUserPolicy", "PutUserPolicy", "CreateAccessKey"]
| sort eventTime desc
| limit 100
```

### Use Case 3: Resource Deletion Tracking

**Monitor deletions of**:
- S3 buckets
- EC2 instances
- RDS databases
- Lambda functions
- DynamoDB tables

**CloudTrail Query**:
```
lookup_events:
  - Event names: Delete*, Terminate*
  - Time range: Last 30 days
```

**Logs Insights Analysis**:
```
# Resource deletion tracking
fields eventTime, eventName, eventSource, userIdentity.userName as user, requestParameters.bucketName, requestParameters.instanceId, requestParameters.dBInstanceIdentifier as dbInstance, sourceIPAddress
| filter eventName like /Delete/ or eventName like /Terminate/
| sort eventTime desc
| limit 100
```

### Use Case 4: Security Group Changes

**Monitor**:
- New security group rules
- Overly permissive rules (0.0.0.0/0)
- Rule deletions

**CloudTrail Query**:
```
lookup_events:
  - Event source: ec2.amazonaws.com
  - Event names: AuthorizeSecurityGroupIngress, 
                 AuthorizeSecurityGroupEgress,
                 RevokeSecurityGroupIngress,
                 RevokeSecurityGroupEgress
  - Time range: Last 7 days
```

**Logs Insights Analysis**:
```
# Security group changes - check for overly permissive rules
fields eventTime, eventName, userIdentity.userName as user, requestParameters.groupId as securityGroupId, requestParameters
| filter eventName in ["AuthorizeSecurityGroupIngress", "AuthorizeSecurityGroupEgress"]
| sort eventTime desc
| limit 100
```

### Use Case 5: Root Account Activity

**Critical to Monitor**:
- Root account usage should be rare
- MFA should always be used
- All root actions should be justified

**CloudTrail Query**:
```
lookup_events:
  - User type: Root
  - Time range: Last 90 days
```

**Logs Insights Analysis**:
```
# Root account activity monitoring
fields eventTime, eventName, eventSource, userIdentity.type as userType, sourceIPAddress, additionalEventData.MFAUsed as mfaUsed
| filter userIdentity.type = "Root"
| sort eventTime desc
| limit 100
```

### Use Case 6: Cross-Account Access

**Monitor**:
- AssumeRole calls from external accounts
- Cross-account resource access
- Federated user activity

**CloudTrail Query**:
```
lookup_events:
  - Event name: AssumeRole
  - Time range: Last 30 days
```

**Logs Insights Analysis**:
```
# Cross-account access monitoring
fields eventTime, userIdentity.principalId as assumingPrincipal, userIdentity.accountId as sourceAccount, requestParameters.roleArn as assumedRole, requestParameters.roleSessionName as sessionName, sourceIPAddress
| filter eventName = "AssumeRole"
| filter userIdentity.accountId != "123456789012"
| sort eventTime desc
| limit 100
```

## Incident Investigation Workflows

### Workflow 1: Suspicious IAM Activity

**Scenario**: Alert for unusual IAM activity

**Investigation Steps**:

1. **Identify the event**:
   ```
   lookup_events:
     - Event source: iam.amazonaws.com
     - Time range: Alert time ± 1 hour
   ```

2. **Trace user activity**:
   ```
   # Trace all activity for suspect user
   fields eventTime, eventName, eventSource, sourceIPAddress, userAgent
   | filter userIdentity.userName = "[suspect-user]"
   | sort eventTime asc
   | limit 500
   ```

3. **Check for privilege escalation**:
   ```
   # Check for privilege escalation attempts
   fields eventTime, eventName, requestParameters.policyDocument as policy
   | filter userIdentity.userName = "[suspect-user]"
   | filter eventName in ["CreatePolicy", "AttachUserPolicy", "PutUserPolicy", "AttachRolePolicy"]
   | sort eventTime asc
   | limit 100
   ```

4. **Identify affected resources**:
   ```
   # Find resources accessed by suspect user
   fields eventName, eventSource, requestParameters.resourceArn as resource
   | filter userIdentity.userName = "[suspect-user]"
   | dedup eventName, eventSource, resource
   | limit 100
   ```

5. **Check access key usage**:
   ```
   lookup_events:
     - Event name: CreateAccessKey
     - Username: [suspect-user]
   ```

### Workflow 2: Resource Deletion Investigation

**Scenario**: Critical resource was deleted

**Investigation Steps**:

1. **Find deletion event**:
   ```
   # Find the deletion event for a specific resource
   fields eventTime, eventName, userIdentity.userName as user, userIdentity.type as userType, sourceIPAddress, requestParameters
   | filter eventName like /Delete/
   | filter requestParameters like /[resource-identifier]/
   | sort eventTime desc
   | limit 10
   ```

2. **Check user's recent activity**:
   ```
   # Review user's recent activity before deletion
   fields eventTime, eventName, eventSource, errorCode
   | filter userIdentity.userName = "[user-from-step-1]"
   | sort eventTime desc
   | limit 100
   ```

3. **Verify authentication**:
   ```
   # Check authentication events for the user
   fields eventTime, eventName, additionalEventData.MFAUsed as mfaUsed, sourceIPAddress, userAgent
   | filter userIdentity.userName = "[user-from-step-1]"
   | filter eventName in ["ConsoleLogin", "GetSessionToken"]
   | sort eventTime desc
   | limit 10
   ```

4. **Check for automation**:
   ```
   Look at userAgent field:
     - aws-cli/* indicates CLI usage
     - Boto3/* indicates SDK usage
     - aws-sdk-* indicates SDK usage
     - Check if expected for this user
   ```

### Workflow 3: Compliance Audit

**Scenario**: Quarterly compliance audit

**Audit Tasks**:

1. **IAM User Audit**:
   ```
   # List all users created in the audit period
   fields eventTime, eventName, userIdentity.userName as creator, requestParameters.userName as newUser
   | filter eventName = "CreateUser"
   | sort eventTime asc
   | limit 500
   ```

2. **Access Key Rotation**:
   ```
   # Find access keys created in the audit period
   fields eventTime, requestParameters.userName as user, responseElements.accessKey.accessKeyId as keyId
   | filter eventName = "CreateAccessKey"
   | sort eventTime asc
   | limit 500
   ```

3. **Policy Changes**:
   ```
   # Track policy modifications
   fields eventTime, eventName, userIdentity.userName as actor, requestParameters.policyName as policy
   | filter eventName in ["CreatePolicy", "CreatePolicyVersion", "DeletePolicy", "DeletePolicyVersion"]
   | sort eventTime asc
   | limit 500
   ```

4. **Encryption Changes**:
   ```
   # Track KMS key operations
   fields eventTime, eventName, userIdentity.userName as user, requestParameters.keyId as keyId
   | filter eventSource = "kms.amazonaws.com"
   | filter eventName in ["CreateKey", "ScheduleKeyDeletion", "DisableKey", "EnableKeyRotation"]
   | sort eventTime asc
   | limit 500
   ```

## Security Alerting Patterns

### Critical Alerts

**1. Root Account Usage**
```
# Alert: Root account activity
fields eventTime, eventName, sourceIPAddress
| filter userIdentity.type = "Root"
| sort eventTime desc
| limit 50
```
**Action**: Page security team immediately

**2. IAM Policy Changes**
```
# Alert: IAM policy modifications
fields eventTime, eventName, userIdentity.userName as user, requestParameters.policyDocument as policy
| filter eventName in ["PutUserPolicy", "PutRolePolicy", "AttachUserPolicy", "AttachRolePolicy"]
| sort eventTime desc
| limit 50
```
**Action**: Review policy changes, alert if suspicious

**3. Security Group Opening to Internet**
```
# Alert: Security group opened to 0.0.0.0/0
fields eventTime, userIdentity.userName as user, requestParameters
| filter eventName = "AuthorizeSecurityGroupIngress"
| filter requestParameters like /0.0.0.0\/0/
| sort eventTime desc
| limit 50
```
**Action**: Verify business justification

**4. KMS Key Deletion Scheduled**
```
# Alert: KMS key scheduled for deletion
fields eventTime, userIdentity.userName as user, requestParameters.keyId as keyId
| filter eventName = "ScheduleKeyDeletion"
| sort eventTime desc
| limit 50
```
**Action**: Confirm key is no longer needed

## Best Practices

### 1. Trail Configuration
- Enable in all regions
- Enable for all accounts (AWS Organizations)
- Enable log file validation
- Integrate with CloudWatch Logs
- Set up S3 lifecycle policies for cost optimization

### 2. Monitoring
- Set up CloudWatch Alarms for critical events
- Create metric filters for suspicious patterns
- Regular review of CloudTrail Insights
- Automate response to known threats

### 3. Analysis
- Use Logs Insights for complex queries across time
- Correlate CloudTrail with application logs
- Build dashboards for security metrics
- Regular audit log reviews

### 4. Retention
- Keep CloudTrail logs for required compliance period
- Consider separate retention for security events
- Archive to Glacier for cost optimization
- Ensure immutability (S3 Object Lock)

### 5. Access Control
- Restrict access to CloudTrail logs
- Use separate S3 buckets for CloudTrail
- Enable S3 access logging on CloudTrail bucket
- MFA Delete on CloudTrail bucket

## Integration with Other Tools

### CloudWatch Logs Insights
```
Send CloudTrail logs to CloudWatch Logs for:
- Real-time alerting
- Complex Logs Insights queries
- Cross-log-group correlation
- Dashboard visualization
```

### Security Hub
```
Integrate CloudTrail with Security Hub for:
- Centralized security findings
- Compliance standard mapping
- Multi-account aggregation
- Automated remediation
```

### GuardDuty
```
GuardDuty uses CloudTrail for:
- Threat detection
- Unusual API activity
- Compromised credentials
- Cryptocurrency mining
```

## Quick Reference

### Common Event Names
- **IAM**: CreateUser, DeleteUser, AttachUserPolicy, CreateAccessKey
- **EC2**: RunInstances, TerminateInstances, AuthorizeSecurityGroupIngress
- **S3**: CreateBucket, DeleteBucket, PutBucketPolicy
- **RDS**: CreateDBInstance, DeleteDBInstance, ModifyDBInstance
- **Lambda**: CreateFunction, DeleteFunction, UpdateFunctionCode

### User Identity Types
- **IAMUser**: Standard IAM user
- **AssumedRole**: Assumed role via STS
- **Root**: Root account
- **FederatedUser**: Federated identity
- **AWSService**: AWS service acting on your behalf

### Error Codes
- **AccessDenied**: Permission denied
- **UnauthorizedOperation**: Not authorized for operation
- **InvalidParameter**: Invalid request parameter
- **ResourceNotFound**: Resource doesn't exist

---

**Remember**: CloudTrail is your audit trail. Preserve log integrity, monitor continuously, and respond quickly to suspicious activity. Always correlate CloudTrail events with application logs and metrics for complete visibility.
