# Hosting a Static Website Using Amazon S3

## What is Static Website Hosting in S3?

Amazon S3 allows hosting static websites that contain HTML, CSS, and JavaScript files. It does not support server-side scripting, but it is ideal for simple websites, portfolios, and documentation pages.

---

## Why Use S3 for Static Website Hosting?

Amazon S3 is used for static website hosting because it is:
- Low cost
- Highly available
- Scalable
- Easy to configure
- Serverless (no EC2 required)

---

## Objective
Host a static website using an S3 bucket.

---

## Steps to Create S3 Bucket for Static Website

1. Login to AWS Management Console
2. Open the **S3** service
3. Click **Create bucket**
4. Enter a globally unique bucket name
5. Select AWS region
6. Keep **Block all public access** unchecked
7. Acknowledge public access warning
8. Click **Create bucket**

---

## Enable Static Website Hosting

1. Open the created bucket
2. Go to the **Properties** tab
3. Scroll down to **Static website hosting**
4. Click **Edit**
5. Enable **Static website hosting**
6. Hosting type: **Host a static website**
7. Index document: `index.html`
8. Error document: `error.html`
9. Save changes

After saving, a **Static website URL** will be generated.

---

## Set Bucket Policy (Public Access)

1. Go to **Permissions** tab
2. Open **Bucket policy**
3. Click **Edit**
4. Paste the policy below (replace `YOUR_BUCKET_NAME`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}


---

## STEP 3 — Commit

**Commit message:**
