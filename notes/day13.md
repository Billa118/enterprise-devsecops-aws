# Day 13 – DevSecOps CI/CD Pipeline with Kubernetes & Helm

## Objective

Complete the DevSecOps CI/CD pipeline by integrating:

- GitHub
- Jenkins
- SonarQube
- Docker
- Trivy
- Docker Hub
- Kubernetes
- Helm

Final pipeline:

GitHub → Jenkins → SonarQube → Docker Build → Trivy Scan → Docker Hub → Helm → Kubernetes → Verification


## 1. Jenkins Access to Minikube

Jenkins needed access to the Minikube Kubernetes cluster.

Jenkins user:

jenkins

Jenkins home:

/var/lib/jenkins

Kubernetes configuration:

/var/lib/jenkins/.kube/config

Minikube certificates were made available to Jenkins under:

/var/lib/jenkins/.kube/minikube/

Required certificate files:

- ca.crt
- client.crt
- client.key

Jenkins Kubernetes access was verified using:

sudo -u jenkins kubectl get nodes

Expected result:

NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   v1.35.1


## 2. Minikube Verification

Check Minikube:

minikube status

Result:

type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

Kubernetes version:

v1.35.1

Container runtime:

docker://29.2.1

Check Kubernetes nodes:

sudo -u jenkins kubectl get nodes

Result:

minikube   Ready   control-plane


## 3. Helm Chart Validation

The Kubernetes application is packaged using Helm.

Project structure:

enterprise-devsecops-aws/
├── Jenkinsfile
├── apps/
├── enterprise-chart/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
└── notes/

Validate the Helm chart:

helm lint ./enterprise-chart

Result:

1 chart(s) linted, 0 chart(s) failed

The Helm chart is valid.


## 4. Kubernetes Resource Limits

The EC2 instance has limited memory because Jenkins, SonarQube, Docker and Minikube are running on the same server.

Memory was checked using:

free -h

Example:

Mem:   1.9Gi
Swap:  2.0Gi

The Kubernetes application was therefore configured with resource requests and limits.

enterprise-chart/values.yaml:

resources:
  requests:
    cpu: 50m
    memory: 32Mi
  limits:
    cpu: 250m
    memory: 128Mi

This prevents the Flask application from consuming excessive CPU and memory.


## 5. SonarQube Integration

Jenkins performs SonarQube analysis before building the Docker image.

SonarQube project:

enterprise-devsecops-aws

Source directory:

apps

Jenkins credential:

sonarqube-token

Scanner configuration:

-Dsonar.projectKey=enterprise-devsecops-aws
-Dsonar.sources=apps
-Dsonar.host.url=http://localhost:9000
-Dsonar.token=${SONAR_TOKEN}
-Dsonar.python.version=3.11

SonarQube successfully processed the project.

Result:

ANALYSIS SUCCESSFUL

Note:

SonarQube 9.9 reported that Python 3.11 did not have explicit analyzer support and therefore used Python 3.10 for analysis.

This was only a warning and did not fail the pipeline.


## 6. Docker Image Build

The Flask application is containerized using Docker.

Jenkins builds the image from the apps directory:

docker build -t enterprise-devsecops:v1 .

The Docker image uses:

Python 3.11
Debian Trixie

The Docker image was successfully built.


## 7. Trivy Security Scan

After building the Docker image, Jenkins scans it using Trivy.

Command:

trivy image \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  --exit-code 1 \
  enterprise-devsecops:v1

Purpose:

- Scan the Docker image for vulnerabilities
- Detect HIGH vulnerabilities
- Detect CRITICAL vulnerabilities
- Ignore vulnerabilities that do not yet have a fix
- Fail the Jenkins pipeline if HIGH/CRITICAL vulnerabilities are detected

Result:

Vulnerabilities: 0

The container image passed the Trivy security gate.


## 8. Push Image to Docker Hub

After the Trivy scan passed, Jenkins logs into Docker Hub using Jenkins credentials.

Image:

billa1108/enterprise-devsecops

The pipeline tagged the image as:

billa1108/enterprise-devsecops:30

Push command:

docker push billa1108/enterprise-devsecops:30

The image was successfully pushed.

Digest:

sha256:cf35c59ff79ffea1b83038703aa2184a5a7096fb04dd2b61763907591b079b0e

Jenkins logs out after the push:

docker logout


## 9. Helm Deployment

The Docker image was deployed to Kubernetes using Helm.

Command:

helm upgrade --install enterprise-app ./enterprise-chart \
  --set image.repository=billa1108/enterprise-devsecops \
  --set image.tag=30

Result:

Release "enterprise-app" has been upgraded.

STATUS: deployed
REVISION: 2

Helm release:

enterprise-app

Chart:

enterprise-chart-0.1.0


## 10. Kubernetes Deployment

The application runs inside Kubernetes as a Deployment.

Check deployments:

sudo -u jenkins kubectl get deployments

Result:

NAME                              READY
enterprise-app-enterprise-chart   1/1

Check pods:

sudo -u jenkins kubectl get pods

Result:

enterprise-app-enterprise-chart-...   1/1   Running


## 11. Kubernetes Service

The Flask application is exposed through a NodePort service.

Check service:

sudo -u jenkins kubectl get svc enterprise-app-enterprise-chart

Result:

NAME                              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
enterprise-app-enterprise-chart   NodePort   10.109.39.131   <none>        5000:31567/TCP

Application port:

5000

NodePort:

31567


## 12. Get Kubernetes Node IP

Retrieve the NodePort:

NODEPORT=$(sudo -u jenkins kubectl get svc enterprise-app-enterprise-chart \
-o jsonpath='{.spec.ports[0].nodePort}')

Retrieve the Minikube node IP:

NODEIP=$(sudo -u jenkins kubectl get node minikube \
-o jsonpath='{.status.addresses[?(@.type=="InternalIP")].address}')

Check:

echo $NODEPORT
echo $NODEIP

Example:

NODEPORT=31567
NODEIP=192.168.49.2


## 13. Application Verification

Test the application:

curl "http://${NODEIP}:${NODEPORT}"

Result:

<h1>Enterprise DevSecOps Pipeline</h1>
<p>Flask Application Running Successfully!</p>

This confirms that:

- Kubernetes Service is working
- NodePort is working
- Flask application is running
- Application traffic reaches the pod


## 14. Kubernetes Rollout Verification

Jenkins verifies the Kubernetes rollout using:

kubectl rollout status deployment/enterprise-app-enterprise-chart

Result:

deployment "enterprise-app-enterprise-chart" successfully rolled out

This ensures the pipeline does not finish successfully until Kubernetes confirms that the deployment is available.


## 15. Helm Verification

Check Helm releases:

helm list

Result:

NAME             NAMESPACE   REVISION   STATUS
enterprise-app   default     2           deployed

Helm successfully manages the Kubernetes application.


## 16. Problems Solved During Day 13

### Problem 1 – Jenkins could not access Kubernetes

Jenkins initially had problems communicating with Minikube.

Cause:

Jenkins was using incorrect/outdated Minikube kubeconfig and certificate paths.

Solution:

- Configure Jenkins kubeconfig
- Copy required Minikube certificates
- Correct certificate paths
- Fix permissions
- Test with Jenkins user

Verification:

sudo -u jenkins kubectl get nodes


### Problem 2 – Container Exit Code 137

The application previously terminated with:

Exit Code: 137

The EC2 server has only around 2 GB RAM while running:

- Jenkins
- SonarQube
- Docker
- Minikube
- Kubernetes components

Resource limits were added to the Helm chart:

resources:
  requests:
    cpu: 50m
    memory: 32Mi
  limits:
    cpu: 250m
    memory: 128Mi

This reduced the application's resource consumption.


### Problem 3 – Kubernetes Readiness Probe Failures

The application initially experienced readiness/liveness probe failures during startup.

The deployment was eventually able to become healthy.

Current probes:

livenessProbe:
  httpGet:
    path: /
    port: http

readinessProbe:
  httpGet:
    path: /
    port: http


### Problem 4 – Docker Image Pull Failure

During the earlier deployment, Kubernetes temporarily reported:

ErrImagePull

and:

ImagePullBackOff

Cause:

The Minikube environment temporarily experienced a network timeout while pulling the Docker image from Docker Hub.

The image was successfully pulled afterward and the pod started.


### Problem 5 – Git Remote Divergence

Local Git changes and remote Git changes had diverged.

The local branch was behind the remote by four commits.

The changes were committed:

git commit -m "Stabilize SonarQube pipeline and Kubernetes resources"

Then the local branch was rebased:

git pull --rebase origin main

A Jenkinsfile conflict occurred.

The conflict was resolved and the rebase completed successfully.

Final commit:

2a5b471 Stabilize SonarQube pipeline and Kubernetes resources

The commit was pushed successfully:

git push origin main


## 17. Final Jenkins Pipeline

The completed pipeline is:

GitHub
   |
   v
Jenkins
   |
   +---- Checkout Source Code
   |
   +---- SonarQube Analysis
   |
   +---- Docker Image Build
   |
   +---- Trivy Security Scan
   |          |
   |          +---- Fail on HIGH/CRITICAL
   |
   +---- Push Image to Docker Hub
   |
   +---- Deploy with Helm
   |
   +---- Kubernetes Rollout Verification
   |
   v
Application Running


## 18. Final Pipeline Results

SonarQube Analysis       SUCCESS
Docker Build             SUCCESS
Trivy Scan               SUCCESS
Docker Hub Push          SUCCESS
Helm Deployment          SUCCESS
Kubernetes Rollout       SUCCESS

Final Jenkins result:

Finished: SUCCESS


## 19. Important Commands Used

Check memory:

free -h

Check Docker containers:

docker stats --no-stream

Check running processes:

ps aux --sort=-%mem | head -15

Check Minikube:

minikube status

Check Kubernetes nodes:

sudo -u jenkins kubectl get nodes

Check pods:

sudo -u jenkins kubectl get pods

Check deployments:

sudo -u jenkins kubectl get deployments

Check services:

sudo -u jenkins kubectl get svc

Check Helm releases:

sudo -u jenkins helm list

Check Helm chart:

helm lint ./enterprise-chart

Check pod logs:

sudo -u jenkins kubectl logs <POD_NAME>

Describe pod:

sudo -u jenkins kubectl describe pod <POD_NAME>

Check rollout:

kubectl rollout status deployment/enterprise-app-enterprise-chart

Get NodePort:

NODEPORT=$(sudo -u jenkins kubectl get svc enterprise-app-enterprise-chart \
-o jsonpath='{.spec.ports[0].nodePort}')

Get Node IP:

NODEIP=$(sudo -u jenkins kubectl get node minikube \
-o jsonpath='{.status.addresses[?(@.type=="InternalIP")].address}')

Test application:

curl "http://${NODEIP}:${NODEPORT}"


## 20. Final Project Status

Day 13 is COMPLETE.

The project now contains a working end-to-end DevSecOps CI/CD pipeline with:

- GitHub source control
- Jenkins CI/CD
- SonarQube static code analysis
- Docker containerization
- Trivy security scanning
- Docker Hub image registry
- Kubernetes deployment
- Helm deployment
- Kubernetes resource limits
- Kubernetes health probes
- Automated Kubernetes rollout verification

Final architecture:

GitHub
   ↓
Jenkins
   ↓
SonarQube
   ↓
Docker Build
   ↓
Trivy Security Scan
   ↓
Docker Hub
   ↓
Helm
   ↓
Kubernetes / Minikube
   ↓
Flask Application


## Day 13 Milestone

Built and successfully validated an end-to-end DevSecOps CI/CD pipeline that performs source checkout, static code analysis, containerization, vulnerability scanning, container registry publishing, Helm-based Kubernetes deployment, and automated deployment verification.

Pipeline result:

GITHUB → JENKINS → SONARQUBE → DOCKER → TRIVY → DOCKER HUB → HELM → KUBERNETES

STATUS: SUCCESS

DAY 13: COMPLETE