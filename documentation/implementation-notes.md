# Implementation Notes

## Project Objective

Build an AWS security-monitoring pipeline that detects access to a secret stored in AWS Secrets Manager and sends an email alert.

## Monitoring Architecture

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
Amazon SNS
        │
        ▼
Email Notification
```

## AWS Services Used

| Service | Implementation role |
|---|---|
| AWS Secrets Manager | Provided a controlled secret-access event |
| AWS CloudTrail | Recorded the `GetSecretValue` API call |
| Amazon CloudWatch Logs | Received and stored CloudTrail events |
| CloudWatch Metric Filter | Detected matching secret-access events |
| Amazon CloudWatch Metrics | Recorded matches as `SecretAccessCount` |
| Amazon CloudWatch Alarm | Evaluated the metric against the threshold |
| Amazon SNS | Sent the email notification |

## Implementation Process

### 1. Create a Sample Secret

A sample secret was created in AWS Secrets Manager.

The secret was used only to generate controlled monitoring events. No production credentials or sensitive information are included in this repository.

### 2. Configure AWS CloudTrail

AWS CloudTrail was configured to record management events.

The trail delivered recorded activity to an Amazon CloudWatch Logs log group.

### 3. Generate the Monitored Event

The sample secret was retrieved, generating the API event:

```text
GetSecretValue
```

### 4. Verify the CloudTrail Event

The event was located and reviewed to confirm that CloudTrail recorded the secret-access activity.

Important event context included:

- Event name
- Event time
- AWS identity
- AWS Region
- Source IP address
- User agent
- Requested resource

Sensitive values have not been included in the public repository.

### 5. Configure the Metric Filter

A CloudWatch Logs metric filter was configured with:

```text
{ $.eventName = "GetSecretValue" }
```

The filter searches incoming CloudTrail log events for the monitored API action.

### 6. Configure the Custom Metric

A matching event generated the following custom metric:

```text
Namespace: SecurityMetrics
Metric name: SecretAccessCount
Metric value: 1
Default value: 0
```

### 7. Configure the CloudWatch Alarm

The alarm was configured to enter the `ALARM` state when:

```text
SecretAccessCount >= 1
```

The evaluation period was configured as five minutes.

### 8. Configure Amazon SNS

An SNS topic was created and connected to the CloudWatch alarm.

An email subscription was then added and confirmed.

### 9. Run the End-to-End Test

The sample secret was retrieved again.

The monitoring pipeline processed the event through CloudTrail, CloudWatch Logs, the metric filter, the custom metric, the alarm, and Amazon SNS.

### 10. Verify the Result

The CloudWatch alarm entered the `ALARM` state and the SNS notification was delivered to the confirmed email subscription.

## Final Outcome

The project successfully demonstrated an end-to-end AWS security-monitoring workflow for sensitive Secrets Manager activity.

## Security Considerations

- A sample secret was used instead of production credentials.
- Secret values are not stored in the repository.
- AWS account identifiers and ARNs are excluded.
- Email addresses are redacted.
- Screenshots must be reviewed before publication.
- The alert detects activity but does not block the API request.
