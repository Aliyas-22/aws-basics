# Amazon S3 Files — Mount an S3 Bucket on an EC2 Instance

## Step 1 — Create the S3 Bucket

Go to:

```
AWS Console → S3 → Buckets → Create bucket
```

Create a private bucket.

Example:

```
s3-file-mounting
```

Keep Block Public Access enabled.

The bucket will be used as the backend storage for the S3 File System.

## Step 2 — Create Folders and Upload Files

Inside the S3 bucket, create the following structure:

```
s3-file-mounting/
│
├── README.md
│
├── documents/
│   ├── project-notes.txt
│   └── deployment-guide.txt
│
├── reports/
│   ├── infrastructure-report.txt
│   └── deployment-status.txt
│
└── logs/
    └── application.log
```

Example `README.md`:

```
This file is stored in Amazon S3 and will be accessed
from the EC2 instance through Amazon S3 Files.
```

Example `project-notes.txt`:

```
S3 Files mounting practical.
EC2 is being used as the client.
```

<img width="1920" height="528" alt="Screenshot (2633)" src="https://github.com/user-attachments/assets/53ca1d65-6df3-4af0-a0c7-cb0214ff8e81" />

## Step 3 — Create the IAM Role

Go to:

```
AWS Console → IAM → Roles → Create role
```

Select:

```
Trusted entity type: AWS service
Use case: EC2
```

This allows an EC2 instance to assume the role.

Example role name:

```
EC2-S3Files-mount-Role
```

### IAM Trust Policy

The trust relationship allows EC2 to assume the role.

Conceptually:

```
EC2
 │
 │ Assume Role
 ▼
EC2-S3Files-mount-Role
```

The trusted service is:

```
ec2.amazonaws.com
```

## Step 4 — Configure IAM Permissions

The IAM role needs permissions that allow the EC2 instance to interact with the S3 File System.

For this lab, the role was configured with the required S3 Files permissions.

The role included:

```
AmazonS3FilesClientFullAccess
```

An additional inline policy was also configured for the required S3 Files access.

### Managed Policy vs Inline Policy

**Managed Policy**

A managed policy is a reusable IAM policy that can be attached to multiple identities.

Example:

```
AmazonS3FilesClientFullAccess
```

Conceptually:

```
Managed Policy
      │
      ├── Role A
      ├── Role B
      └── Role C
```

**Inline Policy**

An inline policy is a policy directly embedded into a specific IAM user, group, or role.

Conceptually:

```
EC2-S3Files-mount-Role
          │
          └── Inline Policy
```

**Simple Difference**

```
Permission        = individual rule
Policy             = collection of permissions
Managed Policy     = reusable policy
Inline Policy      = policy directly attached to one identity
```

<img width="1920" height="484" alt="Screenshot (2611)" src="https://github.com/user-attachments/assets/9ade9ebb-9e13-42b9-af44-53aed10c14d9" />


## Step 5 — Create the EC2 Instance

Go to:

```
AWS Console → EC2 → Instances → Launch Instance
```

Example:

```
Instance Name: EC2-S3Files-mount-instance
```

Use:

```
AMI: Amazon Linux 2023
```

Attach the IAM role created earlier:

```
EC2-S3Files-mount-Role
```

<img width="1920" height="736" alt="Screenshot (2634)" src="https://github.com/user-attachments/assets/68971537-a534-4f5c-86b8-88842478e161" />


## Step 6 — Configure the Security Group

The EC2 instance uses a security group to control network traffic.

For this lab, the required rules were configured for:

| Type | Protocol | Port | Purpose |
|------|----------|------|---------|
| SSH  | TCP      | 22   | Connect to EC2 |
| HTTP | TCP      | 80   | HTTP traffic |
| NFS  | TCP      | 2049 | S3 Files filesystem traffic |

### What is NFS?

NFS = Network File System

NFS is a network filesystem protocol that allows a filesystem to be accessed over a network.

For this lab:

```
EC2
 │
 │ TCP 2049
 │
 ▼
S3 Files Mount Target
```

Port `2049` is the standard NFS port.

<img width="1920" height="661" alt="Screenshot (2630)" src="https://github.com/user-attachments/assets/dfebb806-3146-4cf2-8e8c-71cd7befdef0" />


## Step 7 — Create the S3 File System

Go to:

```
AWS Console → S3 → File systems
```

Click:

```
Create file system
```

Select the S3 bucket:

```
s3://s3-file-mounting
```

The created S3 File System had an ID similar to:

```
fs-086b72fc16b82f13e
```

The S3 File System provides a filesystem interface to the objects stored in the S3 bucket.

<img width="1920" height="723" alt="Screenshot (2613)" src="https://github.com/user-attachments/assets/ecd63383-b817-4d72-94d9-65f3fe0b832a" />


## Step 8 — Mount Targets

After creating the S3 File System, AWS created mount targets in multiple Availability Zones.

Example:

```
VPC
│
├── Availability Zone 1
│      └── Mount Target
│
├── Availability Zone 2
│      └── Mount Target
│
└── Availability Zone 3
       └── Mount Target
```

The mount target acts as the network endpoint through which the EC2 instance connects to the S3 File System.

Architecture:

```
EC2
 │
 │ NFS / TCP 2049
 ▼
Mount Target
 │
 ▼
S3 File System
 │
 ▼
S3 Bucket
```

## Step 9 — Configure the Mount Target Security Group

Initially, the mount target used its default security-group configuration.

The mount target security group was then configured to allow the EC2 instance to communicate with it using NFS.

The required traffic is:

```
EC2 Security Group
        │
        │ TCP 2049
        ▼
Mount Target Security Group
```

The EC2 security group was used for the required NFS communication.

This configuration was important because the initial mount attempt resulted in a timeout.

## Step 10 — Connect to the EC2 Instance

SSH into the EC2 instance.

Example:

```bash
ssh -i my-key.pem ec2-user@<EC2-PUBLIC-IP>
```

Once connected:

```bash
whoami
```

Expected:

```
ec2-user
```

## Step 11 — Install the Required Client

Install the required Amazon EFS utilities package.

Run:

```bash
curl https://amazon-efs-utils.aws.com/efs-utils-installer.sh | sudo sh -s -- --install
```

The installation provides the utilities required to mount the S3 File System.

Verify installation:

```bash
rpm -qa | grep amazon-efs-utils
```

Example output:

```
amazon-efs-utils-3.2.0-1.amzn2023.x86_64
```


## Step 12 — Create the Mount Directory

Create a directory that will be used as the mount point:

```bash
sudo mkdir -p /mnt/s3files
```

Before mounting:

```
EC2
└── /mnt/s3files
```

The directory is initially just a normal empty directory.

## Step 13 — Mount the S3 File System

Use the S3 File System ID.

Example:

```
fs-086b72fc16b82f13e
```

Mount it:

```bash
sudo mount -t s3files fs-086b72fc16b82f13e:/ /mnt/s3files
```

The general syntax is:

```bash
sudo mount -t s3files <file-system-id>:/ <mount-point>
```

## Step 14 — Verify the Mount

Check the mounted filesystem:

```bash
df -h /mnt/s3files
```

Example:

```
Filesystem   Size   Used   Avail   Use%   Mounted on
127.0.0.1:/  8.0E     0    8.0E     0%   /mnt/s3files
```

This confirms that the S3 File System is mounted at `/mnt/s3files`.

## Step 15 — View S3 Files from EC2

Run:

```bash
ls -la /mnt/s3files
```

The S3 bucket contents should now be visible from EC2.

Example:

```
README.md
documents
logs
reports
```

This demonstrates:

```
S3 Bucket
    ↓
S3 Files
    ↓
/mnt/s3files
    ↓
EC2
```

<img width="1289" height="814" alt="Screenshot (2621)" src="https://github.com/user-attachments/assets/54815193-4e8c-4153-b988-569f71091c1d" />


## Step 16 — Read an S3 Object from EC2

Because the S3 bucket is mounted as a filesystem, normal Linux commands can be used.

For example:

```bash
cat /mnt/s3files/README.md
```

The contents of the S3 object are displayed directly from EC2.
<img width="1920" height="822" alt="Screenshot (2624)" src="https://github.com/user-attachments/assets/3d9cbf1d-2410-4c83-9d42-67b57c445434" />


## Step 17 — Create a File from EC2

Create a new file:

```bash
echo "Hello from EC2 through S3 Files!" | sudo tee /mnt/s3files/ec2-test.txt
```

Verify:

```bash
ls -l /mnt/s3files
```

You should see:

```
ec2-test.txt
README.md
documents
logs
reports
```

The new file is stored as an object in the S3 bucket.

## Step 18 — Read the Newly Created File

Run:

```bash
cat /mnt/s3files/ec2-test.txt
```

Expected output:

```
Hello from EC2 through S3 Files!
```

## Step 19 — Verify the File in S3

Open:

```
AWS Console → S3 → s3-file-mounting
```

The following object should now be visible:

```
ec2-test.txt
```

This proves that a file created using a normal Linux filesystem command on EC2 was stored in S3 through S3 Files.
<img width="1307" height="396" alt="Screenshot (2626)" src="https://github.com/user-attachments/assets/07b71661-fc93-4a01-865f-947ce7fa7825" />


## Step 20 — Delete a File

A file can also be deleted using normal Linux commands.

Example:

```bash
sudo rm /mnt/s3files/ec2-test.txt
```

Verify:

```bash
ls -la /mnt/s3files
```

Then refresh the S3 bucket and verify that the corresponding S3 object has been removed.

## 🔄 Complete Read / Write / Delete Flow

```
                    S3 Bucket
                       │
                       │
                 S3 File System
                       │
                       │
                Mount Target
                       │
                  TCP 2049
                       │
                       ▼
                    EC2
                       │
                /mnt/s3files
                       │
          ┌────────────┼────────────┐
          │            │            │
         READ         WRITE       DELETE
          │            │            │
          ▼            ▼            ▼
         S3           S3           S3
       Object        Object       Object
```

## 🧠 Important Concepts Learned

### 1. IAM Role

An IAM role provides temporary permissions to the EC2 instance.

```
EC2 → IAM Role → Permissions
```

### 2. IAM Policy

A policy defines what actions are allowed or denied.

Example:

```
s3files:ClientMount
s3files:ClientWrite
```

### 3. Inline Policy

An inline policy is directly embedded into a specific IAM identity.

```
IAM Role
   │
   └── Inline Policy
```

It is useful when the policy is intended specifically for that one identity.

### 4. Security Group

Security groups control network traffic.

For this lab, NFS traffic uses `TCP 2049`.

### 5. NFS

NFS stands for **Network File System**. It allows filesystem access over a network.

### 6. Mount Point

The mount point is `/mnt/s3files`. It is the location where the S3 File System becomes accessible inside the EC2 operating system.

### 7. S3 File System

The S3 File System provides a filesystem interface to S3 objects.

Instead of:

```bash
aws s3 cp
aws s3 ls
aws s3 rm
```

we can use normal filesystem commands:

```bash
ls
cat
echo
touch
rm
```

## ⚠️ Important Difference: S3 vs EBS vs EFS

| Storage | Type | Access Method |
|---------|------|----------------|
| S3 | Object storage | Objects/API |
| EBS | Block storage | Block device/filesystem |
| EFS | Network file storage | NFS |
| S3 Files | Filesystem interface to S3 | Filesystem operations |

S3 Files does not turn S3 into an EBS disk. The actual objects continue to live in Amazon S3.

## 🧹 Cleanup

When the lab is complete, clean up resources to avoid unnecessary AWS charges.

**Unmount the filesystem**

```bash
sudo umount /mnt/s3files
```

**Delete the S3 File System**

```
S3 → File systems → Select file system → Delete
```

**Remove the IAM role**

If the role was created only for this lab:

```
IAM → Roles → EC2-S3Files-mount-Role
```

Remove/delete it after confirming it is no longer required.

**Terminate EC2**

If the instance is no longer required:

```
EC2 → Instances → Terminate instance
```

**Delete the S3 bucket**

First empty the bucket:

```
S3 → Bucket → Empty
```

Then:

```
Delete bucket
```

## 🎯 Final Result

The lab successfully demonstrated how to expose an Amazon S3 bucket through a filesystem interface on an EC2 instance.

The final access path was:

```
S3 Bucket
    │
    ▼
S3 File System
    │
    ▼
Mount Target
    │
    │ NFS / TCP 2049
    ▼
EC2 Instance
    │
    ▼
/mnt/s3files
```

From EC2, we were able to:

```bash
ls /mnt/s3files
cat /mnt/s3files/README.md
echo "Hello from EC2" | sudo tee /mnt/s3files/ec2-test.txt
sudo rm /mnt/s3files/ec2-test.txt
```

Therefore, the practical demonstrated:

- S3 → EC2 read access
- EC2 → S3 write access
- EC2 → S3 delete access

using a mounted filesystem interface.

## 📚 AWS Documentation

- Amazon S3 Files — Mounting
- Amazon S3 Files — Getting Started
- Amazon S3 Files — Troubleshooting

> **Note:** This README intentionally uses the term **"S3 File System"** rather than saying the bucket was "mounted directly." Technically, the flow is **S3 bucket → S3 File System → mount target → EC2**, which is the more accurate description for an interview or project README.
