# AWS EC2 Backup - Complete Production Setup

## Production Architecture

```text
                    Production AWS Account
                           │
                    EC2 Instance
                           │
          ┌────────────────┴────────────────┐
          │                                 │
     Root EBS Volume                  Data EBS Volume
          │                                 │
          └──────────────┬──────────────────┘
                         │
                    AWS Backup
                         │
                  Backup Plan
                         │
         Daily @ 2 AM (Snapshot Backup)
                         │
                 Backup Vault (Encrypted)
                         │
       Retention: 30 Days / 90 Days / 1 Year
                         │
          Cross-Region Copy (Optional)
                         │
          Cross-Account Copy (Optional)
                         │
                   Restore When Needed
```

---

# Step 1: Create a Backup Vault

A **Backup Vault** stores all recovery points (backups).

### Steps

1. Open **AWS Backup**.
2. Go to **Backup Vaults**.
3. Click **Create Backup Vault**.
4. Enter:

   * **Vault Name:** `Production-Vault`
   * **Encryption:** AWS Managed KMS (or Customer Managed KMS)
5. Click **Create Vault**.

### Result

```text
Backup Vault
    │
    ├── EC2 Backups
    ├── EBS Snapshots
    ├── RDS Backups
    └── EFS Backups
```

---

# Step 2: Create a Backup Plan

Navigate to:

```text
AWS Backup
    ↓
Backup Plans
    ↓
Create Backup Plan
```

### Example Configuration

**Plan Name**

```text
Production-Backup-Plan
```

**Backup Rule Name**

```text
Daily-Backup
```

**Frequency**

```text
Daily
```

**Backup Window**

```text
2:00 AM
```

**Completion Window**

```text
8 Hours
```

**Lifecycle**

```text
Move to Cold Storage
After 30 Days
```

```text
Delete After
365 Days
```

**Destination**

```text
Production Vault
```

---

# Step 3: Assign Resources

AWS Backup needs to know **which EC2 instances** should be protected.

You can assign resources using either:

## Option 1: Resource ID

```text
i-0123456789abcdef
```

## Option 2: Tags (Recommended)

### Tag the EC2 Instance

```text
Key   : Backup
Value : Daily
```

### Assign Resources

```text
AWS Backup

↓

Assign Resources

↓

Resource Type

EC2

↓

Tag

Backup = Daily
```

Now every EC2 instance with the tag **Backup=Daily** will automatically be included in the backup plan.

**Benefit:** No need to modify the backup plan whenever new servers are created.

---

# Step 4: IAM Role

AWS Backup requires permissions.

Create or use the default service role:

```text
AWSBackupDefaultServiceRole
```

### Required Permissions

* EC2
* EBS
* RDS
* EFS
* Backup
* Restore
* AWS KMS
* CloudWatch

---

# Step 5: Backup Starts Automatically

At **2:00 AM** every day:

```text
EC2
 │
 ├── Root Volume Snapshot
 ├── Data Volume Snapshot
 └── Metadata
          │
          ▼
     Backup Vault
```

No manual action is required.

---

# Step 6: Monitor Backup Jobs

Navigate to:

```text
AWS Backup

↓

Backup Jobs
```

### Example

```text
Job ID

Status

Started Time

Completed Time

Recovery Point
```

Verify that all backup jobs complete successfully.

---

# Step 7: Recovery Points

Every successful backup creates a **Recovery Point**.

```text
Backup Vault

↓

Recovery Points

↓

Today's Backup

↓

Yesterday's Backup

↓

Last Week's Backup
```

Recovery Points are used to restore EC2 instances.

---

# Step 8: Restore EC2

Suppose an EC2 instance is accidentally deleted.

Navigate to:

```text
Backup Vault

↓

Recovery Point

↓

Restore
```

Choose:

```text
Restore As

EC2 Instance
```

Provide:

* VPC
* Subnet
* Security Group
* IAM Role
* Key Pair
* Instance Type

Click **Restore**.

AWS launches a new EC2 instance using the selected recovery point.

---

# Cross-Region Backup (Disaster Recovery)

```text
Mumbai
     │
     ▼
Daily Backup
     │
     ▼
Copy
     │
     ▼
Singapore
```

If the Mumbai Region experiences an outage, the backup can be restored in Singapore.

---

# Cross-Account Backup

```text
Production Account

111111111111

↓

Copy Backup

↓

Disaster Recovery Account

222222222222
```

This protects backups against accidental deletion or compromise of the production AWS account.

---

# Monitoring

Use the following AWS services for monitoring:

* AWS Backup Jobs
* Amazon CloudWatch
* Amazon EventBridge
* Amazon SNS

### Purpose

* Check backup status
* Check restore status
* Configure CloudWatch alarms
* Receive email notifications on backup failures or successes

---

# Production Best Practices

* Use **tag-based resource assignments** (e.g., `Backup=Daily`).
* Encrypt Backup Vaults using **AWS KMS**.
* Enable **Cross-Region Backup** for disaster recovery.
* Enable **Cross-Account Backup** for additional protection.
* Regularly test restore procedures.
* Configure lifecycle policies based on business requirements.
* Monitor backup jobs and configure alerts.
* Follow the principle of least privilege for IAM roles.

---

# Real-Time Production Example

Production environment:

```text
EC2
├── Web Server
├── API Server
├── Jenkins Server
└── Bastion Host
```

Each instance is tagged:

```text
Backup = Daily
```

Backup Schedule:

```text
Every Day

2:00 AM
```

Retention:

```text
365 Days
```

Storage:

```text
Production Backup Vault
```

Cross-Region Backup:

```text
Mumbai

↓

Singapore
```

### Disaster Recovery Scenario

Suppose the **API Server** is accidentally terminated.

Restore Steps:

1. Open AWS Backup.
2. Select the latest Recovery Point.
3. Click **Restore**.
4. Select:

   * VPC
   * Subnet
   * Security Group
   * IAM Role
   * Key Pair
   * Instance Type
5. Click **Launch**.

AWS creates a new EC2 instance from the backup.

---

# Summary

This production backup architecture provides:

* Automated backups
* Centralized backup management
* Encryption using AWS KMS
* Backup retention policies
* Cross-Region disaster recovery
* Cross-Account backup protection
* Easy EC2 restoration
* CloudWatch monitoring
* EventBridge integration
* SNS email notifications

This is the standard backup strategy used in enterprise AWS production environments.
