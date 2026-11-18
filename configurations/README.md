# Configurations

This directory contains sanitized detection and alarm configuration for the AWS Security Monitoring System.

## Files

### `metric-filter-pattern.txt`

Contains the CloudWatch Logs metric-filter pattern used to identify `GetSecretValue` activity:

```text
{ $.eventName = "GetSecretValue" }
```

### `cloudtrail-event-pattern.json`

Provides a sanitized reference showing the important CloudTrail fields associated with a Secrets Manager `GetSecretValue` event.

> This file is a documentation reference. It is not a complete CloudTrail event or a deployable EventBridge event pattern.

### `alarm-configuration.md`

Documents:

- Custom metric namespace and name
- Alarm threshold
- Evaluation period
- Alarm-state behavior
- Amazon SNS notification flow
- Production improvement opportunities

## Security Notice

This directory does not contain:

- AWS credentials
- Secret values
- AWS account IDs
- User or role ARNs
- Secret ARNs
- SNS topic ARNs
- Email addresses
- Source IP addresses
- Session tokens

All configuration examples have been sanitized for public use.
