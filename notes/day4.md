# 📅 Day 4 - DevSecOps Server Setup & Infrastructure Optimization

## 🎯 Objective

Set up a complete DevSecOps server by installing Infrastructure as Code (IaC), cloud, and Kubernetes tools. Optimized the EC2 instance by increasing storage and adding swap memory to ensure Jenkins and SonarQube run reliably.

---

# 🛠️ Installed Tools

## Terraform

- Installed Terraform using the official HashiCorp repository.
- Verified the installation.

```bash
terraform version
```

**Output**

```
Terraform v1.15.8
```

---

## AWS CLI

- Installed AWS CLI v2.
- Verified the installation.

```bash
aws --version
```

**Output**

```
aws-cli/2.36.17
```

---

## kubectl

- Installed Kubernetes CLI.
- Verified the installation.

```bash
kubectl version --client
```

**Output**

```
Client Version: v1.36.3
```

---

# Infrastructure Optimization

## Issue 1 - Low Disk Space

### Problem

The EC2 instance was created with an **8 GB** root EBS volume.

```
Filesystem : 6.7 GB
Available  : ~1 GB
```

This caused:

- Disk space warnings in Jenkins.
- Unable to create a swap file.
- SonarQube instability.

### Solution

Expanded the root EBS volume from **8 GB** to **20 GB**.

Resized the partition:

```bash
sudo growpart /dev/nvme0n1 1
```

Resized the filesystem:

```bash
sudo resize2fs /dev/nvme0n1p1
```

Verified:

```bash
df -h
```

**Output**

```
Filesystem      Size  Used Avail Use%
/dev/root        19G  6.6G   12G   37%
```

---

## Issue 2 - SonarQube Stopped Repeatedly

### Problem

SonarQube continuously stopped after startup.

Using kernel logs:

```bash
sudo journalctl -k | grep -i oom
```

It was confirmed that the Linux OOM Killer was terminating the Java process due to insufficient memory.

### Solution

Created a **2 GB swap file**.

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Verified:

```bash
free -h
```

**Output**

```
Swap: 2.0Gi
```

---

# Service Verification

## Git

```bash
git --version
```

**Output**

```
git version 2.53.0
```

---

## Java

```bash
java -version
```

**Output**

```
OpenJDK 21
```

---

## Docker

```bash
docker --version
docker ps
```

**Verified**

- Docker Engine installed.
- SonarQube container running.

---

## Jenkins

```bash
sudo systemctl status jenkins --no-pager
```

**Status**

```
Active: active (running)
```

---

## SonarQube

```bash
curl http://localhost:9000/api/system/status
```

**Output**

```json
{
  "status": "UP"
}
```

---

## Terraform

```bash
terraform version
```

**Status**

```
Installed
```

---

## AWS CLI

```bash
aws --version
```

**Status**

```
Installed
```

---

## kubectl

```bash
kubectl version --client
```

**Status**

```
Installed
```

---

# Final Environment

| Component | Status |
|-----------|--------|
| Ubuntu EC2 | ✅ Running |
| Git | ✅ Installed |
| Java 21 | ✅ Installed |
| Docker | ✅ Installed |
| Jenkins | ✅ Running |
| SonarQube | ✅ Running |
| Terraform | ✅ Installed |
| AWS CLI | ✅ Installed |
| kubectl | ✅ Installed |
| Root Volume | ✅ 20 GB |
| Swap Memory | ✅ 2 GB |

---

# Key Learnings

- Installed Terraform, AWS CLI, and kubectl.
- Expanded an AWS EBS volume without recreating the EC2 instance.
- Resized an ext4 filesystem online.
- Diagnosed Linux Out Of Memory (OOM) issues.
- Created persistent swap memory.
- Stabilized SonarQube by resolving memory constraints.
- Verified all DevSecOps tools before starting CI/CD implementation.

---

# Next Step

- Configure Jenkins plugins.
- Integrate Jenkins with SonarQube.
- Configure Jenkins credentials.
- Create the first Declarative Pipeline.
- Build a Docker image using Jenkins.
- Perform SonarQube code analysis.
- Add Trivy vulnerability scanning.
- Deploy applications using Kubernetes.