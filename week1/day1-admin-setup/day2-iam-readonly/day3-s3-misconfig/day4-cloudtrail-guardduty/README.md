# Day 4 – CloudTrail (Audit Logging) & GuardDuty (Threat Detection)

## Objective
- Enable and test **CloudTrail** for AWS API activity logging.
- Demonstrate how read actions vs denied actions appear in logs.
- Prepare for **GuardDuty** (to be added once registration completes).

---

## Part A – CloudTrail

### Steps Completed
1. Opened **CloudTrail → Event history** in AWS console.
2. Logged in as both:
   - `cl0udadmin` (admin)
   - `cl0udreadonly` (read-only)
3. Performed actions:
   - **ConsoleLogin** with both users.
   - **Read-only action**: EC2 → `DescribeRegions`.
   - **Denied actions**: IAM → `CreateUser`, S3 → `CreateBucket` as `cl0udreadonly`.
4. Retrieved event records (JSON) from CloudTrail.

---

### Evidence (CloudTrail JSON excerpts)

#### Console Login (Success)
```json
{
  "eventTime": "2025-09-22T13:08:09Z",
  "eventSource": "signin.amazonaws.com",
  "eventName": "ConsoleLogin",
  "userIdentity": {
    "arn": "arn:aws:iam::******728494:user/cl0udreadonly"
  },
  "responseElements": { "ConsoleLogin": "Success" }
}


{
  "eventTime": "2025-09-22T13:08:22Z",
  "eventSource": "ec2.amazonaws.com",
  "eventName": "DescribeRegions",
  "userIdentity": {
    "arn": "arn:aws:iam::******728494:user/cl0udreadonly"
  }
}


{
  "eventTime": "2025-09-22T13:09:07Z",
  "eventSource": "iam.amazonaws.com",
  "eventName": "CreateUser",
  "userIdentity": {
    "arn": "arn:aws:iam::******728494:user/cl0udreadonly"
  },
  "requestParameters": { "userName": "testing-readonly" },
  "errorCode": "AccessDenied"
}


{
  "eventTime": "2025-09-22T13:12:44Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "CreateBucket",
  "userIdentity": {
    "arn": "arn:aws:iam::******728494:user/cl0udreadonly"
  },
  "requestParameters": { "bucketName": "earl-test-bucket" },
  "errorCode": "AccessDenied"
}


GuardDuty registration pending. Findings to be added once enabled.
