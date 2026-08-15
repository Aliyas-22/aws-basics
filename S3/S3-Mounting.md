# Mounting S3 Bucket to EC2

In this lab, I mounted an Amazon S3 bucket to an EC2 instance using Mountpoint for Amazon S3 and then configured Nginx to serve the S3 file through the EC2 public IP.

## S3 Bucket Setup

Created a private S3 bucket:

```
learning-s3-mount
```

Uploaded:

```
hello.txt
```

## IAM Role Setup

Created an IAM Role for EC2:

```
EC2-S3-Mount-Role
```

Attached:

```
AmazonS3ReadOnlyAccess
```

This allows the EC2 instance to access S3 without storing AWS access keys on the server.

## EC2 Launch

Launched an Amazon Linux 2023 EC2 instance and attached the IAM role:

```
EC2-S3-Mount-Role
```

Connected to the instance via SSH / EC2 Instance Connect.

## IAM Role Verification

```bash
aws sts get-caller-identity
```

<img width="1920" height="842" alt="Screenshot (2600)" src="https://github.com/user-attachments/assets/ff3e514c-0936-4629-adcb-af5ffdae1c45" />


This verifies that the EC2 instance is using the attached IAM role.

## S3 Access Verification

```bash
aws s3 ls
```

This lists the S3 buckets accessible by the EC2 instance.

```bash
aws s3 ls s3://learning-s3-mount/
```

This verifies that EC2 can access the bucket and shows:

```
hello.txt
```

## Mountpoint Installation

```bash
sudo dnf install mount-s3 -y
```

This installs Mountpoint for Amazon S3, which allows an S3 bucket to be accessed through a filesystem-like path.

Check the installation:

```bash
mount-s3 --version
```

## Mount Directory Creation

```bash
sudo mkdir -p /mnt/s3
```

This creates the directory where the S3 bucket will be mounted.

## S3 Bucket Mounting

```bash
sudo mount-s3 learning-s3-mount /mnt/s3
```

Expected:

```
bucket learning-s3-mount is mounted at /mnt/s3
```

Now the S3 bucket is available through `/mnt/s3`.

## Mount Verification

```bash
mount | grep s3
```

This confirms that the S3 bucket is mounted.

Example:

```
mountpoint-s3 on /mnt/s3 type fuse (...)
```

## S3 Files Check from EC2

```bash
sudo ls -lah /mnt/s3
```

Output showed:

```
hello.txt
```

Read the file:

```bash
sudo cat /mnt/s3/hello.txt
```

This confirms that the EC2 instance can read the file stored in S3.

<img width="1920" height="801" alt="Screenshot (2602)" src="https://github.com/user-attachments/assets/8c1923af-64f1-4e94-9bfe-dc5de0818bec" />


## Nginx Access to the Mount

Initially, access to `/mnt/s3` was restricted.

Unmounted the bucket:

```bash
sudo umount /mnt/s3
```

Mounted it again using:

```bash
sudo mount-s3 learning-s3-mount /mnt/s3 --allow-other
```

The `--allow-other` option allows other users and processes, such as Nginx, to access the mounted S3 filesystem.

Verified:

```bash
sudo ls -lah /mnt/s3
```

## Nginx Installation

```bash
sudo dnf install nginx -y
```

Start Nginx:

```bash
sudo systemctl start nginx
```

Enable Nginx at boot:

```bash
sudo systemctl enable nginx
```

Check status:

```bash
sudo systemctl status nginx
```

## Nginx Configuration

Created:

```
/etc/nginx/conf.d/s3.conf
```

Command:

```bash
sudo nano /etc/nginx/conf.d/s3.conf
```

Configuration:

```nginx
server {
    listen 80;
    server_name _;

    root /mnt/s3;

    location / {
        autoindex on;
    }
}
```

This tells Nginx to serve files from `/mnt/s3`.

## Nginx conf.d Enablement

Opened:

```bash
sudo nano /etc/nginx/nginx.conf
```

Made sure this line was active:

```
include /etc/nginx/conf.d/*.conf;
```

It must **not** be commented:

```
# include /etc/nginx/conf.d/*.conf;
```

This allows Nginx to load `/etc/nginx/conf.d/s3.conf`.

## Default Server Block Removal

The default Nginx server block was using `/usr/share/nginx/html` and `server_name _;`.

This conflicted with our S3 configuration.

The default server block was commented out so that the active server configuration points to `/mnt/s3`.

## Nginx Configuration Test

```bash
sudo nginx -t
```

Expected:

```
syntax is ok
test is successful
```

## Nginx Reload

```bash
sudo systemctl reload nginx
```

Verify that Nginx is listening on port 80:

```bash
sudo ss -lntp | grep :80
```

## Security Group HTTP Rule

In the EC2 Security Group, added an inbound rule:

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| HTTP | TCP      | 80   | My IP  |

This allows HTTP traffic to reach Nginx on the EC2 instance.

## Local Test from EC2

```bash
curl http://localhost/hello.txt
```

This request goes to:

```
EC2
 ↓
Nginx
 ↓
/mnt/s3
 ↓
S3
 ↓
hello.txt
```

The contents of `hello.txt` were returned successfully.

## Access via EC2 Public IP

Finally, accessed the S3 file from a browser using the EC2 public IP:

```
http://<EC2-PUBLIC-IP>/hello.txt
```

Example:

```
http://65.1.95.186/hello.txt
```

<img width="1920" height="1080" alt="Screenshot (2603)" src="https://github.com/user-attachments/assets/4e07f8a7-57a5-40a3-b025-564dda600491" />
<img width="1920" height="1025" alt="Screenshot (2604)" src="https://github.com/user-attachments/assets/db769f54-da8a-47e3-aec0-5b96e3bccea2" />


The request flow is:

```
Browser
   ↓
EC2 Public IP
   ↓
Nginx :80
   ↓
/mnt/s3
   ↓
Mountpoint for Amazon S3
   ↓
S3 Bucket
   ↓
hello.txt
```

The S3 bucket remained private. The file was served through Nginx running on the EC2 instance.

## S3 Unmounting

After completing the lab, stopped Nginx:

```bash
sudo systemctl stop nginx
```

Unmounted S3:

```bash
sudo umount /mnt/s3
```

Verified:

```bash
mount | grep s3
```

No output confirms that the S3 bucket is unmounted.

## Difference: S3 Bucket Mount vs S3 Files Mount

This lab mounted a normal S3 bucket, not an S3 Files (EFS-like) filesystem. The table below shows the difference:

| Our Lab | AWS S3 Files |
|---|---|
| Normal S3 bucket | S3 Files file system |
| `learning-s3-mount` | File System ID |
| `mount-s3` | `mount -t s3files` |
| `/mnt/s3` | `/mnt/s3files` |
| Directly mounts an S3 bucket | Mounts an S3 Files filesystem |
| No File System ID | Requires File System ID |
| No mount target | Uses S3 Files mount target |
