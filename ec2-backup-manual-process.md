# Manual EC2 Backup Process (Using AMI and EBS Snapshots)

## Overview

A manual EC2 backup is typically performed by creating an **Amazon Machine Image (AMI)**. During AMI creation, AWS automatically creates **EBS snapshots** of all attached EBS volumes.

### Architecture

```text
              EC2 Instance
                   │
        ┌──────────┴──────────┐
        │                     │
   Root EBS Volume      Data EBS Volume
        │                     │
        └──────────┬──────────┘
                   │
            Create AMI
                   │
        AWS Creates EBS Snapshots
                   │
              Amazon Machine Image (AMI)
                   │
           Stored in AWS Account
                   │
      Launch New EC2 Anytime
```

---

# Step 1: Verify the EC2 Instance

Go to:

```
AWS Console

↓

EC2

↓

Instances
```

Select the EC2 instance you want to back up.

Example:

```
Instance Name : Production-WebServer
Instance ID   : i-0123456789abcdef
State         : Running
```

---

# Step 2: (Optional) Prepare the Instance

For application servers:

* Stop application writes if possible.
* Ensure important data is saved.

For database servers:

* Flush or stop the database briefly to ensure a consistent backup, if your application allows.

---

# Step 3: Create an AMI

Navigate to:

```
EC2

↓

Instances

↓

Select Instance

↓

Actions

↓

Image and Templates

↓

Create Image
```

Provide the following details:

**Image Name**

```
Production-WebServer-Backup-2026-07-04
```

**Description**

```
Production server backup before application deployment.
```

Choose whether to reboot the instance:

* **Reboot enabled:** More consistent filesystem state (recommended for critical workloads).
* **No reboot:** Faster backup with minimal interruption, but data being actively written may not be fully consistent.

Click **Create Image**.

---

# Step 4: AWS Creates the Backup

AWS automatically performs the following:

```
EC2 Instance
      │
      ▼
Read Root Volume
      │
      ▼
Create Root Volume Snapshot
      │
      ▼
Read Data Volume
      │
      ▼
Create Data Volume Snapshot
      │
      ▼
Create Amazon Machine Image (AMI)
```

---

# Step 5: Verify the AMI

Navigate to:

```
EC2

↓

AMIs
```

You should see:

```
Name

Production-WebServer-Backup-2026-07-04

Status

Available
```

---

# Step 6: Verify Snapshots

Navigate to:

```
EC2

↓

Elastic Block Store

↓

Snapshots
```

Example:

```
Snapshot ID

snap-0a123456789

Volume

Root Volume

Status

Completed
```

```
Snapshot ID

snap-0b987654321

Volume

Data Volume

Status

Completed
```

---

# Step 7: Restore the Backup

Navigate to:

```
EC2

↓

AMIs

↓

Select the AMI

↓

Launch Instance
```

Configure:

* Instance Type
* VPC
* Subnet
* Security Group
* IAM Role
* Key Pair

Click **Launch Instance**.

AWS creates a new EC2 instance identical to the original at the time the AMI was created.

---

# Alternative Manual Backup (EBS Snapshot Only)

If you only need the disk data:

Navigate to:

```
EC2

↓

Volumes

↓

Select Volume

↓

Actions

↓

Create Snapshot
```

Enter:

```
Snapshot Name

Production-Data-Backup
```

Click **Create Snapshot**.

This creates a backup of only the selected EBS volume.

---

# Restore from an EBS Snapshot

Navigate to:

```
EC2

↓

Snapshots

↓

Select Snapshot

↓

Create Volume
```

Choose:

* Availability Zone
* Volume Type
* Size

Create the volume, then:

```
EC2

↓

Volumes

↓

Attach Volume
```

Attach it to an existing EC2 instance.

---

# AMI vs EBS Snapshot

| Feature                | AMI                                                               | EBS Snapshot |
| ---------------------- | ----------------------------------------------------------------- | ------------ |
| Operating System       | ✅ Yes                                                             | ❌ No         |
| Installed Software     | ✅ Yes                                                             | ❌ No         |
| Application Code       | ✅ Yes                                                             | ❌ No         |
| Instance Configuration | Partially (image only; launch settings are chosen when launching) | ❌ No         |
| Volume Data            | ✅ Yes                                                             | ✅ Yes        |
| Launch New EC2         | ✅ Yes                                                             | ❌ No         |
| Restore Single Volume  | ❌ No                                                              | ✅ Yes        |

---

# Real-Time Example

Suppose your production EC2 is running:

```
EC2

├── Nginx
├── Java
├── Spring Boot Application
└── MySQL Data Volume
```

Before deploying a new application version:

1. Create an AMI.
2. Wait until the AMI status becomes **Available**.
3. Verify the associated snapshots are complete.
4. Deploy the new application.
5. If deployment fails, launch a new EC2 instance from the AMI or restore the required volume from the snapshot.

---

# Interview Answer

**Question:** How do you manually back up an EC2 instance?

**Answer:**

> To manually back up an EC2 instance, I create an Amazon Machine Image (AMI) from the EC2 console. AWS automatically creates EBS snapshots for all attached EBS volumes and registers a new AMI. I verify that the AMI status is **Available** and that the snapshots have completed successfully. If I need to recover the server later, I launch a new EC2 instance from the AMI. If I only need to back up the disk data, I create an EBS snapshot directly from the volume and can later create a new volume from that snapshot and attach it to an EC2 instance.
