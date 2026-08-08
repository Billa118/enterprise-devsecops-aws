# Day 11 - Helm Deployment

## Objective

Introduce Helm into the Kubernetes deployment and deploy the Enterprise DevSecOps Flask application using a Helm chart.

---

## 1. Install Helm

Installed Helm on the EC2 server:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

helm version

helm create enterprise-chart

enterprise-chart/
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── serviceaccount.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    └── ...

    image:
  repository: billa1108/enterprise-devsecops
  pullPolicy: Always
  tag: "v1"

  service:
  type: NodePort
  port: 5000

  containers:
  - name: enterprise-chart
    image: "billa1108/enterprise-devsecops:v1"
    imagePullPolicy: Always
    ports:
      - name: http
        containerPort: 5000

        livenessProbe:
  httpGet:
    path: /
    port: http

readinessProbe:
  httpGet:
    path: /
    port: http

    helm lint .

    minikube status

    minikube start --driver=docker

    kubectl get nodes

    NAME       STATUS   ROLES           VERSION
minikube   Ready    control-plane   v1.35.1

helm install enterprise-app .

NAME: enterprise-app
STATUS: deployed
REVISION: 1

helm upgrade enterprise-app .

REVISION: 4
STATUS: deployed

helm list

helm status enterprise-app

helm history enterprise-app

enterprise-app
STATUS: deployed
REVISION: 4

kubectl get pods
enterprise-app-enterprise-chart-xxxxx   1/1   Running

minikube service enterprise-app-enterprise-chart --url
http://192.168.58.2:30219


Test Pod
   ↓
Kubernetes DNS
   ↓
Helm Service
   ↓
Application Pod
   ↓
Flask Application


helm version
helm create enterprise-chart
helm lint .
helm install enterprise-app .
helm upgrade enterprise-app .
helm list
helm status enterprise-app
helm history enterprise-app
helm get manifest enterprise-app
helm get values enterprise-app --all

Day 11 Outcome
Successfully introduced Helm into the project.
Completed:
Helm installation
Helm chart creation
Helm values.yaml configuration
Kubernetes Deployment template
Kubernetes Service template
Docker Hub image configuration
Health probes
Helm installation
Helm upgrades
Helm release management
Kubernetes service verification
Internal Kubernetes DNS testing
Flask application connectivity testing

Current Architecture
GitHub
   ↓
Jenkins
   ↓
SonarQube
   ↓
Docker Build
   ↓
Trivy
   ↓
Docker Hub
   ↓
Kubernetes
   ↓
Helm
   ↓
Flask Application
