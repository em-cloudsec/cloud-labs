# Day 5 – CloudWatch Logs, Metric Filter & Alarm (AccessDenied)

## Objective
- Detect unauthorized activity in AWS using **CloudTrail + CloudWatch + SNS**.
- Convert AccessDenied log events into a metric → trigger an alarm → notify via email.

---

## Steps Completed
1. Linked CloudTrail logs to CloudWatch Log Group:  
   `aws-cloudtrail-logs-******728494-<suffix>`

2. Created **Metric Filter**:
   - Name: `AccessDeniedFilter`
   - Pattern:
     ```
     { ($.errorCode = "*Unauthorized*") || ($.errorCode = "AccessDenied*") }
     ```
   - Metric Namespace: `Security`
   - Metric Name: `AccessDeniedCount`
   - Value: `1`

3. Created **CloudWatch Alarm**:
   - Name: `AccessDenied`
   - Condition: `AccessDeniedCount >= 1` for 1 datapoint in 5 minutes
   - State changes: OK → ALARM → OK
   - Action: send notification via SNS topic `SecurityAlerts`

4. Configured **SNS Topic**:
   - Name: `SecurityAlerts`
   - Subscribed email: **confirmed**

5. **Tested** by logging in as `cl0udreadonly`:
   - Attempted IAM action: `CreateUser`
   - Result: **AccessDenied**
   - Alarm triggered
   - SNS delivered email notification

---

## Results
- ✅ Unauthorized actions now create CloudWatch metric datapoints.
- ✅ Alarm transitions to **ALARM** on first AccessDenied event.
- ✅ SNS sends alert email to subscribed address.
- ⏳ Alarm resets to **OK** automatically after a new period with no AccessDenied events.

---

## Key Learnings
- CloudTrail + CloudWatch → powerful pipeline for **detection**.
- **Metric Filters** turn log events into numbers that alarms can watch.
- **SNS** enables fast response with email/text/webhook notifications.
- Default “greater than” threshold may miss single events — use **>= 1** for better coverage.
- Alarms remain in **ALARM** until a fresh period without violations clears them.

---

## Evidence
### CloudTrail JSON Snippet
```json
{
  "eventTime": "2025-09-28T21:20:04Z",
  "eventSource": "iam.amazonaws.com",
  "eventName": "CreateUser",
  "userIdentity": {
    "arn": "arn:aws:iam::******728494:user/cl0udreadonly"
  },
  "errorCode": "AccessDenied"
}

