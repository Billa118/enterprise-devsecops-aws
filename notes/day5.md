# Day 5 - Jenkins Pipeline Setup and GitHub Integration

## Objective

Set up Jenkins on AWS EC2 and create the first Declarative CI pipeline integrated with GitHub.

---

# Environment

- Cloud Provider: AWS
- Operating System: Ubuntu 24.04 LTS
- CI Tool: Jenkins 2.568.2
- Version Control: Git & GitHub
- Repository:
  https://github.com/Billa118/enterprise-devsecops-aws

---

# Tasks Completed

## 1. Verified Jenkins Installation

Verified Jenkins service was running.

```bash
sudo systemctl status jenkins
```

---

## 2. Verified Git Installation

```bash
git --version
```

Git was successfully detected by Jenkins.

---

## 3. Fixed Jenkins Built-in Node

Issue:

- Jenkins Built-in Node was Offline due to temporary disk threshold.

Resolution:

- Brought the Built-in Node online.
- Adjusted disk monitoring threshold.

---

## 4. Created First Pipeline Job

Created a new Pipeline project:

```
aws-devsecops-project
```

---

## 5. Connected Jenkins with GitHub Repository

Repository URL:

```
https://github.com/Billa118/enterprise-devsecops-aws.git
```

Branch:

```
*/main
```

Pipeline Definition:

```
Pipeline script from SCM
```

---

## 6. Created First Jenkinsfile

Created a Declarative Pipeline containing:

- Checkout
- Build
- Test
- Post actions

---

## 7. Fixed Jenkinsfile Location

Initial Error:

```
ERROR: Unable to find Jenkinsfile from git
```

Root Cause:

The Jenkinsfile was inside the `jenkins/` directory.

Resolution:

Moved the Jenkinsfile to the repository root.

Final structure:

```
enterprise-devsecops-aws/
│
├── Jenkinsfile
├── apps/
├── architecture/
├── jenkins/
├── kubernetes/
├── notes/
├── README.md
```

---

## 8. Successful Pipeline Execution

Pipeline stages executed successfully:

- Checkout
- Build
- Test
- Post Actions

Pipeline Status:

```
Finished: SUCCESS
```

---

# Jenkins Console Output Highlights

```
Obtained Jenkinsfile from GitHub

Repository cloned successfully!

Starting build...

Running tests...

Pipeline Finished!

Finished: SUCCESS
```

---

# Key Learnings

- Installed and configured Jenkins on AWS EC2.
- Understood Declarative Jenkins Pipelines.
- Integrated Jenkins with GitHub.
- Learned Pipeline as Code using Jenkinsfile.
- Understood Jenkins workspace and SCM checkout.
- Fixed Jenkinsfile path issues.
- Executed the first successful CI pipeline.

---

# Challenges Faced

### Issue

```
Unable to find Jenkinsfile from git
```

Resolution:

Moved the Jenkinsfile to the repository root.

---

### Issue

Built-in Node Offline

Resolution:

Adjusted disk monitoring threshold and brought the node online.

---

# Project Status

✅ Jenkins Installed

✅ GitHub Connected

✅ Pipeline Created

✅ Jenkinsfile Configured

✅ First CI Pipeline Successful

---

# Next Step (Day 6)

- Configure SonarQube
- Integrate SonarQube with Jenkins
- Configure Jenkins Credentials
- Add Quality Gate
- Integrate Trivy
- Build Docker Image
- Perform Docker Image Scanning