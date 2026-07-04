# Amazon Data Lifecycle Manager (DLM)

## Overview
Amazon DLM automates EBS snapshot creation and deletion.

## Architecture
```text
EBS Volume
    │
    ▼
Tag (Backup=Daily)
    │
    ▼
Lifecycle Policy
    │
    ▼
Automatic Snapshot
    │
    ▼
Retention Policy
```

## Prerequisites
- EBS volumes
- IAM permissions
- Tags on volumes

## Step 1: Tag Volumes
```text
Key   : Backup
Value : Daily
```

## Step 2: Create Lifecycle Policy
EC2 → Lifecycle Manager → Create lifecycle policy

Configuration:
- Policy Type: EBS Snapshot Policy
- Target: Volumes
- Target Tags: Backup=Daily
- Schedule: Daily
- Time: 1:00 AM
- Retention: Keep last 7 snapshots

## Step 3: Save Policy
Enable the policy.

## Step 4: Automatic Operation
DLM automatically:
- Creates snapshots
- Deletes old snapshots based on retention

## Step 5: Restore
Snapshots → Select Snapshot → Create Volume → Attach Volume to EC2

## Best Practices
- Use tags consistently
- Choose appropriate retention
- Monitor snapshot creation
