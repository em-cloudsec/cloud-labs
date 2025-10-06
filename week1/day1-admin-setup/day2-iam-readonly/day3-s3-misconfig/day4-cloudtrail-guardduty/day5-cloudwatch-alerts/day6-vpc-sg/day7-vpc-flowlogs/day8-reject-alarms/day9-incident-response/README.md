# Day 9 – Incident Response Mini-Drill (Completed)

## Objective
Build and validate a complete end-to-end security alerting system in AWS that detects unauthorized access attempts and notifies the administrator in real time.

---

## Steps Completed
1. Logged in as `cl0udreadonly` and attempted restricted actions (`CreateUser`, `CreateBucket`).
2. CloudTrail captured the `AccessDenied` event.
3. CloudWatch metric filter detected the event using pattern:


4. Alarm `AccessDenied` triggered when `AccessDeniedCount >= 1`.
5. SNS topic `SecurityAlerts` delivered alert email.
6. Alarm transitioned: **OK → ALARM** (email received).

---

## Results
✅ Verified AccessDenied pipeline end-to-end.  
✅ SNS topic and subscription confirmed working.  
✅ First successful automated alert delivery.  

---

## Evidence
### CloudTrail Event
```json
{
"eventName": "CreateBucket",
"errorMessage": "Access Denied",
"userIdentity": {
 "arn": "arn:aws:iam::******728494:user/cl0udreadonly"
},
"sourceIPAddress": "***.***.***.106",
"eventTime": "2025-10-06T02:25:00Z"
}
