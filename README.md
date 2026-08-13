# Enterprise DevSecOps CI/CD Pipeline on AWS

An end-to-end DevSecOps project that demonstrates how to build, secure, containerize, publish, deploy, and verify a Flask application using an automated Jenkins CI/CD pipeline.

The project is implemented on an AWS EC2 Ubuntu server and integrates GitHub, Jenkins, SonarQube, Docker, Trivy, Docker Hub, Kubernetes, Minikube, and Helm.

---

# 🚀 Project Overview

This project demonstrates a complete DevSecOps lifecycle:

```text
SOURCE CODE
     ↓
   GITHUB
     ↓
   JENKINS
     ↓
SONARQUBE
     ↓
DOCKER BUILD
     ↓
TRIVY SECURITY SCAN
     ↓
DOCKER HUB
     ↓
HELM DEPLOYMENT
     ↓
KUBERNETES / MINIKUBE
     ↓
ROLLOUT VERIFICATION
     ↓
APPLICATION TEST
```

The Jenkins pipeline automates the complete workflow so that an application can move from source code to a running Kubernetes deployment with security checks integrated into the CI/CD process.

---

# 🎯 Project Objectives

The main objectives of this project are:

- Build an end-to-end DevSecOps pipeline
- Automate CI/CD using Jenkins
- Integrate GitHub with Jenkins
- Perform static code analysis using SonarQube
- Build a secure Docker image
- Scan Docker images using Trivy
- Block vulnerable images from being published
- Push validated images to Docker Hub
- Deploy applications to Kubernetes
- Use Helm for Kubernetes package management
- Configure Kubernetes resource requests and limits
- Configure liveness and readiness probes
- Verify Kubernetes rollouts automatically
- Validate the deployed application
- Manage secrets through Jenkins credentials
- Document the complete DevSecOps workflow

---

# ☁️ AWS Environment

The project runs on an AWS EC2 instance.

## Infrastructure

```text
AWS
 |
 └── EC2 Instance
      |
      └── Ubuntu Linux
           |
           ├── Jenkins
           ├── Docker
           ├── SonarQube
           ├── Trivy
           ├── Minikube
           ├── Kubernetes
           └── Helm
```

The EC2 instance acts as the main DevSecOps server where the CI/CD tooling and Kubernetes environment are hosted.

---

# 🏗️ Architecture

```text
                         Developer
                             |
                             v
                          GitHub
                             |
                             v
                          Jenkins
                             |
              +--------------+--------------+
              |                             |
              v                             v
         SonarQube                    Docker Build
       Static Analysis                     |
              |                            v
              |                          Trivy
              |                    Security Scanning
              |                            |
              |                            v
              |                       Docker Hub
              |                            |
              +-------------+--------------+
                            |
                            v
                           Helm
                            |
                            v
                    Kubernetes / Minikube
                            |
                    +-------+-------+
                    |               |
                    v               v
               Deployment       NodePort
                    |               |
                    v               v
                  Pod         Flask Application
```

---

# 🛠️ Technology Stack

| Category | Technology |
|---|---|
| Cloud | AWS EC2 |
| Operating System | Ubuntu Linux |
| Source Control | Git |
| Repository | GitHub |
| CI/CD | Jenkins |
| Code Quality | SonarQube |
| Containerization | Docker |
| Container Security | Trivy |
| Container Registry | Docker Hub |
| Orchestration | Kubernetes |
| Kubernetes Environment | Minikube |
| Kubernetes Package Manager | Helm |
| Application | Flask |
| Application Server | Gunicorn |
| Scripting | Bash |
| Configuration | YAML |

---

# 📁 Repository Structure

```text
enterprise-devsecops-aws/
│
├── apps/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── application source
│
├── enterprise-chart/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── templates/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── serviceaccount.yaml
│   │   ├── _helpers.tpl
│   │   └── tests/
│   │
├── notes/
│   ├── day1.md
│   ├── day2.md
│   ├── day3.md
│   ├── day4.md
│   ├── day5.md
│   ├── day6.md
│   ├── day7.md
│   ├── day8.md
│   ├── day9.md
│   ├── day10.md
│   ├── day11.md
│   ├── day12.md
│   ├── day13.md
│   ├── day14.md
│   └── day15.md
│
├── Jenkinsfile
├── .gitignore
└── README.md
```

---

# 🔄 CI/CD Pipeline

The Jenkins pipeline consists of the following major stages:

```text
Checkout
   ↓
SonarQube Analysis
   ↓
Docker Build
   ↓
Trivy Scan
   ↓
Docker Hub Push
   ↓
Helm Deployment
   ↓
Kubernetes Verification
   ↓
Application Verification
```

---

# 1️⃣ Source Code Checkout

Jenkins retrieves the project source code from GitHub.

Repository:

```text
https://github.com/Billa118/enterprise-devsecops-aws
```

Jenkins checks out the latest `main` branch before executing the pipeline.

The pipeline therefore works directly from the Git repository instead of relying on manually copied source files.

---

# 2️⃣ SonarQube Static Code Analysis

The pipeline performs static code analysis using SonarQube.

The application source analyzed by SonarQube is:

```text
apps/
```

The SonarQube project key is:

```text
enterprise-devsecops-aws
```

The pipeline uses Jenkins-managed credentials for authentication.

The SonarQube token is not hardcoded directly into the Jenkinsfile.

Example pipeline configuration:

```text
-Dsonar.projectKey=enterprise-devsecops-aws
-Dsonar.sources=apps
-Dsonar.host.url=http://localhost:9000
-Dsonar.token=$SONAR_TOKEN
-Dsonar.python.version=3.11
```

The final pipeline successfully completed the SonarQube analysis.

SonarQube result:

```text
ANALYSIS SUCCESSFUL
```

---

# 3️⃣ Docker Image Build

The Flask application is packaged into a Docker image.

The Dockerfile uses:

```text
python:3.11-slim-trixie
```

The Docker build uses a multi-stage approach.

The build stage installs the Python dependencies and the runtime stage receives only the required installed packages and application files.

The final image removes unnecessary packaging components such as:

```text
pip
setuptools
wheel
pkg_resources
```

This reduces unnecessary components in the runtime image.

Local image created by Jenkins:

```text
enterprise-devsecops:v1
```

---

# 4️⃣ Trivy Container Security Scan

Before the Docker image is published, Jenkins scans the image using Trivy.

The pipeline executes:

```bash
trivy image \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  --exit-code 1 \
  enterprise-devsecops:v1
```

## Security Gate

The pipeline is configured so that applicable HIGH or CRITICAL vulnerabilities cause the stage to fail.

```text
HIGH       → Pipeline failure
CRITICAL   → Pipeline failure
```

The final validated image reported:

```text
HIGH vulnerabilities:       0
CRITICAL vulnerabilities:  0
```

Trivy final result:

```text
PASS
```

The scan also detected the Debian base image and Python packages inside the container.

---

# 5️⃣ Docker Hub Publishing

Only after successfully passing the Trivy security scan does Jenkins publish the Docker image.

Jenkins authenticates with Docker Hub using Jenkins-managed credentials.

The Docker image repository is:

```text
billa1108/enterprise-devsecops
```

The final validated image was:

```text
billa1108/enterprise-devsecops:31
```

The image was successfully pushed to Docker Hub during the final Jenkins pipeline execution.

---

# 6️⃣ Helm Deployment

Jenkins deploys the application to Kubernetes using Helm.

The Helm chart is located at:

```text
enterprise-chart/
```

The deployment command used by the pipeline is:

```bash
helm upgrade --install enterprise-app ./enterprise-chart \
  --set image.repository=billa1108/enterprise-devsecops \
  --set image.tag=31
```

Helm successfully deployed the application.

Final Helm release:

```text
NAME:        enterprise-app
NAMESPACE:   default
REVISION:    3
STATUS:      deployed
CHART:       enterprise-chart-0.1.0
APP VERSION: 1.16.0
```

---

# 7️⃣ Kubernetes Deployment

The application is deployed to Kubernetes through the Helm chart.

The Kubernetes deployment contains:

```text
Deployment
   |
   └── ReplicaSet
         |
         └── Pod
```

The application container exposes:

```text
Container Port: 5000
```

The Kubernetes service is:

```text
Type: NodePort
Port: 5000
NodePort: 31567
```

The final deployment contained:

```text
1 Replica
1 Available Pod
```

---

# 8️⃣ Kubernetes Resource Management

CPU and memory requests and limits are configured for the application container.

Configuration:

```yaml
resources:
  requests:
    cpu: 50m
    memory: 32Mi

  limits:
    cpu: 250m
    memory: 128Mi
```

## Requests

```text
CPU:    50m
Memory: 32Mi
```

## Limits

```text
CPU:    250m
Memory: 128Mi
```

These settings provide predictable resource allocation and prevent the application container from consuming unlimited cluster resources.

---

# 9️⃣ Liveness Probe

A Kubernetes liveness probe is configured for the application.

Configuration:

```text
HTTP GET /
Port: 5000
```

The liveness probe determines whether the application is still running correctly.

If the container becomes unhealthy, Kubernetes can restart the container.

---

# 🔟 Readiness Probe

A Kubernetes readiness probe is also configured.

Configuration:

```text
HTTP GET /
Port: 5000
```

The readiness probe determines whether the application is ready to receive traffic.

This prevents traffic from being sent to a pod that has not successfully started.

---

# 1️⃣1️⃣ Kubernetes Rollout Verification

After Helm deployment, Jenkins verifies the Kubernetes rollout.

Command:

```bash
kubectl rollout status deployment/enterprise-app-enterprise-chart
```

Final result:

```text
deployment "enterprise-app-enterprise-chart" successfully rolled out
```

Final deployment status:

```text
READY:        1/1
UP-TO-DATE:   1
AVAILABLE:    1
```

Final pod:

```text
READY:       1/1
STATUS:      Running
RESTARTS:    0
```

---

# 1️⃣2️⃣ Kubernetes Service Verification

The deployed application is exposed using a NodePort service.

Service:

```text
enterprise-app-enterprise-chart
```

Service type:

```text
NodePort
```

Service port:

```text
5000
```

NodePort:

```text
31567
```

The service was successfully verified using Kubernetes commands.

---

# 1️⃣3️⃣ Application Verification

The final application was tested through the Kubernetes NodePort.

The Node IP inside Minikube was:

```text
192.168.49.2
```

NodePort:

```text
31567
```

The application was tested using:

```bash
curl "http://${NODEIP}:${NODEPORT}"
```

Final response:

```html
<h1>Enterprise DevSecOps Pipeline</h1>
<p>Flask Application Running Successfully!</p>
```

This confirms that:

- The container is running
- The Kubernetes pod is healthy
- The service is routing traffic
- The Flask application is responding
- The deployment is operational

---

# 🔐 Security Controls

The project integrates security throughout the CI/CD lifecycle.

## Source Code Security

```text
SonarQube static code analysis
Jenkins credential management
No hardcoded SonarQube token
```

## Container Security

```text
Trivy vulnerability scanning
HIGH/CRITICAL vulnerability gate
Multi-stage Docker build
Minimal Python base image
Removal of unnecessary packaging tools
```

## Kubernetes Security and Reliability

```text
CPU resource requests
Memory resource requests
CPU resource limits
Memory resource limits
Liveness probe
Readiness probe
Helm-managed deployment
Automated rollout verification
```

---

# 🔒 Jenkins Credential Management

Sensitive credentials are managed through Jenkins credentials instead of storing secrets directly in the repository.

Credentials used by the pipeline include:

```text
SonarQube Token
Docker Hub Credentials
```

The Jenkins pipeline masks credential values during execution.

Example:

```text
Masking supported pattern matches
```

This prevents sensitive values from being exposed directly in Jenkins console output.

---

# ☸️ Kubernetes Architecture

The Helm deployment follows this structure:

```text
                  Helm Chart
                      |
                      v
                 Deployment
                      |
                 ReplicaSet
                      |
                      v
                     Pod
                      |
             +--------+--------+
             |                 |
             v                 v
       Resource Limits     Health Probes
                           /         \
                          /           \
                    Liveness       Readiness
                      Probe           Probe
                         \             /
                          \           /
                           v         v
                         Flask
                      Application
                          |
                          v
                       Service
                          |
                       NodePort
                          |
                          v
                       Client
```

---

# 📦 Final Helm Release

```text
NAME:        enterprise-app
NAMESPACE:   default
REVISION:    3
STATUS:      deployed
CHART:       enterprise-chart-0.1.0
APP VERSION: 1.16.0
```

---

# 🧪 Final Pipeline Validation

The final Jenkins pipeline completed successfully.

```text
Jenkins Pipeline       : SUCCESS
Source Checkout        : SUCCESS
SonarQube Analysis     : SUCCESS
Docker Build           : SUCCESS
Trivy Security Scan    : PASS
Docker Hub Push        : SUCCESS
Helm Deployment        : SUCCESS
Kubernetes Rollout     : SUCCESS
Application Test       : SUCCESS
```

---

# 📊 Final Deployment State

```text
Kubernetes Node:
    minikube
    STATUS: Ready

Deployment:
    enterprise-app-enterprise-chart
    READY: 1/1
    AVAILABLE: 1

Pod:
    READY: 1/1
    STATUS: Running
    RESTARTS: 0

Service:
    TYPE: NodePort
    PORT: 5000
    NODEPORT: 31567

Helm:
    STATUS: deployed
    REVISION: 3

Docker Image:
    billa1108/enterprise-devsecops:31

Trivy:
    HIGH: 0
    CRITICAL: 0

Application:
    Flask Application Running Successfully!
```

---

# 📝 Project Documentation

The complete project was developed and documented over 15 days.

The daily documentation is available in:

```text
notes/
```

Documentation includes:

```text
day1.md
day2.md
day3.md
day4.md
day5.md
day6.md
day7.md
day8.md
day9.md
day10.md
day11.md
day12.md
day13.md
day14.md
day15.md
```

These notes document the implementation, configuration, troubleshooting, security improvements, CI/CD development, Kubernetes deployment, validation, and final project completion.

---

# 🧠 DevOps Skills Demonstrated

This project demonstrates practical experience with:

## Cloud

```text
AWS EC2
Ubuntu Linux
AWS infrastructure management
```

## Source Control

```text
Git
GitHub
Git branches
Git commits
Git push/pull
Remote repository management
```

## CI/CD

```text
Jenkins
Declarative Jenkins Pipeline
Pipeline stages
Jenkins credentials
Automated deployment
Pipeline verification
```

## Code Quality

```text
SonarQube
Static code analysis
Quality analysis integration
```

## Containers

```text
Docker
Dockerfile
Multi-stage builds
Docker image hardening
Docker Hub
Container image publishing
```

## Container Security

```text
Trivy
Vulnerability scanning
HIGH/CRITICAL security gate
Unfixed vulnerability handling
```

## Kubernetes

```text
Kubernetes
Minikube
Deployment
ReplicaSet
Pod
Service
NodePort
Resource requests
Resource limits
Liveness probes
Readiness probes
Rollout verification
```

## Helm

```text
Helm charts
values.yaml
Deployment templates
Service templates
helm upgrade --install
Helm release management
```

## Linux

```text
Ubuntu
Bash
system administration
process management
Docker administration
Jenkins workspace management
```

---

# 🏆 Project Outcome

The project successfully implements an end-to-end DevSecOps workflow.

The final workflow is:

```text
                  SOURCE
                    |
                    v
                 GitHub
                    |
                    v
                  BUILD
                    |
                    v
                 Jenkins
                    |
                    v
                ANALYZE
                    |
                    v
               SonarQube
                    |
                    v
                 PACKAGE
                    |
                    v
                  Docker
                    |
                    v
                SECURITY
                    |
                    v
                 Trivy
                    |
              +-----+-----+
              |           |
            FAIL         PASS
              |           |
              X           v
                       Docker Hub
                           |
                           v
                        DEPLOY
                           |
                           v
                          Helm
                           |
                           v
                     Kubernetes
                           |
                           v
                        VERIFY
                           |
                           v
                    Flask Application
```

The final pipeline successfully:

```text
✓ Retrieved source code from GitHub
✓ Performed SonarQube analysis
✓ Built the Docker image
✓ Passed the Trivy security scan
✓ Published the validated Docker image
✓ Deployed the application using Helm
✓ Created the Kubernetes deployment
✓ Configured resource requests and limits
✓ Configured liveness and readiness probes
✓ Successfully rolled out the deployment
✓ Verified the Kubernetes service
✓ Successfully tested the Flask application
```

---

# 💼 Resume Relevance

This project demonstrates practical knowledge of an enterprise-style DevSecOps workflow rather than only individual tool usage.

Key resume-relevant areas include:

```text
CI/CD Automation
DevSecOps
Jenkins
AWS EC2
Docker
Trivy
SonarQube
Kubernetes
Minikube
Helm
GitHub
Docker Hub
Linux
Bash
Container Security
Kubernetes Deployment
Infrastructure and Deployment Automation
```

The project can be presented as a hands-on DevSecOps project demonstrating the integration of development, security, containerization, CI/CD, and Kubernetes deployment.

---

# 🔗 Repository

GitHub:

```text
https://github.com/Billa118/enterprise-devsecops-aws
```

---

# 📌 Final Summary

This project demonstrates a complete DevSecOps CI/CD implementation on AWS.

The application moves through the following automated lifecycle:

```text
CODE
  ↓
SOURCE CONTROL
  ↓
CI/CD
  ↓
CODE ANALYSIS
  ↓
CONTAINER BUILD
  ↓
SECURITY SCAN
  ↓
CONTAINER REGISTRY
  ↓
KUBERNETES DEPLOYMENT
  ↓
HEALTH VERIFICATION
  ↓
APPLICATION TEST
```

The final result is a working Flask application that is automatically built, analyzed, security scanned, published, deployed, and verified through a Jenkins-based DevSecOps pipeline.