# AWS Security Monitoring System

A cloud-native security monitoring pipeline built on AWS that detects suspicious activity, filters events in real time, and delivers instant email alerts.

## Architecture

```
Secrets Manager ←── CloudTrail ──→ CloudWatch (Log Group → Filter → Alarm) ──→ SNS ──→ Email
                          │
                          ▼
                       S3 Bucket
```

![Architecture Diagram](architecture.png)

## How It Works

1. **AWS CloudTrail** — Captures all API calls and account activity across your AWS environment.
2. **Amazon S3** — CloudTrail logs are archived to an S3 bucket for long-term storage and audit purposes.
3. **AWS Secrets Manager** — Stores and manages sensitive credentials referenced by CloudTrail and other services.
4. **Amazon CloudWatch**
   - **Log Group** — Receives CloudTrail logs as a streaming data source.
   - **Metric Filter** — Parses logs and extracts metrics based on defined security patterns (e.g., unauthorized API calls, root account usage, IAM changes).
   - **Alarm** — Triggers when a metric breaches its threshold.
5. **Amazon SNS** — Receives the alarm notification and fans it out to subscribers.
6. **Email** — End recipients receive an immediate alert when a security event is detected.

## AWS Services Used

| Service | Purpose |
|---|---|
| AWS CloudTrail | API activity logging and auditing |
| Amazon S3 | Long-term log archival |
| AWS Secrets Manager | Secure credential storage |
| Amazon CloudWatch Logs | Log ingestion and storage |
| CloudWatch Metric Filters | Pattern-based event detection |
| CloudWatch Alarms | Threshold-based alerting |
| Amazon SNS | Notification fanout |

## Getting Started

### Prerequisites

- An AWS account with appropriate IAM permissions
- AWS CLI installed and configured
- (Optional) Terraform or AWS CDK if deploying via IaC

### Deployment Steps

1. **Enable CloudTrail**
   - Create a trail in the AWS Console or via CLI
   - Enable log file validation
   - Point the trail to an S3 bucket and a CloudWatch Log Group

2. **Configure Secrets Manager**
   - Store any credentials or keys needed by your pipeline

3. **Create CloudWatch Metric Filters**
   - Define filter patterns for the security events you want to monitor
   - Example patterns:
     - Root account login: `{ $.userIdentity.type = "Root" }`
     - Unauthorized API calls: `{ $.errorCode = "AccessDenied" }`
     - IAM policy changes: `{ $.eventName = "PutUserPolicy" || $.eventName = "AttachRolePolicy" }`

4. **Set Up CloudWatch Alarms**
   - Create an alarm for each metric filter
   - Set an appropriate threshold and evaluation period

5. **Create an SNS Topic and Subscription**
   - Create a new SNS topic
   - Add email subscriptions for your security team
   - Link the CloudWatch alarms to the SNS topic

## Example Metric Filter Patterns

```
# Detect root account usage
{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }

# Detect unauthorized API calls
{ ($.errorCode = "AccessDenied") || ($.errorCode = "UnauthorizedAccess") }

# Detect console login without MFA
{ $.eventName = "ConsoleLogin" && $.additionalEventData.MFAUsed != "Yes" }
```

## Security Best Practices

- Enable **MFA Delete** on the S3 log archive bucket
- Set **S3 bucket policies** to deny direct deletion of CloudTrail logs
- Use **least privilege IAM roles** for all services in the pipeline
- Rotate secrets in Secrets Manager on a regular schedule
- Set CloudWatch log retention to meet your compliance requirements

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License

[MIT](LICENSE)
