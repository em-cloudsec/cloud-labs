# Day 12 – AWS Security Hub Integration & Automated EC2 Quarantine

## Overview

This lab implements a real-world incident response workflow using AWS native services.  
When Security Hub or GuardDuty detects a high-severity finding against an EC2 instance, an AWS Systems Manager (SSM) Automation runbook can automatically isolate that instance by replacing its security groups with a restrictive quarantine group.

The goal is to establish a cloud-native detection-to-response pipeline that applies least privilege, is fully auditable, and can be reused across environments.

---

## Objectives

1. Enable AWS Security Hub and integrate it with GuardDuty.  
2. Create an SSM Automation runbook to quarantine EC2 instances.  
3. Configure an execution IAM role with least-privilege permissions.  
4. Set up PassRole delegation so analysts can trigger automated response actions.  
5. Test and validate the workflow by quarantining a live EC2 test instance.

---

## Architecture Summary

| Component | Purpose |
|------------|----------|
| **AWS Security Hub** | Aggregates security findings from GuardDuty and compliance standards. |
| **Amazon EventBridge** | Routes Security Hub events to response targets. |
| **AWS Systems Manager Automation** | Executes incident response playbooks. |
| **IAM Execution Role** | Grants SSM the specific EC2 actions required to isolate resources. |
| **IAM PassRole Policy** | Allows authorized users or automation rules to invoke the execution role securely. |

---

## Implementation Details

### 1. Execution Role (`test-2automation`)

**Trust Policy**

```json
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
    { "Effect": "Allow", "Action": ["ec2:DescribeInstances"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["ec2:DescribeNetworkInterfaces"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["ec2:ModifyNetworkInterfaceAttribute"], "Resource": "*" }
  ]
}


{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::<ACCOUNT_ID>:role/test-2automation"
    },
    {
      "Effect": "Allow",
      "Action": "ssm:StartAutomationExecution",
      "Resource": "*"
    }
  ]
}

schemaVersion: '0.3'
description: 'Attach a quarantine Security Group to an EC2 instance'
assumeRole: '{{ AutomationAssumeRole }}'
parameters:
  InstanceId:
    type: String
    description: 'EC2 instance ID to quarantine'
  QuarantineSecurityGroupId:
    type: String
    description: 'Security Group ID to attach for isolation'
  AutomationAssumeRole:
    type: String
    description: 'IAM role used by Systems Manager Automation'
mainSteps:
  - name: GetPrimaryEni
    action: 'aws:executeAwsApi'
    inputs:
      Service: ec2
      Api: DescribeInstances
      InstanceIds:
        - '{{ InstanceId }}'
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

ExecutionId: 4d12bfbf-2a35-4f2f-9b20-5b6e812b3b4e
Status: Success
Steps:
  - GetPrimaryEni: Success
  - AttachQuarantineSG: Success
