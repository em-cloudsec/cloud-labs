# Day 2 – IAM Permissions Lab (ReadOnly User)

## Objective
- Prove the **Principle of Least Privilege** in AWS.
- Compare Admin vs. ReadOnly permissions.

## Steps Completed
1. Logged in as `cl0udadmin` (Admin IAM user).
2. Created IAM user `cl0udreadonly` with:
   - Console login enabled
   - ReadOnlyAccess policy attached
3. Logged in as `cl0udreadonly` and tested actions.

## Results
- ✅ Able to view IAM users, EC2 regions, and other resources.
- ❌ Attempted to create a new IAM user → *Access Denied*.

### Example AccessDenied Error Output
```text
User: arn:aws:iam::<******728494>:user/cl0udreadonly
Action: iam:CreateUser
Resource: arn:aws:iam::<******728494>:user/testing-readonly
Result: AccessDenied – no identity-based policy allows the action

Event: ListNotificationHubs
Time: September 22, 2025, 13:09:15 (UTC-07:00)
Service: notifications.amazonaws.com

Event: ListAnalyzers
Time: September 22, 2025, 13:09:07 (UTC-07:00)
Service: access-analyzer.amazonaws.com

Event: DescribeRegions
Time: September 22, 2025, 13:08:22 (UTC-07:00)
Service: ec2.amazonaws.com

Event: ConsoleLogin
Time: September 22, 2025, 13:08:09 (UTC-07:00)
Service: signin.amazonaws.com
