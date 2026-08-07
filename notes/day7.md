# Day 7 - Docker + Trivy Integration

## Objective

Integrate Docker into the DevSecOps pipeline, containerize the Flask application, and perform container image vulnerability scanning using Trivy.

---

## Tasks Completed

### Docker

- Installed Docker on AWS EC2.
- Verified Docker installation.
- Configured Jenkins user to access Docker by adding it to the docker group.
- Restarted Docker and Jenkins services.

### Flask Application

Created a simple Flask application containing:

- app.py
- requirements.txt
- Dockerfile
- .dockerignore

Verified the application locally on:

http://localhost:5001

Successfully accessed the application through EC2 Public IP:

http://13.203.219.97:5000

---

### Docker Image

Built the Docker image:

```bash
docker build -t enterprise-devsecops:v1 .
```

Verified image:

```bash
docker images
```

Ran the container:

```bash
docker run -d --name enterprise-app -p 5000:5000 enterprise-devsecops:v1
```

Verified:

```bash
docker ps
```

---

### Trivy

Installed Trivy.

Verified installation:

```bash
trivy --version
```

Performed image vulnerability scan:

```bash
trivy image enterprise-devsecops:v1
```

Successfully integrated Trivy into the Jenkins pipeline.

---

## Jenkins Pipeline

Pipeline Stages:

- Checkout
- SonarQube Analysis
- Docker Build
- Trivy Image Scan

Pipeline executed successfully.

---

## Result

SUCCESS

Completed:

- Docker Installation
- Flask Application Containerization
- Docker Image Build
- Docker Container Deployment
- Trivy Image Vulnerability Scan
- Jenkins Docker Integration

---

## Key Learnings

- Building Docker images
- Running Docker containers
- Port Mapping
- Dockerfile creation
- Docker image scanning using Trivy
- Integrating Docker into Jenkins pipelines
- Basic container security scanning

---

## Next Step

- Create Docker Hub repository.
- Push Docker image to Docker Hub.
- Integrate Docker Hub push into Jenkins.
- Prepare for Kubernetes deployment.