# Day 8 - Docker Hub Integration

## Objective

Integrate Docker Hub with the DevSecOps pipeline to automatically store Docker images in a remote container registry.

---

## Tasks Completed

### Docker Hub

- Created Docker Hub account.
- Created public repository:
  - enterprise-devsecops

Docker Hub Username:

```
billa1108
```

---

### Docker Login

Authenticated Docker CLI using Docker Hub credentials.

```bash
docker login
```

Login Status:

```
Login Succeeded
```

---

### Tag Docker Image

Tagged the locally built Docker image for Docker Hub.

```bash
docker tag enterprise-devsecops:v1 billa1108/enterprise-devsecops:v1
```

---

### Push Image to Docker Hub

Successfully pushed the Docker image.

```bash
docker push billa1108/enterprise-devsecops:v1
```

Image uploaded successfully.

Repository:

```
docker.io/billa1108/enterprise-devsecops:v1
```

---

### Jenkins Integration

Added Docker Hub credentials in Jenkins.

Credential Type:

- Username with Password

Credential ID:

```
dockerhub
```

Configured Jenkins to securely access Docker Hub during pipeline execution.

---

### Jenkins Pipeline

Updated Jenkinsfile.

Pipeline stages:

- Checkout
- SonarQube Analysis
- Build Docker Image
- Trivy Image Scan
- Push Docker Image to Docker Hub

Successfully executed Jenkins pipeline.

---

## Pipeline Architecture

```
GitHub
   │
   ▼
Jenkins
   │
   ▼
Checkout
   │
   ▼
SonarQube Analysis
   │
   ▼
Docker Build
   │
   ▼
Trivy Image Scan
   │
   ▼
Docker Hub Push
   │
   ▼
Pipeline Success
```

---

## Result

Completed:

- Docker Hub Repository Creation
- Docker Authentication
- Docker Image Tagging
- Docker Image Push
- Jenkins Docker Credentials
- Automated Docker Hub Push

---

## Key Learnings

- Docker Hub Registry
- Docker Login
- Docker Image Tagging
- Docker Image Versioning
- Secure Jenkins Credentials
- Automated Docker Image Publishing
- CI Pipeline Enhancement

---

## Next Step

Deploy the Docker image to Kubernetes.

Topics:

- Kubernetes
- Pods
- Deployments
- Services
- ReplicaSets
- Rolling Updates
- Jenkins to Kubernetes Deployment