# Day 14 — Kubernetes Production Validation & Deployment Hardening

## Objective

Validate the Kubernetes deployment created through the Day 13 DevSecOps pipeline and confirm that the application is healthy, the Helm chart is valid, resources are configured, probes are working, the deployment rolls out successfully, and the application is accessible through the Kubernetes NodePort.

---

# 1. Environment

Project:

    enterprise-devsecops-aws

AWS EC2 environment:

    Ubuntu EC2
    Jenkins
    Docker
    SonarQube
    Minikube
    Kubernetes
    Helm

Kubernetes:

    Minikube
    Kubernetes v1.35.1
    Docker container runtime

Application:

    Flask
    Port: 5000

---

# 2. Check EC2 Memory

Command:

    free -h

Observed environment:

    RAM: approximately 1.9 GiB
    Swap: approximately 2.0 GiB

The EC2 instance is memory constrained because Jenkins, SonarQube, Minikube, Kubernetes components and Docker are running simultaneously.

Docker memory usage was checked using:

    docker stats --no-stream

Observed containers:

    minikube
    sonarqube

Important lesson:

    Avoid increasing Kubernetes replicas or running unnecessary services on a low-memory EC2 instance.

Earlier in the project, the application experienced:

    Exit Code 137

This reinforced the importance of monitoring memory usage on the EC2 host.

---

# 3. Verify Minikube

Command:

    minikube status

Expected/observed:

    type: Control Plane
    host: Running
    kubelet: Running
    apiserver: Running
    kubeconfig: Configured

Minikube cluster was healthy.

---

# 4. Verify Kubernetes Node

Command:

    sudo -u jenkins kubectl get nodes

Observed:

    NAME       STATUS   ROLES           VERSION
    minikube   Ready    control-plane   v1.35.1

The Kubernetes node was successfully Ready.

---

# 5. Verify Kubernetes Resources

Command:

    sudo -u jenkins kubectl get all

Observed:

    Pod:
    enterprise-app-enterprise-chart-6cf979fcfb-tqfng
    1/1 Running

    Service:
    enterprise-app-enterprise-chart
    NodePort
    5000:31567/TCP

    Deployment:
    enterprise-app-enterprise-chart
    1/1 Ready

The previous ReplicaSet was scaled down and the new ReplicaSet became active.

---

# 6. Verify Pod Health

Command:

    sudo -u jenkins kubectl get pods -o wide

Observed:

    NAME                                               READY   STATUS    RESTARTS
    enterprise-app-enterprise-chart-6cf979fcfb-tqfng   1/1     Running   0

Pod IP:

    10.244.0.8

Node:

    minikube

The current application pod was healthy with zero restarts.

---

# 7. Validate Helm Chart

Command:

    sudo -u jenkins helm lint ./enterprise-chart

Result:

    ==> Linting ./enterprise-chart
    [INFO] Chart.yaml: icon is recommended
    1 chart(s) linted, 0 chart(s) failed

The icon message is only a recommendation and does not represent a chart failure.

Helm chart validation passed successfully.

---

# 8. Kubernetes Resource Requests and Limits

The Helm values file was updated to define resource requests and limits:

    resources:
      requests:
        cpu: 50m
        memory: 32Mi
      limits:
        cpu: 250m
        memory: 128Mi

Command:

    sudo -u jenkins kubectl get pod -o jsonpath='{.items[0].spec.containers[0].resources}'

Observed:

    limits:
      cpu: 250m
      memory: 128Mi

    requests:
      cpu: 50m
      memory: 32Mi

Purpose:

    Requests reserve baseline resources for scheduling.
    Limits prevent the container from consuming unlimited CPU/memory.

This is an important Kubernetes production-hardening practice.

---

# 9. Verify Deployment Configuration

Command:

    sudo -u jenkins kubectl describe deployment enterprise-app-enterprise-chart

Important configuration observed:

    Image:
    billa1108/enterprise-devsecops:30

    Container Port:
    5000/TCP

    CPU Request:
    50m

    Memory Request:
    32Mi

    CPU Limit:
    250m

    Memory Limit:
    128Mi

    Replicas:
    1 desired
    1 updated
    1 available
    0 unavailable

Deployment strategy:

    RollingUpdate

---

# 10. Liveness Probe

Configured in Helm values:

    livenessProbe:
      httpGet:
        path: /
        port: http

Purpose:

    Kubernetes periodically checks whether the application is alive.

If the application becomes unhealthy, Kubernetes can restart the container.

---

# 11. Readiness Probe

Configured in Helm values:

    readinessProbe:
      httpGet:
        path: /
        port: http

Purpose:

    Determines whether the application is ready to receive traffic.

A pod that is not ready should not receive traffic through the Kubernetes Service.

---

# 12. Historical Probe Warnings

Kubernetes events showed earlier warnings such as:

    Liveness probe failed
    Readiness probe failed
    connection refused
    context deadline exceeded

These occurred during the previous pod startup/rollout.

The old pod was:

    enterprise-app-enterprise-chart-7d9b9c9cfd-slnm6

It was eventually terminated and replaced.

The current pod:

    enterprise-app-enterprise-chart-6cf979fcfb-tqfng

is:

    READY: true
    STATUS: Running
    RESTARTS: 0

Therefore, the historical warnings were not treated as an active deployment failure.

---

# 13. Determine NodePort

Command:

    NODEPORT=$(sudo -u jenkins kubectl get svc enterprise-app-enterprise-chart -o jsonpath='{.spec.ports[0].nodePort}')

    echo $NODEPORT

Observed:

    31567

---

# 14. Determine Minikube Node IP

Command:

    NODEIP=$(sudo -u jenkins kubectl get node minikube -o jsonpath='{.status.addresses[?(@.type=="InternalIP")].address}')

    echo $NODEIP

Observed:

    192.168.49.2

---

# 15. Test Application Through Kubernetes

Command:

    curl "http://${NODEIP}:${NODEPORT}"

Observed response:

    <h1>Enterprise DevSecOps Pipeline</h1>
    <p>Flask Application Running Successfully!</p>

This confirmed that:

    Client
      ↓
    Minikube Node IP
      ↓
    NodePort 31567
      ↓
    Kubernetes Service
      ↓
    Pod
      ↓
    Flask Application :5000

The application was successfully accessible through Kubernetes.

---

# 16. Verify Kubernetes Rollout

Command:

    sudo -u jenkins kubectl rollout status deployment/enterprise-app-enterprise-chart

Result:

    deployment "enterprise-app-enterprise-chart" successfully rolled out

This confirmed that the Kubernetes Deployment completed successfully.

---

# 17. Verify Deployment Availability

Command:

    sudo -u jenkins kubectl get deployment

Observed:

    NAME                              READY   UP-TO-DATE   AVAILABLE
    enterprise-app-enterprise-chart   1/1     1            1

The deployment had:

    1 desired replica
    1 updated replica
    1 available replica
    0 unavailable replicas

---

# 18. Verify Helm Release

Command:

    sudo -u jenkins helm list

Observed:

    NAME             NAMESPACE   REVISION   STATUS
    enterprise-app   default     2          deployed

Chart:

    enterprise-chart-0.1.0

Application version:

    1.16.0

---

# 19. Verify Helm Status

Command:

    sudo -u jenkins helm status enterprise-app

Observed:

    NAME: enterprise-app
    NAMESPACE: default
    STATUS: deployed
    REVISION: 2

This confirmed that the application was successfully deployed and managed by Helm.

---

# 20. Verify Deployed Docker Image

Command:

    sudo -u jenkins kubectl get deployment enterprise-app-enterprise-chart \
    -o jsonpath='{.spec.template.spec.containers[0].image}'

Observed:

    billa1108/enterprise-devsecops:30

This verified the complete image promotion flow:

    Jenkins
       ↓
    Docker Build
       ↓
    Trivy Scan
       ↓
    Docker Hub
       ↓
    Image :30
       ↓
    Helm
       ↓
    Kubernetes

---

# 21. Kubernetes Event Validation

Command:

    sudo -u jenkins kubectl get events --sort-by='.lastTimestamp'

Warning events were observed from the previous rollout/startup.

Important current-state validation:

    Current Pod: 1/1 Running
    Current Pod Restarts: 0
    Deployment: Available
    Deployment: Progressing
    Rollout: Successfully rolled out

Therefore, there was no active application failure at the end of Day 14.

---

# 22. Final Day 14 Validation

Final Kubernetes state:

    Minikube              → Running
    Kubernetes Node       → Ready
    Pod                   → 1/1 Running
    Pod Restarts          → 0
    Deployment            → 1/1 Available
    Helm                  → deployed
    Helm Revision         → 2
    Docker Image          → billa1108/enterprise-devsecops:30
    Helm Lint             → Passed
    Resource Requests     → Configured
    Resource Limits       → Configured
    Liveness Probe        → Configured
    Readiness Probe       → Configured
    Rollout               → Successful
    NodePort              → 31567
    Application Test      → Successful

Application response:

    Enterprise DevSecOps Pipeline
    Flask Application Running Successfully!

---

# 23. DevSecOps Architecture Validated

The project now has the following validated flow:

    Developer
        |
        v
    GitHub
        |
        v
    Jenkins CI/CD
        |
        +--------------------+
        |                    |
        v                    v
    SonarQube              Docker Build
    Code Analysis              |
                               v
                            Trivy
                        Vulnerability Scan
                               |
                               v
                          Docker Hub
                               |
                               v
                             Helm
                               |
                               v
                         Kubernetes
                           Minikube
                               |
                               v
                         Flask App
                         Port 5000

---

# 24. Key Day 14 Learnings

## Kubernetes Resource Management

Requests define the minimum resources required by a container.

Limits define the maximum resources the container can consume.

## Liveness vs Readiness

Liveness:

    Is the application alive?

Readiness:

    Is the application ready to receive traffic?

## Rolling Updates

Kubernetes can replace an old ReplicaSet with a new ReplicaSet without manually deleting the application.

## Helm

Helm provides package and release management for Kubernetes applications.

Useful commands:

    helm lint
    helm list
    helm status
    helm upgrade --install

## Kubernetes Deployment Validation

Useful commands:

    kubectl get pods
    kubectl get deployment
    kubectl describe deployment
    kubectl rollout status
    kubectl get events

## Application Validation

Infrastructure being "Running" is not enough.

The actual application endpoint was tested using:

    curl

This verified end-to-end connectivity.

---

# 25. Day 14 Completion

Day 14 objective:

    Kubernetes Production Validation & Deployment Hardening

Status:

    COMPLETE

The Kubernetes deployment, Helm release, resource configuration, health probes, rollout process, Docker image version, and application accessibility were successfully validated.

---

# Day 15 Preview

Day 15 is the FINAL project day.

Focus:

    Final Jenkins pipeline validation
    Final Kubernetes validation
    Project cleanup
    README finalization
    Architecture documentation
    GitHub repository cleanup
    Final project documentation
    Resume-ready project description
    Interview explanation
    Final project review

No additional project days are planned.

Target:

    15-Day Enterprise DevSecOps Project — COMPLETE