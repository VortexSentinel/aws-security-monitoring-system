# CloudWatch Alarm Configuration

## Purpose

The CloudWatch alarm detects when the custom `SecretAccessCount` metric records at least one matching Secrets Manager access event.

## Metric Configuration

| Setting | Value |
|---|---|
| Namespace | `SecurityMetrics` |
| Metric name | `SecretAccessCount` |
| Metric source | CloudWatch Logs metric filter |
| Matching event | `GetSecretValue` |
| Metric value for a match | `1` |
| Default value | `0` |

## Alarm Conditions

| Setting | Value |
|---|---|
| Comparison operator | Greater than or equal to threshold |
| Threshold | `1` |
| Evaluation period | `5 minutes` |
| Datapoints to alarm | `1` |
| Alarm action | Publish notification to Amazon SNS |

## Detection Logic

```text
GetSecretValue event detected
        │
        ▼
SecretAccessCount increases by 1
        │
        ▼
Metric value >= 1
        │
        ▼
CloudWatch alarm enters ALARM state
        │
        ▼
SNS notification is published
        │
        ▼
Email alert is delivered
```

## Expected States

### OK

The metric has not met the configured alarm threshold during the evaluated period.

### ALARM

At least one matching event has caused the metric to meet or exceed the threshold.

### Insufficient Data

CloudWatch does not yet have enough metric data to evaluate the alarm.

## Evaluation Delay

The alert might not appear immediately because the event passes through several services:

1. Secrets Manager processes the API request.
2. CloudTrail records the event.
3. CloudTrail delivers the event to CloudWatch Logs.
4. The metric filter evaluates the event.
5. CloudWatch publishes the custom metric.
6. The alarm evaluates the metric.
7. Amazon SNS publishes the notification.

The configured five-minute evaluation period can introduce additional delay.

## SNS Configuration

The alarm publishes notifications to an Amazon SNS topic with a confirmed email subscription.

The following details are intentionally excluded from this public repository:

- SNS topic ARN
- AWS account ID
- Recipient email address
- Subscription ARN

## Production Considerations

For a production environment, this alarm could be improved by:

- Filtering by a specific secret ARN
- Filtering by AWS identity or role
- Excluding approved application roles
- Creating warning and critical thresholds
- Sending alerts to AWS Security Hub
- Routing alerts through Amazon EventBridge
- Invoking an AWS Lambda response function
- Integrating with Microsoft Teams, Slack, PagerDuty, or ServiceNow
