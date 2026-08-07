# Day 9 - Kubernetes Deployment

## Objective

Deploy the containerized Flask application from Docker Hub to a Kubernetes cluster using Minikube.

---

## Tasks Completed

### Kubernetes Environment

Verified kubectl installation.

```bash
kubectl version --client
```

Installed Minikube.

Verified installation.

```bash
minikube version
```

Started Kubernetes cluster using Docker driver.

```bash
minikube start --driver=docker
```

---

## Cluster Verification

Verified cluster status.

```bash
kubectl get nodes
```

Output:

```
NAME        STATUS   ROLES           VERSION
minikube    Ready    control-plane   v1.35.1
```

Verified Kubernetes system pods.

```bash
kubectl get pods -A
```

---

## Resource Optimization

Stopped SonarQube container to free memory on the EC2 instance.

```bash
docker stop sonarqube
```

Verified available memory.

```bash
free -h
```

---

## Application Deployment

Created Kubernetes deployment using Docker Hub image.

```bash
kubectl create deployment enterprise-app \
--image=billa1108/enterprise-devsecops:v1
```

Verified deployment.

```bash
kubectl get deployments
```

Verified pod.

```bash
kubectl get pods
```

Deployment Status:

```
READY: 1/1
STATUS: Running
```

---

## Service Creation

Exposed deployment using NodePort.

```bash
kubectl expose deployment enterprise-app \
--type=NodePort \
--port=5000
```

Verified service.

```bash
kubectl get svc
```

Output:

```
Service Type : NodePort
Application Port : 5000
NodePort : 31709
```

---

## Access Application

Generated application URL.

```bash
minikube service enterprise-app --url
```

Example:

```
http://192.168.58.2:31709
```

Successfully deployed Flask application on Kubernetes.

---

## Kubernetes Components Learned

- Cluster
- Node
- Pod
- Deployment
- ReplicaSet
- Service
- NodePort
- Labels
- Selectors

---

## Project Architecture

```
GitHub
   │
   ▼
Jenkins
   │
   ▼
SonarQube
   │
   ▼
Docker Build
   │
   ▼
Trivy Scan
   │
   ▼
Docker Hub
   │
   ▼
Kubernetes Deployment
   │
   ▼
Flask Application
```

---

## Key Learnings

- Kubernetes architecture
- Deployments
- Pods
- ReplicaSets
- Services
- NodePort
- Docker Hub integration with Kubernetes
- Container orchestration
- Application deployment using kubectl

---

## Result

Successfully completed:

- Kubernetes Cluster Setup
- Minikube Installation
- Application Deployment
- Service Creation
- Application Exposure
- Docker Hub Integration with Kubernetes

---

## Next Step

Automate Kubernetes deployment using Jenkins.

Topics:

- Kubernetes Credentials
- kubectl from Jenkins
- Automated Deployment
- Rolling Updates