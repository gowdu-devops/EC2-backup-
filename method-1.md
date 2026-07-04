# AWS Backup for Automatic EBS Backups

## Overview
AWS Backup is the recommended enterprise service for automating EBS backups.

## Architecture
```text
EBS Volume
    │
    ▼
AWS Backup Plan
    │
    ▼
Backup Rule (Daily 2:00 AM)
    │
    ▼
Backup Vault (Encrypted)
    │
    ▼
Recovery Points
    │
    ▼
Cross-Region / Cross-Account (Optional)
```

## Prerequisites
- EBS volume attached to an EC2 instance
- IAM permissions
- AWS Backup service enabled
- KMS key (optional)

## Step 1: Create a Backup Vault
1. Open **AWS Backup**
2. Backup vaults → Create backup vault
3. Name: `Production-Vault`
4. Select AWS managed KMS or Customer managed KMS
5. Create the vault

## Step 2: Create a Backup Plan
Backup Plans → Create backup plan

Example:
- Plan Name: `Production-Backup-Plan`
- Rule: `Daily-Backup`
- Frequency: Daily
- Start Time: 2:00 AM
- Completion Window: 8 Hours
- Move to cold storage: After 30 days
- Delete backups: After 365 days
- Destination Vault: Production-Vault

## Step 3: Assign Resources
Recommended: Tag-based assignment

Tag:
```text
Key   : Backup
Value : Daily
```

Assign:
- Resource Type: EC2
- Tag: Backup=Daily

## Step 4: IAM Role
Use:
`AWSBackupDefaultServiceRole`

Permissions:
- EC2
- EBS
- Backup
- Restore
- KMS
- CloudWatch

## Step 5: Automatic Backup
Every day at 2 AM:
- Snapshot root volume
- Snapshot data volumes
- Store recovery point in Backup Vault

## Step 6: Monitor
AWS Backup → Backup Jobs

Verify:
- Job Status
- Recovery Point
- Completion Time

## Step 7: Restore
Backup Vault → Recovery Point → Restore

Select:
- VPC
- Subnet
- Security Group
- IAM Role
- Key Pair
- Instance Type

Launch restored EC2.

## Best Practices
- Use tags
- Encrypt backups
- Enable cross-region copy
- Test restores regularly
- Configure CloudWatch alarms and SNS notifications
