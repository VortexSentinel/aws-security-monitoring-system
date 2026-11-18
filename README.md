# AWS Security Monitoring System

## Project Overview

Built and tested an AWS security-monitoring pipeline that detects access to a secret stored in AWS Secrets Manager.

The solution uses AWS CloudTrail to record `GetSecretValue` API activity, Amazon CloudWatch Logs to analyze the recorded events, a metric filter to generate a custom security metric, a CloudWatch alarm to detect the activity, and Amazon SNS to send an email notification.

The final test confirmed that accessing the monitored secret generated a CloudTrail event, updated the custom metric, changed the CloudWatch alarm state, and delivered an SNS email alert.

---

## Project Objectives

- Create a test secret in AWS Secrets Manager
- Record AWS API activity using CloudTrail
- Send CloudTrail events to CloudWatch Logs
- Detect the `GetSecretValue` API event
- Convert matching log events into a custom security metric
- Create a CloudWatch alarm for suspicious secret access
- Send an email notification through Amazon SNS
- Test and validate the complete monitoring workflow
- Document troubleshooting and security considerations

---

## Architecture

```text
AWS Secrets Manager
        │
        │ GetSecretValue API call
        ▼
AWS CloudTrail
        │
        │ Management event
        ▼
Amazon CloudWatch Logs
        │
        │ Metric filter
        ▼
Custom CloudWatch Metric
SecurityMetrics / SecretAccessCount
        │
        │ Threshold: >= 1
        ▼
CloudWatch Alarm
        │
        ▼
Amazon SNS Topic
        │
        ▼
Email Notification
```

---

## AWS Services and Tools Used

| Service or tool | Purpose |
|---|---|
| AWS Secrets Manager | Store the test secret and generate monitored access activity |
| AWS CloudTrail | Record the `GetSecretValue` API event |
| Amazon CloudWatch Logs | Store and analyze CloudTrail events |
| CloudWatch Metric Filter | Match secret-access events in the log stream |
| Amazon CloudWatch Metrics | Track the number of matching security events |
| Amazon CloudWatch Alarms | Detect when the custom security metric meets the threshold |
| Amazon SNS | Send an email notification when the alarm activates |
| AWS Management Console | Configure, test, and validate the monitoring pipeline |

---

## Security Concepts Demonstrated

- Security-event logging
- AWS API activity monitoring
- Audit-trail configuration
- Log-based threat detection
- Custom security metrics
- Alarm threshold configuration
- Event-driven notifications
- Monitoring sensitive resource access
- Detection and alert validation
- Security troubleshooting

---

## Monitoring Workflow

1. A user or AWS identity retrieves a secret from AWS Secrets Manager.
2. Secrets Manager processes the `GetSecretValue` API request.
3. AWS CloudTrail records the API activity as a management event.
4. CloudTrail sends the recorded event to Amazon CloudWatch Logs.
5. A CloudWatch metric filter searches for matching `GetSecretValue` events.
6. A matching event increments the custom `SecretAccessCount` metric.
7. The CloudWatch alarm evaluates the metric against its threshold.
8. When the metric value is greater than or equal to `1`, the alarm enters the `ALARM` state.
9. The alarm publishes a message to an Amazon SNS topic.
10. Amazon SNS sends the alert to the confirmed email subscription.

---

## Monitored Activity

The monitoring system detects the following AWS API event:

```text
GetSecretValue
```

This API call occurs when an authorized AWS identity retrieves the value of a secret from AWS Secrets Manager.

Monitoring this activity can help identify:

- Unexpected secret retrieval
- Access outside normal application workflows
- Repeated access attempts
- Activity from an unfamiliar AWS identity
- Possible credential misuse

> **Important:** A `GetSecretValue` event is not automatically malicious. The alert indicates that sensitive activity occurred and should be reviewed with additional context, including the identity, source IP address, user agent, resource, and event time.

---

## CloudTrail Configuration

AWS CloudTrail was configured to record management events and deliver them to Amazon CloudWatch Logs.

### CloudTrail Responsibilities

- Record AWS API calls
- Capture the event name
- Record the requesting identity
- Record the event time
- Record the source IP address
- Record the AWS Region
- Send events to the configured CloudWatch log group

Sensitive account identifiers, ARNs, IP addresses, and resource names are removed from the public screenshots.

---

## Metric Filter Configuration

A CloudWatch Logs metric filter was created to identify CloudTrail events containing the `GetSecretValue` API event name.

The sanitized filter configuration is documented in:

configurations/metric-filter-pattern.txt

### Metric Configuration

| Setting | Value |
|---|---|
| Metric namespace | `SecurityMetrics` |
| Metric name | `SecretAccessCount` |
| Metric value for a match | `1` |
| Default value | `0` |

Each matching event contributes a value of `1` to the custom metric.

---

## Alarm Configuration

The CloudWatch alarm monitors the custom `SecretAccessCount` metric.

| Setting | Value |
|---|---|
| Metric namespace | `SecurityMetrics` |
| Metric name | `SecretAccessCount` |
| Threshold | Greater than or equal to `1` |
| Evaluation period | 5 minutes |
| Alarm action | Publish to Amazon SNS |

The sanitized alarm configuration is documented in:

configurations/alarm-configuration.md

> **Response-time note:** Because the alarm uses a five-minute evaluation period and AWS services require time to deliver and process events, the notification might not arrive immediately after the secret is accessed.

---

## SNS Notification Configuration

An Amazon SNS topic was connected to the CloudWatch alarm.

An email subscription was created and confirmed before testing the alert.

### Notification Flow

```text
CloudWatch Alarm
        │
        ▼
Amazon SNS Topic
        │
        ▼
Confirmed Email Subscription
```

The recipient email address and SNS topic ARN are not included in this public repository.

---

## Implementation Walkthrough

### 1. Create a Test Secret

A sample secret was created in AWS Secrets Manager to provide a controlled resource for monitoring.

No real production credentials or sensitive values are stored in this repository.

### 2. Configure CloudTrail

A CloudTrail trail was configured to record AWS management events and deliver the events to a CloudWatch log group.

### 3. Generate Secret-Access Activity

The test secret was retrieved to generate a `GetSecretValue` API event.

### 4. Locate the CloudTrail Event

The corresponding event was identified in CloudTrail and CloudWatch Logs.

The event confirmed that secret-access activity was successfully recorded.

### 5. Create the Metric Filter

A CloudWatch Logs metric filter was configured to identify matching `GetSecretValue` events.

### 6. Create the Custom Metric

Matching events generated the custom metric:

```text
SecurityMetrics / SecretAccessCount
```

### 7. Configure the CloudWatch Alarm

A CloudWatch alarm was created with a threshold of:

```text
SecretAccessCount >= 1
```

### 8. Configure the SNS Topic

An SNS topic and email subscription were created.

The email subscription was confirmed before the final alert test.

### 9. Test the Monitoring Pipeline

The test secret was accessed again to generate a new monitored event.

The alarm later entered the `ALARM` state and sent an email notification through Amazon SNS.

---

## Project Images

> **Security notice:** AWS account IDs, ARNs, email addresses, secret names, log-group identifiers, IP addresses, and other environment-specific details must be redacted before screenshots are uploaded.

Screenshots will be added in implementation order:

1. Security-monitoring architecture
2. Secrets Manager test secret
3. CloudTrail configuration
4. CloudWatch log group
5. `GetSecretValue` event
6. CloudWatch metric filter
7. CloudWatch alarm
8. SNS subscription
9. Security-alert email

---

## Testing and Validation

### Test Action

The monitored test secret was retrieved from AWS Secrets Manager.

### Expected Result

The monitoring pipeline should:

1. Record the `GetSecretValue` event in CloudTrail.
2. Deliver the event to CloudWatch Logs.
3. Match the event using the metric filter.
4. Increment the `SecretAccessCount` metric.
5. Trigger the CloudWatch alarm.
6. Publish a message to Amazon SNS.
7. Send an email notification.

### Actual Result

The monitoring workflow completed successfully and the SNS email alert was received.

| Validation step | Expected result | Result |
|---|---|---|
| CloudTrail records secret access | Event recorded | Passed |
| Event appears in CloudWatch Logs | Event delivered | Passed |
| Metric filter matches the event | Metric updated | Passed |
| Alarm reaches threshold | Alarm activated | Passed |
| SNS publishes notification | Message published | Passed |
| Email alert is delivered | Email received | Passed |

---

## Troubleshooting

### SNS Subscription Remained Pending

The SNS email subscription remained in the `Pending confirmation` state until the confirmation link in the subscription email was opened.

#### Resolution

- Checked the recipient inbox and spam folder
- Opened the AWS subscription-confirmation email
- Selected the confirmation link
- Verified that the SNS subscription status changed to `Confirmed`

### Alarm Did Not Trigger Immediately

The alarm did not change state immediately after the test secret was accessed.

#### Explanation

Several stages require processing time:

- CloudTrail records the API activity
- CloudTrail delivers the event to CloudWatch Logs
- The metric filter evaluates the event
- CloudWatch updates the custom metric
- The alarm evaluates the metric during its configured period
- SNS processes the alarm notification

#### Resolution

Waited for the five-minute evaluation period and verified each stage of the monitoring pipeline individually.

---

## Security Measures

- Used a sample secret instead of production credentials
- Monitored sensitive `GetSecretValue` activity
- Recorded API events through CloudTrail
- Sent audit events to CloudWatch Logs
- Used a log metric filter for automated detection
- Used a custom metric and alarm threshold
- Required confirmation for the SNS email subscription
- Redacted identifying information from public screenshots
- Excluded secret values and AWS credentials from the repository

---

## Challenges Faced

### Understanding the Complete Event Flow

The project required understanding how several AWS services work together.

A secret-access event does not trigger an email directly. The event must pass through:

```text
Secrets Manager
→ CloudTrail
→ CloudWatch Logs
→ Metric Filter
→ Custom Metric
→ CloudWatch Alarm
→ SNS
→ Email
```

Testing each component separately made it easier to identify where delays or configuration problems occurred.

### Distinguishing Detection from Prevention

This pipeline detects and reports secret access. It does not prevent the API call.

Access prevention must be handled separately through controls such as:

- IAM policies
- Service Control Policies
- Resource policies
- Permission boundaries
- Multi-factor authentication
- Network and endpoint policies

---

## Key Learnings

This project provided practical experience with:

- AWS CloudTrail management events
- CloudWatch log groups
- Log metric filters
- Custom CloudWatch metrics
- CloudWatch alarm evaluation
- Amazon SNS topics and subscriptions
- Secrets Manager activity monitoring
- End-to-end security alert testing
- Monitoring-pipeline troubleshooting
- Detection versus prevention

---

## Business Value

This monitoring pattern helps organizations detect access to sensitive secrets and notify the appropriate team.

The solution can help:

- Improve visibility into sensitive API activity
- Reduce the time required to detect unusual access
- Support security investigations
- Provide evidence for audit and compliance reviews
- Create a foundation for automated incident response
- Standardize security notifications across AWS environments

---

## Limitations

- The system alerts on every matching `GetSecretValue` event unless the filter is refined.
- Authorized application access may generate expected alerts.
- A five-minute evaluation period introduces detection delay.
- Email alerts alone may not be sufficient for production incident response.
- CloudTrail, CloudWatch Logs, custom metrics, alarms, SNS, and Secrets Manager can generate AWS charges outside eligible usage allowances.

---

## Future Improvements

- Refine the filter using AWS identity or resource context
- Send alerts to Microsoft Teams, Slack, or an incident-management platform
- Add AWS Lambda for automated response actions
- Use Amazon EventBridge for event-driven routing
- Add CloudWatch dashboards for security-event visualization
- Enable CloudTrail log-file validation
- Encrypt log data using an AWS KMS customer-managed key
- Protect CloudTrail logs using a dedicated Amazon S3 bucket
- Add alert-severity levels
- Deploy the monitoring system through Terraform
- Integrate findings into AWS Security Hub
- Create an incident-response runbook

---

## Repository Structure

```text
aws-security-monitoring-system/
├── README.md
├── LICENSE
├── images/
│   └── README.md
├── configurations/
│   ├── README.md
│   ├── cloudtrail-event-pattern.json
│   ├── metric-filter-pattern.txt
│   └── alarm-configuration.md
└── documentation/
    ├── README.md
    ├── implementation-notes.md
    ├── alert-testing.md
    └── troubleshooting.md
```

---

## Original Project Reference

This project was originally completed as part of the NextWork Cloud Engineering learning path.

The repository was expanded with:

- Professional architecture documentation
- Monitoring-flow explanations
- Sanitized configuration files
- Testing and validation results
- Troubleshooting documentation
- Security limitations
- Business-value analysis
- Future security improvements

These additions better demonstrate practical AWS security-monitoring and incident-detection skills.

---

## Project Status

✅ Completed

---

## Author

### Rajnish Jha

Aspiring Cloud and DevOps Engineer focused on AWS, Linux, Docker, Git, cloud security, monitoring, and automation.

### Connect With Me

- [LinkedIn](https://www.linkedin.com/in/rajnish-k-jha)
- [GitHub](https://github.com/VortexSentinel)
- [Email](mailto:rajnish96jha@gmail.com)
