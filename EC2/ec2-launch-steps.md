## What is Amazon EC2?

Amazon Elastic Compute Cloud (EC2) is an AWS service that provides resizable virtual servers in the cloud. It allows users to run applications without managing physical hardware.

## Why EC2 is Used

- To host websites and applications
- To run servers on demand
- To scale compute resources up or down as needed
- To avoid buying and maintaining physical servers
- 
## EC2 Instance Launch (Beginner)

### Objective
Launch an EC2 instance using AWS Management Console.

### Steps
1. Login to AWS Management Console
2. Open the EC2 service
3. Click **Launch Instance**
4. Choose **Ubuntu Server AMI** (Free Tier eligible)
5. Select **t2.micro** instance type
6. Create a key pair (used for secure SSH access)
7. Configure storage (EBS) with default **8 GB**
8. Launch the instance

### What I Learned
- EBS (Elastic Block Store) is used as storage for EC2 instances
- Key pairs are required to securely connect to EC2 instances

