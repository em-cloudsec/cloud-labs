# Day 11 – Amazon GuardDuty: Threat Detection & Response

## Objective
Enable GuardDuty and simulate threat detection to understand AWS-native intrusion detection capabilities.

---

## Steps Completed
1. Opened **Amazon GuardDuty** in AWS Console.
2. Enabled GuardDuty for account/region.
3. Generated **sample findings** to simulate malicious events.
4. Analyzed findings such as:
   - `Recon:EC2/Portscan`
   - `UnauthorizedAccess:IAMUser/ConsoleLogin`
   - `Trojan:EC2/BlackholeTraffic`

---

# Day 11 – GuardDuty Sample Finding (Masked JSON)

## Objective
Demonstrate how Amazon GuardDuty correlates multiple suspicious signals (API misuse, Tor access, MITRE ATT&CK mapping) into a single threat sequence representing a potential IAM credential compromise.

---

## Masked Example JSON
```json
[
  {
    "AccountId": "********8494",
    "Arn": "arn:aws:guardduty:us-east-1:********8494:detector/************/finding/************",
    "CreatedAt": "2025-10-14T03:10:43.791Z",
    "Description": "A sequence of actions involving multiple signals indicating a possible credential compromise was observed for IAMUser/<redacted>.",
    "Id": "************",
    "Region": "us-east-1",
    "Resource": {
      "ResourceType": "AttackSequence"
    },
    "Service": {
      "Detection": {
        "Sequence": {
          "Actors": [
            {
              "User": {
                "Name": "<redacted_user>",
                "Type": "IAMUser"
              }
            }
          ],
          "Endpoints": [
            {
              "Ip": "10.0.0.1",
              "Connection": { "Direction": "INBOUND" },
              "Location": { "City": "New York", "Country": "US" }
            }
          ],
          "SequenceIndicators": [
            {
              "Key": "ATTACK_TACTIC",
              "Values": ["Defense Evasion", "Persistence"]
            },
            {
              "Key": "HIGH_RISK_API",
              "Values": [
                "cloudtrail:DeleteTrail",
                "iam:AttachRolePolicy",
                "iam:CreateRole",
                "iam:ListUsers"
              ]
            },
            {
              "Key": "TOR_IP",
              "Values": ["10.0.0.1"]
            },
            {
              "Key": "ATTACK_TECHNIQUE",
              "Values": [
                "T1078.004 - Valid Accounts: Cloud Accounts",
                "T1087.004 - Account Discovery: Cloud Account",
                "T1098 - Account Manipulation",
                "T1562.008 - Impair Defenses: Disable or Modify Cloud Logs"
              ]
            }
          ]
        }
      },
      "DetectorId": "************",
      "FeatureName": "Correlation",
      "ServiceName": "guardduty"
    },
    "Severity": 9,
    "Title": "Potential credential compromise of IAMUser/<redacted> indicated by a sequence of actions.",
    "Type": "AttackSequence:IAM/CompromisedCredentials",
    "UpdatedAt": "2025-10-14T03:10:43.791Z"
  }
]


# Day 11.5 – AWS Billing Alarm Setup (Cost Protection)

## Objective
Protect against unexpected AWS charges by creating a CloudWatch billing alarm that triggers email alerts when total costs exceed a safe threshold.

---

## Steps Completed
1. Switched to region **us-east-1 (N. Virginia)**.
2. Opened **CloudWatch → Alarms → Create alarm**.
3. Selected metric:
   - **Billing → Total Estimated Charge → USD → EstimatedCharges**
4. Set condition:
   - `EstimatedCharges >= $5`
5. Configured SNS action:
   - Topic: `BillingAlerts`
   - Email: verified subscription
6. Named alarm: `AWS-Billing-Alarm`
7. Verified test email and alarm status.

---

## Key Learnings
- Billing metrics are only available in **us-east-1**.
- CloudWatch alarms + SNS = automated cost monitoring.
- Demonstrates **FinOps discipline** — a critical cloud skill.
- Can reuse the same SNS topic for multiple alert categories.

---

## Example Configuration
