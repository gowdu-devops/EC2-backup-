# EventBridge + Lambda Automatic EBS Backup

## Overview
Use EventBridge and Lambda for custom backup workflows.

## Architecture
```text
Amazon EventBridge
        │
        ▼
Scheduled Rule
        │
        ▼
AWS Lambda
        │
        ▼
CreateSnapshot API
        │
        ▼
EBS Snapshot
```

## Prerequisites
- Lambda execution role
- EventBridge schedule
- EC2/EBS permissions

## Step 1: Create IAM Role
Allow:
- ec2:CreateSnapshot
- ec2:DescribeVolumes
- ec2:CreateTags

## Step 2: Create Lambda Function
Language: Python (or another supported runtime)

Responsibilities:
- Find target EBS volumes
- Create snapshots
- Add tags
- Log results

## Step 3: Create EventBridge Rule
Schedule:
- Daily
- 2:00 AM

Target:
- Lambda function

## Step 4: Automatic Flow
At the scheduled time:
1. EventBridge invokes Lambda.
2. Lambda calls CreateSnapshot API.
3. Snapshot is created.
4. Logs are written to CloudWatch.

## Step 5: Restore
Snapshots → Create Volume → Attach Volume to EC2.

## Optional Enhancements
- SNS notifications
- Cross-region copy
- Snapshot naming convention
- Failure alerts
- Retention cleanup logic

## Best Practices
- Use least-privilege IAM
- Monitor CloudWatch logs
- Test restore process regularly
