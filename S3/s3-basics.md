# Amazon S3 Basics

## What is Amazon S3?

Amazon Simple Storage Service (S3) is an object storage service provided by AWS. It is used to store and retrieve any amount of data such as images, videos, files, and backups over the internet.

---

## Why We Use S3

Amazon S3 is used to:
- Store files and objects securely
- Backup important data
- Host static websites
- Store application data such as images and logs

---

## How to Create an S3 Bucket (Using Console)

### Objective
Create an S3 bucket using AWS Management Console.

### Steps
1. Login to AWS Management Console
2. Open the Amazon S3 service
3. Click **Create bucket**
4. Enter a globally unique bucket name
5. Select the AWS region
6. Keep **Block all public access** enabled (recommended for security)
7. Add tags:
   - Key: Name
   - Value: aws
8. Under **Default encryption**:
   - Enable encryption
   - Select **Server-side encryption with Amazon S3 managed keys (SSE-S3)**
9. Review all settings
10. Click **Create bucket**

---
## How to Delete an S3 Bucket

1. Select the S3 bucket
2. Click **Delete**
3. Enter the bucket name to confirm deletion
4. Click **Delete bucket**


## What I Learned
- S3 stores data as objects, not blocks
- Bucket names must be globally unique
- S3 is highly durable and scalable
