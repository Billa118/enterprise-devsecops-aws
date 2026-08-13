# Day 15 — Final DevSecOps Pipeline Validation & Project Completion

## Objective

Complete the final validation of the Enterprise DevSecOps AWS project by verifying the complete CI/CD pipeline, Docker image security, Docker Hub publishing, Helm deployment, Kubernetes rollout, and application availability.

---

## 1. Final Jenkins Pipeline

Jenkins successfully executed the complete CI/CD pipeline from Git checkout to Kubernetes deployment.

### Pipeline Flow

GitHub
↓
Jenkins
↓
SonarQube Analysis
↓
Docker Image Build
↓
Trivy Security Scan
↓
Docker Hub Push
↓
Helm Deployment
↓
Kubernetes Rollout
↓
Application Verification

---

## 2. Git Checkout

Jenkins successfully checked out the latest repository commit.

Repository:

https://github.com/Billa118/enterprise-devsecops-aws.git

The pipeline successfully checked out the latest `main` branch revision.

---

## 3. SonarQube Analysis

SonarQube analysis completed successfully.

Command used by Jenkins:

sonar-scanner \
-Dsonar.projectKey=enterprise-devsecops-aws \
-Dsonar.sources=apps \
-Dsonar.host.url=http://localhost:9000 \
-Dsonar.token=$SONAR_TOKEN \
-Dsonar.python.version=3.11

Result:

ANALYSIS SUCCESSFUL

SonarQube project:

enterprise-devsecops-aws

SonarQube successfully analyzed the application source code under:

apps/

The authentication token was supplied through Jenkins Credentials rather than being hardcoded.

---

## 4. Docker Image Build

Jenkins built the application Docker image successfully.

Image:

enterprise-devsecops:v1

Base image:

python:3.11-slim-trixie

The Dockerfile uses a multi-stage build.

The image contains:

- Python 3.11
- Flask
- Gunicorn
- Application source code
- Production dependencies

The Docker image was successfully created without build errors.

---

## 5. Trivy Security Scan

Trivy scanned the generated Docker image for security vulnerabilities.

Command:

trivy image --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 enterprise-devsecops:v1

Scan results:

- Debian vulnerabilities: 0
- Python package vulnerabilities: 0
- HIGH vulnerabilities: 0
- CRITICAL vulnerabilities: 0

Result:

PASS

The pipeline is configured to fail when HIGH or CRITICAL vulnerabilities are detected.

This provides a security gate before publishing the Docker image.

---

## 6. Docker Hub Push

After the security scan passed, Jenkins authenticated with Docker Hub using Jenkins credentials.

Docker image published:

billa1108/enterprise-devsecops:31

Docker image digest:

sha256:05c8e3da424a7e2c131883439107d52277f8872d0bab5afd12ec17ecbc59e16e

The image was successfully pushed to Docker Hub.

After pushing, Jenkins logged out from Docker Hub.

---

## 7. Helm Deployment

Jenkins deployed the application to Kubernetes using Helm.

Command:

helm upgrade --install enterprise-app ./enterprise-chart \
--set image.repository=billa1108/enterprise-devsecops \
--set image.tag=31

Deployment result:

Release "enterprise-app" has been upgraded.

Helm release:

enterprise-app

Namespace:

default

Helm revision:

3

Status:

deployed

Chart:

enterprise-chart-0.1.0

---

## 8. Kubernetes Deployment

The Kubernetes deployment successfully rolled out the new Docker image.

Deployment:

enterprise-app-enterprise-chart

Replicas:

1 desired
1 updated
1 available

Current pod:

enterprise-app-enterprise-chart-d59f5b64f-vmfrf

Status:

Running

Ready:

1/1

Restarts:

0

---

## 9. Kubernetes Service

The application is exposed using a Kubernetes NodePort service.

Service:

enterprise-app-enterprise-chart

Type:

NodePort

Application port:

5000

NodePort:

31567

Service:

5000:31567/TCP

---

## 10. Kubernetes Resource Configuration

The application container has resource requests and limits configured.

Requests:

CPU: 50m
Memory: 32Mi

Limits:

CPU: 250m
Memory: 128Mi

This prevents the application from consuming unlimited Kubernetes resources.

---

## 11. Health Probes

The Kubernetes deployment includes:

### Liveness Probe

HTTP GET:

/

Port:

5000

### Readiness Probe

HTTP GET:

/

Port:

5000

These probes allow Kubernetes to determine whether the application is alive and ready to receive traffic.

Some historical warning events occurred during pod startup while the application was not yet listening on port 5000.

The current pod is healthy:

READY: 1/1
STATUS: Running
RESTARTS: 0

Therefore, no further configuration change was required.

---

## 12. Helm Validation

Helm chart validation was performed using:

helm lint ./enterprise-chart

Result:

1 chart(s) linted, 0 chart(s) failed

The chart passed Helm lint validation.

---

## 13. Deployment Rollout Verification

Command:

kubectl rollout status deployment/enterprise-app-enterprise-chart

Result:

deployment "enterprise-app-enterprise-chart" successfully rolled out

This confirms that Kubernetes successfully completed the deployment rollout.

---

## 14. Application Verification

The Kubernetes NodePort was retrieved dynamically.

Node IP:

192.168.49.2

NodePort:

31567

Application test:

curl "http://${NODEIP}:${NODEPORT}"

Response:

<h1>Enterprise DevSecOps Pipeline</h1><p>Flask Application Running Successfully!</p>

This confirms that the deployed Flask application is reachable and functioning correctly.

---

## 15. Final Kubernetes State

Final pod:

enterprise-app-enterprise-chart-d59f5b64f-vmfrf

Status:

Running

Ready:

1/1

Restarts:

0

Deployment:

1/1 Available

Helm:

Revision 3

Status:

deployed

Application:

Successfully responding

---

## 16. Final Jenkins Result

The complete Jenkins pipeline finished successfully.

Pipeline stages completed:

1. Checkout
2. SonarQube Analysis
3. Build Docker Image
4. Trivy Scan
5. Push to Docker Hub
6. Deploy with Helm
7. Verify Deployment

Final result:

Finished: SUCCESS

---

# Final Project Architecture

Developer
    |
    v
GitHub
    |
    v
Jenkins
    |
    +--> SonarQube
    |      |
    |      +--> Static Code Analysis
    |
    +--> Docker Build
    |      |
    |      +--> Docker Image
    |
    +--> Trivy
    |      |
    |      +--> Vulnerability Scan
    |
    +--> Docker Hub
    |      |
    |      +--> enterprise-devsecops:31
    |
    +--> Helm
           |
           v
       Kubernetes
           |
           +--> Deployment
           |
           +--> Pod
           |
           +--> NodePort Service
           |
           v
       Flask Application

---

# DevSecOps Security Controls

The project implements the following security controls:

- SonarQube static code analysis
- Jenkins credential management
- Docker image vulnerability scanning
- HIGH/CRITICAL vulnerability pipeline gate
- Non-root/container hardening
- Minimal Python base image
- Multi-stage Docker build
- Kubernetes resource requests and limits
- Kubernetes liveness probes
- Kubernetes readiness probes
- Secure Docker Hub authentication through Jenkins credentials

---

# Technologies Used

- AWS EC2
- Ubuntu Linux
- Jenkins
- Git
- GitHub
- Docker
- Docker Hub
- Trivy
- SonarQube
- Kubernetes
- Minikube
- Helm
- Flask
- Gunicorn
- YAML
- Bash

---

# Final Project Outcome

The Enterprise DevSecOps project successfully demonstrates an end-to-end automated CI/CD workflow.

Source code is stored in GitHub.

Jenkins automatically performs:

- Source checkout
- Static code analysis
- Docker image creation
- Container security scanning
- Docker image publishing
- Helm-based Kubernetes deployment
- Kubernetes rollout verification

The final Docker image passed the Trivy security gate and was successfully deployed to Kubernetes.

The application was verified successfully through the Kubernetes NodePort.

Final application response:

Enterprise DevSecOps Pipeline
Flask Application Running Successfully!

Final pipeline status:

SUCCESS

---

# Day 15 Conclusion

Day 15 completes the final technical validation of the Enterprise DevSecOps AWS project.

The project now demonstrates a complete DevSecOps lifecycle:

CODE
→ BUILD
→ SCAN
→ SECURE
→ PUBLISH
→ DEPLOY
→ VERIFY

Project status:

COMPLETE

Total project duration:

15 Days

Final CI/CD status:

PASS

Final Kubernetes status:

HEALTHY

Final security scan:

0 HIGH/CRITICAL vulnerabilities