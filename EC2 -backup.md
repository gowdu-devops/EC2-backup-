# AWS EC2 Backup – Complete Production Setup

## Overview

In production, backing up an EC2 instance is not limited to taking a manual AMI. Organizations use **AWS Backup** to automate backups, enforce retention policies, enable disaster recovery, and restore resources when needed.

---

# Architecture

```
                     Production AWS Account
                              │
                         EC2 Instance
                              │
              ┌───────────────┴───────────────┐
              │                               │
        Root EBS Volume                 Data EBS Volume
              │                               │
              └───────────────┬───────────────┘
                              │
                         AWS Backup
                              │
                        Backup Plan
                              │
                     Daily Backup Schedule
                              │
                        Backup Vault
                              │
                  Recovery Points Created
                              │
        ┌─────────────────────┴────────────────────┐
        │                                          │
 Cross Region Copy                         Cross Account Copy
        │                                          │
 Disaster Recovery                    Secondary AWS Account
```

---

# Prerequisites

- AWS Account
- EC2 Instance
- EBS Volumes
- IAM Permissions
- AWS Backup Service
- KMS Key (Optional for encryption)

---

# Step 1: Launch EC2 Instance

Suppose you have:

- Amazon Linux 2023
- Nginx
- Java
- Tomcat
- Spring Boot Application

Volumes

- Root Volume : 30 GB
- Data Volume : 100 GB

Instance Name

```
Production-WebServer
```

---

# Step 2: Tag EC2 Instance

Go to

EC2

↓

Tags

Add

```
Key

Backup
```

```
Value

Daily
```

Using tags allows AWS Backup to automatically protect new instances.

---

# Step 3: Create Backup Vault

Navigate

```
AWS Backup

↓

Backup Vaults

↓

Create Backup Vault
```

Configuration

```
Vault Name

Production-Backup-Vault
```

Encryption

```
AWS Managed KMS
```

or

```
Customer Managed KMS
```

Result

```
Production-Backup-Vault

│

├── Recovery Point 1

├── Recovery Point 2

├── Recovery Point 3
```

---

# Step 4: Create Backup Plan

Navigate

```
AWS Backup

↓

Backup Plans

↓

Create Backup Plan
```

Plan Name

```
Production-Daily-Backup
```

Rule Name

```
Daily-Backup
```

Backup Frequency

```
Daily
```

Backup Time

```
2:00 AM
```

Completion Window

```
8 Hours
```

Lifecycle

```
Move to Cold Storage

After 30 Days
```

Delete Backup

```
After 365 Days
```

Destination

```
Production-Backup-Vault
```

---

# Step 5: Assign Resources

Choose

```
Assign Resources
```

Resource Type

```
EC2
```

Assignment Method

```
Tags
```

Tag

```
Backup = Daily
```

AWS Backup now automatically backs up every EC2 with this tag.

---

# Step 6: IAM Role

AWS Backup uses

```
AWSBackupDefaultServiceRole
```

Permissions

- EC2
- EBS
- Backup
- Restore
- KMS
- CloudWatch
- IAM

---

# Step 7: Backup Execution

Every day at 2 AM

AWS Backup performs

```
EC2

↓

Root Volume Snapshot

↓

Data Volume Snapshot

↓

Metadata Collection

↓

Recovery Point Creation

↓

Backup Vault
```

No manual action required.

---

# Step 8: Monitor Backup Jobs

Navigate

```
AWS Backup

↓

Backup Jobs
```

Example

```
Job ID

Status

Started Time

Completed Time

Recovery Point
```

Status

```
Completed
```

or

```
Failed
```

---

# Step 9: Recovery Points

Navigate

```
Backup Vault

↓

Recovery Points
```

Example

```
Recovery Point

July 1

July 2

July 3

July 4
```

Each recovery point can restore the EC2 instance.

---

# Step 10: Restore EC2

Suppose someone accidentally terminates the instance.

Navigate

```
AWS Backup

↓

Backup Vault

↓

Recovery Point

↓

Restore
```

Select

```
Restore As

EC2 Instance
```

Provide

```
VPC

Subnet

Security Group

IAM Role

Key Pair

Instance Type
```

Click

```
Restore Backup
```

AWS creates a new EC2 instance from the backup.

---

# Cross Region Disaster Recovery

```
Mumbai Region

↓

Daily Backup

↓

Copy

↓

Singapore Region
```

If Mumbai becomes unavailable, restore the EC2 in Singapore.

---

# Cross Account Backup

```
Production Account

111111111111

↓

Copy Backup

↓

DR Account

222222222222
```

This protects against accidental deletion or account compromise.

---

# Monitoring

Monitor backup jobs using

- AWS Backup Jobs
- CloudWatch Metrics
- CloudWatch Alarms
- EventBridge
- SNS Email Notifications

Example

```
Backup Failed

↓

CloudWatch Alarm

↓

SNS

↓

Email to DevOps Team
```

---

# Backup Flow

```
EC2 Instance

↓

Tag

↓

AWS Backup Plan

↓

Backup Rule

↓

Backup Vault

↓

Recovery Point

↓

Cross Region Copy

↓

Cross Account Copy

↓

Restore When Needed
```

---

# Recovery Flow

```
EC2 Deleted

↓

AWS Backup

↓

Recovery Point

↓

Restore

↓

New EC2 Instance Created

↓

Application Available
```

---

# Production Best Practices

- Use tag-based backup assignments.
- Encrypt backups using AWS KMS.
- Enable cross-region copy.
- Enable cross-account copy.
- Test restores regularly.
- Configure CloudWatch alarms.
- Configure SNS notifications.
- Apply lifecycle policies.
- Use least-privilege IAM roles.
- Retain backups according to compliance requirements.

---

# Backup Methods Comparison

| Method | Backup Type | Best Use Case |
|---------|-------------|---------------|
| AMI | Entire EC2 | Full server clone or migration |
| EBS Snapshot | Volume Only | Data recovery |
| AWS Backup | Automated Backup | Production |
| S3 Backup | Files Only | Logs, Documents |
| Database Dump | Database | MySQL/PostgreSQL backup |

---

# Interview Answer

**Question:** How do you take EC2 backups in production?

**Answer:**

> In production, we use AWS Backup instead of manual backups. First, we create a Backup Vault and a Backup Plan with scheduled backup rules. We assign EC2 instances using tags such as `Backup=Daily`, allowing new instances to be included automatically. AWS Backup creates recovery points by taking EBS snapshots of the instance volumes. We configure lifecycle policies, encrypt backups with KMS, and enable cross-region and cross-account copies for disaster recovery. Backup jobs are monitored using AWS Backup, CloudWatch, EventBridge, and SNS. If an instance is lost, we restore it from the latest recovery point by selecting the VPC, subnet, security groups, IAM role, and instance type.
