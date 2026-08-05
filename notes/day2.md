# Day 2 - Launching an AWS EC2 Instance

## Objective
Deploy an Ubuntu Linux virtual machine on AWS and connect to it securely using SSH.

## Services Used
- Amazon EC2
- VPC
- Security Groups
- SSH

## Instance Configuration

- Region: ap-south-1 (Mumbai)
- OS: Ubuntu Server 24.04 LTS
- Instance Type: t3.micro
- Storage: 8 GB gp3
- Key Pair: devsecops-key
- Auto Assign Public IP: Enabled

## Security Group Rules

| Port | Protocol | Purpose |
|------|----------|---------|
|22|SSH|Remote Login|
|80|HTTP|Web Server|
|443|HTTPS|Secure Web|
|8080|TCP|Jenkins|

## Commands Used

```bash
chmod 400 devsecops-key.pem

ssh -i ~/Downloads/devsecops-key.pem ubuntu@<PUBLIC-IP>

whoami

hostname

pwd