# Troubleshooting Guide

## Overview

This document records common issues that can prevent the AWS security-monitoring pipeline from generating an email alert.

## Issue 1: SNS Subscription Remains Pending

### Symptom

The email subscription displays:

```text
Pending confirmation
```

### Cause

The recipient has not confirmed the subscription.

### Resolution

1. Check the email inbox.
2. Check the spam or junk folder.
3. Open the AWS notification email.
4. Select the subscription-confirmation link.
5. Return to Amazon SNS.
6. Confirm that the subscription status is `Confirmed`.

## Issue 2: CloudTrail Event Is Not Visible

### Possible causes

- CloudTrail is not recording management events.
- The event has not yet been delivered.
- The wrong AWS Region is selected.
- The wrong trail or event history is being reviewed.
- The monitored secret was not retrieved.

### Resolution

1. Confirm the trail is enabled.
2. Confirm management events are included.
3. Verify the AWS Region.
4. Retrieve the sample secret again.
5. Search for the event name `GetSecretValue`.
6. Allow time for event delivery.

## Issue 3: Event Does Not Appear in CloudWatch Logs

### Possible causes

- CloudTrail is not connected to the intended log group.
- The CloudTrail service role does not have the required permissions.
- The wrong log group is being reviewed.
- Log delivery is delayed.

### Resolution

1. Open the CloudTrail configuration.
2. Verify the configured CloudWatch log group.
3. Verify the CloudTrail role.
4. Open the correct log stream.
5. Search for `GetSecretValue`.
6. Allow time for delivery.

## Issue 4: Metric Filter Does Not Match

### Expected filter

```text
{ $.eventName = "GetSecretValue" }
```

### Possible causes

- The event name is misspelled.
- The filter uses the wrong JSON field.
- The event was generated before the filter was created.
- The filter was attached to the wrong log group.

### Resolution

1. Verify the exact capitalization of `GetSecretValue`.
2. Confirm the JSON field is `eventName`.
3. Confirm the filter is attached to the CloudTrail log group.
4. Generate a new secret-access event after creating the filter.
5. Use the CloudWatch filter-testing feature with a sanitized sample event.

## Issue 5: Custom Metric Is Missing

### Possible causes

- No event has matched the metric filter.
- The wrong namespace is selected.
- The wrong AWS Region is selected.
- Metric creation is delayed.

### Resolution

1. Generate a new `GetSecretValue` event.
2. Wait for CloudTrail and CloudWatch Logs delivery.
3. Open CloudWatch Metrics.
4. Select the `SecurityMetrics` namespace.
5. Check for `SecretAccessCount`.
6. Confirm the AWS Region.

## Issue 6: Alarm Does Not Enter ALARM State

### Possible causes

- The threshold is configured incorrectly.
- The evaluation period has not completed.
- The custom metric did not receive a value.
- The alarm monitors the wrong namespace or metric.
- Missing-data treatment affects evaluation.

### Resolution

1. Confirm the metric name is `SecretAccessCount`.
2. Confirm the namespace is `SecurityMetrics`.
3. Confirm the threshold is greater than or equal to `1`.
4. Confirm the evaluation period is five minutes.
5. Check the metric graph for a matching value.
6. Wait for the evaluation period to complete.

## Issue 7: Alarm Activates but No Email Arrives

### Possible causes

- The SNS subscription is not confirmed.
- The alarm is connected to the wrong SNS topic.
- The email was routed to spam.
- The SNS topic policy prevents publication.
- The alarm action is disabled.

### Resolution

1. Verify the subscription status is `Confirmed`.
2. Verify the alarm action references the correct SNS topic.
3. Check the spam or junk folder.
4. Review the SNS topic access policy.
5. Confirm alarm actions are enabled.
6. Review SNS delivery status if available.

## Issue 8: Too Many Alerts

### Cause

The filter matches every `GetSecretValue` event, including authorized application activity.

### Improvements

- Filter by a specific secret ARN.
- Filter by a specific AWS identity.
- Exclude approved application roles.
- Create separate metrics for human and application access.
- Route expected and unexpected activity to different alarm thresholds.
- Use Amazon EventBridge for more detailed event matching.

## Investigation Order

When an alert fails, verify the pipeline in this order:

```text
1. Was the secret retrieved?
2. Did CloudTrail record the event?
3. Did CloudWatch Logs receive the event?
4. Did the metric filter match?
5. Did the custom metric update?
6. Did the alarm change state?
7. Did SNS publish the notification?
8. Was the email delivered?
```

Checking each service in order helps identify the exact stage where the workflow failed.
