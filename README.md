# Root User Activity Monitoring Solution

This CloudFormation project deploys a **multi-account root user activity monitoring solution**. It provides centralized monitoring of root account activities within an AWS organization. Root user logins are identified and alerted, ensuring heightened security and visibility across AWS accounts.

---

## Features
- **Centralized Monitoring**: Captures root activity events in a designated monitoring account.
- **Alert System**: Sends email notifications for root logins using Amazon SNS.
- **Multi-Account Support**: Member accounts forward events to the central monitoring account.
- **Secure Architecture**: Implements IAM roles with least privilege for event forwarding and Lambda execution.
- **Automation**: Detects and alerts on root user logins with no manual intervention.

---

## Architecture Overview
The solution comprises two main components:

1. **Monitoring Account Resources**:
   - **Custom Event Bus**: Receives root activity events from member accounts.
   - **Lambda Function**: Processes events and sends email alerts.
   - **SNS Topic**: Distributes email notifications to specified recipients.

2. **Member Account Resources**:
   - **Event Rule**: Captures root activity events locally.
   - **IAM Role**: Forwards these events to the central monitoring account.

---

## Prerequisites
1. **AWS Organization**: Ensure all accounts are part of a centralized AWS Organization.
2. **Email Address**: Valid email address for receiving alerts.
3. **CloudTrail**: Enabled across all accounts to capture root login events.

---

## Parameters
| Parameter              | Description                                 | Default                 |
|------------------------|---------------------------------------------|-------------------------|
| `EmailIdtoNotify`      | Email address for receiving notifications. | `example@yourcorp.com` |
| `MonitoringAccountId`  | AWS Account ID for central monitoring.     | None (Required)         |
| `OrganizationId`       | AWS Organization ID.                       | None (Required)         |

---

## Deployment Instructions

### Step 1: Prepare AWS Accounts
1. Enable AWS CloudTrail in all accounts.
2. Ensure the monitoring account has the required IAM permissions to manage resources across the organization.

### Step 2: Deploy the CloudFormation Template
1. Upload the CloudFormation template to an S3 bucket or use it directly from your local environment.
2. Deploy the stack in both:
   - The **Monitoring Account**.
   - Each **Member Account**.

### Monitoring Account Deployment
```bash
aws cloudformation deploy \
  --template-file root-monitoring-solution.yaml \
  --stack-name RootActivityMonitoring \
  --parameter-overrides \
    EmailIdtoNotify="your-email@example.com" \
    MonitoringAccountId="123456789012" \
    OrganizationId="o-xxxxxxxxxx" \
  --capabilities CAPABILITY_NAMED_IAM
```

### Member Account Deployment
```bash
aws cloudformation deploy \
  --template-file root-monitoring-solution.yaml \
  --stack-name RootActivityMonitoring \
  --parameter-overrides \
    MonitoringAccountId="123456789012" \
    OrganizationId="o-xxxxxxxxxx" \
  --capabilities CAPABILITY_NAMED_IAM
```

### Step 3: Confirm Email Subscription
- Check your email inbox for a confirmation message from Amazon SNS.
- Confirm the subscription to start receiving alerts.

---

## Outputs
| Output Name               | Description                                            |
|---------------------------|--------------------------------------------------------|
| `CentralEventBusArn`      | ARN of the central event bus (monitoring account).     |
| `LambdaRoleArn`           | ARN of the Lambda execution role (monitoring account). |
| `EventForwardingRoleArn`  | ARN of the forwarding role (member account).           |
| `LambdaFunctionArn`       | ARN of the Lambda function (monitoring account).       |

---

## Event Flow
1. **Root Login Detected**:
   - CloudTrail logs a root login event in any account.
   - EventBridge captures the event.

2. **Member Account Event Forwarding**:
   - Member accounts forward the event to the monitoring account.

3. **Central Processing**:
   - The central account's EventBridge triggers a Lambda function.
   - The Lambda function sends an alert via Amazon SNS.

---

## Security Best Practices
1. **Enable MFA**: Ensure root accounts have Multi-Factor Authentication enabled.
2. **Limit Root Access**: Use root accounts only for account setup and critical operations.
3. **Monitor Regularly**: Review logs and alerts for any unauthorized activity.

---

## Troubleshooting
- **No Notifications Received**:
  - Check if the email subscription is confirmed.
  - Verify that the root login event is being logged by CloudTrail.

- **Member Account Events Not Forwarding**:
  - Confirm that the `EventForwardingRole` exists and has correct permissions.
  - Ensure the `OrganizationId` matches between the template and AWS Organization.

---

## License
This project is licensed under the MIT License. See the LICENSE file for details.

