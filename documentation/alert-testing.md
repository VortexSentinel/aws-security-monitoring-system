# Alert Testing and Validation

## Testing Objective

Verify that retrieving a secret from AWS Secrets Manager produces a complete monitoring and notification workflow.

## Test Action

The controlled sample secret was accessed using the Secrets Manager `GetSecretValue` operation.

## Expected Event Flow

```text
Secret retrieved
      │
      ▼
CloudTrail event recorded
      │
      ▼
Event delivered to CloudWatch Logs
      │
      ▼
Metric filter matches GetSecretValue
      │
      ▼
SecretAccessCount increases
      │
      ▼
CloudWatch alarm enters ALARM state
      │
      ▼
SNS publishes notification
      │
      ▼
Email alert is received
```

## Validation Checklist

| Validation stage | Expected result | Result |
|---|---|---|
| Secrets Manager access | Test secret retrieved | Passed |
| CloudTrail recording | `GetSecretValue` event recorded | Passed |
| Log delivery | Event appears in CloudWatch Logs | Passed |
| Metric filter | Event matches filter pattern | Passed |
| Custom metric | `SecretAccessCount` receives value `1` | Passed |
| Alarm evaluation | Threshold `>= 1` is reached | Passed |
| Alarm state | Alarm enters `ALARM` | Passed |
| SNS publication | Notification published | Passed |
| Email delivery | Alert email received | Passed |

## Event Validation

The CloudTrail event was checked for:

- `eventSource` equal to `secretsmanager.amazonaws.com`
- `eventName` equal to `GetSecretValue`
- Management-event classification
- Event timestamp
- Requesting AWS identity
- Source IP address
- AWS Region

Environment-specific values were removed from public documentation.

## Metric Validation

The custom metric was checked in the namespace:

```text
SecurityMetrics
```

The monitored metric was:

```text
SecretAccessCount
```

A matching event contributed:

```text
1
```

## Alarm Validation

The alarm condition was:

```text
SecretAccessCount >= 1
```

The alarm was expected to enter the `ALARM` state after the matching event was processed and evaluated.

## SNS Validation

The SNS subscription was checked to confirm that its status was:

```text
Confirmed
```

A subscription that remains in `Pending confirmation` cannot receive alarm notifications.

## Email Validation

The delivered alert was checked for:

- CloudWatch alarm name
- New alarm state
- State-change reason
- AWS Region
- Metric name
- Threshold information
- Event timestamp

The recipient email address and account-specific details are excluded from this repository.

## Final Result

The end-to-end monitoring test passed.

The system detected the controlled secret-access event and successfully delivered an email notification through Amazon SNS.
