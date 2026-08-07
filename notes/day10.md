# Day 10 - Jenkins to Kubernetes Continuous Deployment

## Objective

Automate Kubernetes deployment after a successful Jenkins pipeline execution.

---

# Architecture

```
Developer
     │
     ▼
GitHub Repository
     │
     ▼
Jenkins Pipeline
     │
     ├── Checkout
     ├── SonarQube Analysis
     ├── Docker Build
     ├── Trivy Security Scan
     ├── Push Docker Image to Docker Hub
     └── Deploy to Kubernetes
                │
                ▼
         Rolling Update
                │
                ▼
        Flask Application
```

---

# Tasks Completed

## 1. Configured Jenkins Access to Kubernetes

Initially, Jenkins could not access the Kubernetes cluster.

Error:

```
Authentication required
```

Copied Kubernetes configuration to Jenkins.

```bash
sudo mkdir -p /var/lib/jenkins/.kube

sudo cp /home/ubuntu/.kube/config /var/lib/jenkins/.kube/config

sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
```

Copied Minikube certificates.

```bash
sudo cp -r /home/ubuntu/.minikube /var/lib/jenkins/

sudo chown -R jenkins:jenkins /var/lib/jenkins/.minikube
```

Updated kubeconfig paths.

```bash
sudo sed -i 's#/home/ubuntu/.minikube#/var/lib/jenkins/.minikube#g' /var/lib/jenkins/.kube/config
```

Verified Jenkins access.

```bash
sudo su - jenkins

kubectl get nodes

kubectl get deployments
```

Output:

```
minikube Ready

enterprise-app
```

---

## 2. Updated Jenkins Pipeline

Added Kubernetes deployment stage.

```groovy
stage('Deploy to Kubernetes') {
    steps {
        sh '''
        kubectl set image deployment/enterprise-app \
        enterprise-devsecops=billa1108/enterprise-devsecops:v1

        kubectl rollout status deployment/enterprise-app
        '''
    }
}
```

---

## 3. SonarQube Connectivity

Pipeline initially failed because Jenkins could not communicate with SonarQube.

Resolved by using localhost.

```groovy
-Dsonar.host.url=http://localhost:9000
```

---

## 4. Kubernetes Deployment Issue

Pipeline failed with:

```
error: unable to find container named "enterprise-app"
```

Verified deployment.

```bash
kubectl get deployment enterprise-app -o yaml | grep name
```

Output:

```
name: enterprise-app
name: enterprise-devsecops
```

Updated Jenkinsfile.

Changed from:

```bash
enterprise-app=billa1108/enterprise-devsecops:v1
```

to

```bash
enterprise-devsecops=billa1108/enterprise-devsecops:v1
```

---

## 5. Successful Pipeline

Pipeline executed successfully.

Stages completed:

- Checkout
- SonarQube Analysis
- Docker Build
- Trivy Scan
- Push Docker Hub
- Deploy to Kubernetes

Deployment output:

```
deployment "enterprise-app" successfully rolled out
```

---

# Commands Used

Check Kubernetes.

```bash
kubectl get nodes

kubectl get deployments

kubectl get pods

kubectl rollout status deployment/enterprise-app
```

---

# Key Learnings

- Kubernetes authentication
- Jenkins kubeconfig configuration
- Minikube certificate management
- Rolling Updates
- Continuous Deployment
- kubectl set image
- Kubernetes deployment automation
- Jenkins integration with Kubernetes

---

# Result

Successfully built an automated CI/CD pipeline.

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
Kubernetes
   │
   ▼
Automatic Rolling Update
```

---

# Day 10 Status

✅ Jenkins authenticated with Kubernetes

✅ Automatic deployment implemented

✅ Rolling updates working

✅ Complete CI/CD pipeline operational

---

# Next Step

Day 11

- Helm Installation
- Helm Charts
- values.yaml
- Deployment Templates
- Service Templates
- Helm Upgrade
- Helm Rollback