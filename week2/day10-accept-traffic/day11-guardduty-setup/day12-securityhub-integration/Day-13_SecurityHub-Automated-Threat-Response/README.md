# Day 13 – AWS Security Hub Automated Threat Response

## 1. Overview / Objective
In Day 13, we integrated **AWS Security Hub** with **EventBridge**, **SNS**, and **SSM Automation** to create an automated incident-response pipeline.

When a **HIGH** or **CRITICAL** EC2-related finding appears in Security Hub (for example, a GuardDuty detection), the system:
1. Detects and normalizes the finding in Security Hub  
2. Routes it through EventBridge  
3. Sends an alert email via SNS  
4. Executes a quarantine runbook via SSM to isolate the affected instance  
5. Logs actions in CloudTrail and stores any failures in an SQS Dead Letter Queue (DLQ)

This is the foundation of a production-grade AWS Security Orchestration, Automation, and Response (SOAR) workflow.

---

## 2. Concept Explanation

**Security Hub:** Centralizes security findings from multiple AWS services such as GuardDuty, Inspector, and IAM Access Analyzer.

**EventBridge:** Evaluates incoming Security Hub events in real time and routes them to automation or alerting targets based on filters (event patterns).

**SNS (Simple Notification Service):** Delivers alerts to email, Slack, or ticketing systems.

**SSM Automation (Systems Manager):** Executes a runbook to respond automatically (for example, quarantine an EC2 instance).

**SQS DLQ (Dead Letter Queue):** Captures failed event deliveries to ensure no loss of critical alerts.

**IAM & CloudTrail:** Provide least-privilege access, auditing, and traceability for every automation action.

---

## 3. Step-by-Step Implementation

### A) Configure Security Hub
1. Go to **Security Hub → Settings → Integrations → Enable GuardDuty** (if not already).  
2. In **Security Hub → Findings**, generate sample findings via **GuardDuty → Settings → Generate sample findings**.  

### B) Create SNS Topic
1. **SNS → Topics → Create topic**  
   - Type: Standard  
   - Name: `securityhub-alerts`
2. **Create subscription**  
   - Protocol: Email  
   - Endpoint: Your verified email address  
   - Confirm subscription via email link.

### C) Create SSM Automation Runbook – Quarantine EC2
1. **Systems Manager → Documents → Create automation document**  
2. Name: `Quarantine-EC2-AttachQSG`  
3. Paste the following YAML:

```yaml
schemaVersion: '0.3'
description: 'Attach a quarantine Security Group to an EC2 instance (accepts ARN or instance-id)'
assumeRole: '{{ AutomationAssumeRole }}'
parameters:
  InstanceId:
    type: String
    description: 'EC2 instance ID or ARN'
  QuarantineSecurityGroupId:
    type: String
    description: 'Security Group to attach'
  AutomationAssumeRole:
    type: String
    description: 'IAM role ARN for Automation to perform actions'
mainSteps:
  - name: NormalizeInstanceId
    action: 'aws:executeScript'
    inputs:
      Runtime: python3.11
      Handler: handler
      Script: |
        def handler(events, context):
            iid = events.get('InstanceId', '')
            if iid.startswith('arn:') and '/instance/' in iid:
                iid = iid.split('/instance/')[1]
            if iid.startswith('arn:') and ':instance/' in iid:
                iid = iid.split(':instance/')[1]
            if not iid.startswith('i-'):
                raise ValueError(f"Unrecognized InstanceId: {iid}")
            return {'NormalizedInstanceId': iid}
    outputs:
      - Name: NormalizedInstanceId
        Selector: '$.NormalizedInstanceId'
        Type: String

  - name: GetPrimaryEni
    action: 'aws:executeAwsApi'
    inputs:
      Service: ec2
      Api: DescribeInstances
      InstanceIds:
        - '{{ NormalizeInstanceId.NormalizedInstanceId }}'
    outputs:
      - Name: PrimaryEni
        Selector: '$.Reservations[0].Instances[0].NetworkInterfaces[0].NetworkInterfaceId'
        Type: String

  - name: AttachQuarantineSG
    action: 'aws:executeAwsApi'
    inputs:
      Service: ec2
      Api: ModifyNetworkInterfaceAttribute
      NetworkInterfaceId: '{{ GetPrimaryEni.PrimaryEni }}'
      Groups:
        - '{{ QuarantineSecurityGroupId }}'


{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:ModifyNetworkInterfaceAttribute"
      ],
      "Resource": "*"
    }
  ]
}

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ssm.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPassAutomationRoleOnlyToSSM",
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::693986728494:role/test-2automation",
      "Condition": {
        "StringEquals": { "iam:PassedToService": "ssm.amazonaws.com" }
      }
    },
    {
      "Sid": "AllowStartAutomation",
      "Effect": "Allow",
      "Action": "ssm:StartAutomationExecution",
      "Resource": "*"
    }
  ]
}

{
  "source": ["aws.securityhub"],
  "detail-type": ["Security Hub Findings - Imported"],
  "detail": {
    "findings": {
      "Severity": { "Label": ["HIGH", "CRITICAL"] },
      "Resources": { "Type": ["AwsEc2Instance"] },
      "Workflow": { "Status": ["NEW"] },
      "Sample": [false]
    }
  }
}

{ "instancearn": "$.detail.findings[0].Resources[0].Id" }

{
  "InstanceId": "<instancearn>",
  "QuarantineSecurityGroupId": "sg-REPLACE_WITH_YOUR_QSG",
  "AutomationAssumeRole": "arn:aws:iam::693986728494:role/test-2automation"
}

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEventBridgeToSendMessages",
      "Effect": "Allow",
      "Principal": { "Service": "events.amazonaws.com" },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:us-east-1:693986728494:securityhub-eb-dlq",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "arn:aws:events:us-east-1:693986728494:rule/SecurityHub-HighCritical-Alerts"
        }
      }
    }
  ]
}

schemaVersion: '0.3'
description: >
  Placeholder runbook for restoring original Security Groups on EC2 instances.
  Will be populated in Day 14.
mainSteps:
  - name: PlaceholderStep
    action: 'aws:executeScript'
    inputs:
      Runtime: python3.11
      Handler: handler
      Script: |
        def handler(event, context):
            return {"Message": "Restore-Original-SG placeholder created successfully."}

