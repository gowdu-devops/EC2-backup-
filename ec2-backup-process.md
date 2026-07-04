Production Architecture
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
Step 1: Create a Backup Vault

A Backup Vault stores all recovery points (backups).

Open AWS Backup.
Go to Backup vaults.
Click Create backup vault.
Enter:
Vault Name: Production-Vault
Encryption: AWS managed KMS (or Customer Managed KMS)
Create the vault.

Result:

Backup Vault
    │
    ├── EC2 Backups
    ├── EBS Snapshots
    ├── RDS Backups
    └── EFS Backups
Step 2: Create a Backup Plan

Go to:

AWS Backup
    ↓
Backup Plans
    ↓
Create Backup Plan

Example:

Plan Name

Production-Backup-Plan

Rule:

Backup Rule Name

Daily-Backup

Frequency

Daily

Backup Window

2 AM

Completion Window

8 Hours

Lifecycle

Move to Cold Storage
After 30 Days

Delete After
365 Days

Destination

Production Vault
Step 3: Assign Resources

AWS Backup needs to know which EC2 instances to protect.

You can assign:

By Resource ID
i-0123456789abcdef

or

By Tag (Recommended)

Tag EC2

Backup = Daily

AWS Backup

Assign Resources

Resource Type

EC2

Tag

Backup = Daily

Now every EC2 with that tag is automatically included.

This avoids editing the backup plan whenever new servers are created.

Step 4: IAM Role

AWS Backup requires permissions.

Create or use:

AWSBackupDefaultServiceRole

Permissions include:

EC2
EBS
RDS
EFS
Backup
Restore
KMS
CloudWatch
Step 5: Backup Starts Automatically

At 2 AM every day:

EC2
 │
 ├── Root Volume Snapshot
 ├── Data Volume Snapshot
 └── Metadata

↓

Backup Vault

No manual action is required.

Step 6: Monitor Backup Jobs
AWS Backup

↓

Backup Jobs

Example

Job ID

Status

Started

Completed

Recovery Point

Verify all jobs finish successfully.

Step 7: Recovery Points

Every successful backup creates a recovery point.

Backup Vault

↓

Recovery Points

↓

Today's Backup

↓

Yesterday's Backup

↓

Last Week's Backup

These are what you'll restore from.

Step 8: Restore EC2

Suppose an instance is accidentally deleted.

Go to:

Backup Vault

↓

Recovery Point

↓

Restore

Choose:

Restore as

EC2 Instance

Provide:

VPC

Subnet

Security Group

IAM Role

Key Pair

Instance Type

Click Restore.

AWS launches a new EC2 instance using the backup.

Cross-Region Backup (Disaster Recovery)
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

If the Mumbai Region has an outage, you can restore from Singapore.

Cross-Account Backup

Production Account

111111111111

↓

Copy Backup

↓

Disaster Recovery Account

222222222222

This protects against accidental deletion or compromise of the production account.

Monitoring

Use:

AWS Backup Jobs to check backup and restore status.
Amazon CloudWatch for metrics and alarms.
Amazon EventBridge to notify on backup failures or successes.
Amazon SNS to send email notifications to the operations team.
Production Best Practices
Use tag-based assignments (for example, Backup=Daily) instead of selecting instances manually.
Encrypt backup vaults with AWS KMS.
Enable cross-region backup for disaster recovery.
Consider cross-account copies for additional protection.
Test restore procedures regularly to verify backups are usable.
Apply different retention policies based on business requirements.
Monitor backup jobs and configure alerts for failures.
Use least-privilege IAM roles for AWS Backup.
Real-Time Example

Your production environment has:

EC2
├── Web Server
├── API Server
├── Jenkins Server
└── Bastion Host

Each instance is tagged:

Backup = Daily

Backup plan:

Every Day

2:00 AM

Retention:

365 Days

Storage:

Production Backup Vault

Cross-region copy:

Mumbai
↓

Singapore

If the API server is accidentally terminated:

Open AWS Backup.
Select the latest recovery point.
Click Restore.
Choose the VPC, subnet, security group, IAM role, and instance type.
Launch the restored EC2 instance.

This is a common enterprise setup because it provides automated backups, centralized management, encryption, retention policies, disaster recovery options, and straightforward restoration.
