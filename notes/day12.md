# Day 12 - Jenkins + Helm CI/CD Deployment

## Objective

Today I integrated Helm deployment into the Jenkins CI/CD pipeline and automated deployment of the application to Kubernetes.

The final pipeline is:

GitHub → Jenkins → SonarQube → Docker Build → Trivy → Docker Hub → Helm → Kubernetes → Verification

## 1. Helm Chart

Created the Helm chart:

enterprise-chart/
├── Chart.yaml
├── values.yaml
├── .helmignore
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── serviceaccount.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    ├── httproute.yaml
    ├── _helpers.tpl
    ├── NOTES.txt
    └── tests/
        └── test-connection.yaml

Validate the chart:

    helm lint .

Result:

    1 chart(s) linted, 0 chart(s) failed

Render the Helm templates:

    helm template enterprise-app .

## 2. Helm Deployment

Deploy the application using Helm:

    helm upgrade --install enterprise-app .

Check the release:

    helm list
    helm status enterprise-app

The Helm release was successfully deployed.

Final revision:

    REVISION: 6
    STATUS: deployed

## 3. Kubernetes Verification

Check pods:

    kubectl get pods

Check services:

    kubectl get svc

Check deployment rollout:

    kubectl rollout status deployment/enterprise-app-enterprise-chart

Result:

    deployment "enterprise-app-enterprise-chart" successfully rolled out

The application pod was running successfully.

## 4. Kubernetes Service Testing

Created a temporary test pod:

    kubectl run curl-test --image=curlimages/curl --restart=Never -- sleep 3600

Tested the Kubernetes service from inside the cluster:

    kubectl exec curl-test -- curl http://enterprise-app-enterprise-chart:5000

Application response:

    Enterprise DevSecOps Pipeline
    Flask Application Running Successfully!

This confirmed that:

- Kubernetes DNS resolution works
- The Kubernetes Service is reachable
- Service routing works
- Flask is running on port 5000

Deleted the temporary test pod:

    kubectl delete pod curl-test

## 5. Jenkins Helm Integration

Added a Helm deployment stage to Jenkinsfile:

    stage('Deploy with Helm') {
        steps {
            sh '''
                helm upgrade --install enterprise-app ./enterprise-chart
            '''
        }
    }

Added a verification stage:

    stage('Verify Deployment') {
        steps {
            sh '''
                kubectl rollout status deployment/enterprise-app-enterprise-chart
                kubectl get pods
                kubectl get svc
                helm list
            '''
        }
    }

## 6. Jenkins Pipeline

The Jenkins pipeline now performs:

1. Checkout code from GitHub
2. SonarQube analysis
3. Build Docker image
4. Run Trivy security scan
5. Push Docker image to Docker Hub
6. Deploy using Helm
7. Verify Kubernetes deployment

Pipeline flow:

GitHub
  ↓
Jenkins
  ↓
SonarQube
  ↓
Docker Build
  ↓
Trivy Scan
  ↓
Docker Hub
  ↓
Helm
  ↓
Kubernetes
  ↓
Deployment Verification

## 7. Helm Deployment Error

Initially Jenkins failed at the Helm deployment stage with:

    Error: path "./enterprise-chart" not found

### Cause

The enterprise-chart directory existed on the Ubuntu server but was not present in the Git repository.

Jenkins checks out the Git repository into its workspace, so Jenkins could not access a Helm chart that existed only on the server.

### Solution

Copied the Helm chart from the Ubuntu server to the local Git repository:

    scp -i ~/Downloads/devsecops-key.pem \
    -r ubuntu@13.232.137.121:~/enterprise-devsecops-aws/enterprise-chart .

Then added the chart to Git:

    git add enterprise-chart
    git commit -m "Add Helm chart for Kubernetes deployment"
    git push origin main

After pushing the Helm chart to GitHub, Jenkins was able to find:

    ./enterprise-chart

## 8. Jenkinsfile Merge Conflicts

There were multiple Git merge conflicts in Jenkinsfile while synchronizing local and remote changes.

Checked for conflict markers using:

    grep -nE '^(<<<<<<<|=======|>>>>>>>)' Jenkinsfile

After resolving the conflicts, the command returned no output.

Completed the rebase using:

    git add Jenkinsfile
    git rebase --continue

Then synchronized the repository using:

    git fetch origin
    git rebase origin/main
    git push origin main

## 9. Final Jenkins Pipeline Result

The final Jenkins pipeline completed successfully.

Docker login:

    Login Succeeded

Docker image was successfully pushed to Docker Hub.

Helm deployment:

    Release "enterprise-app" has been upgraded. Happy Helming!
    STATUS: deployed
    REVISION: 6

Kubernetes rollout:

    deployment "enterprise-app-enterprise-chart" successfully rolled out

Final Jenkins result:

    Finished: SUCCESS

## 10. Trivy Notice

Trivy displayed a version update notice:

    Version 0.73.0 of Trivy is now available,
    current version is 0.71.2

This was only an informational notice and did not cause the pipeline to fail.

The Trivy stage completed successfully.

## 11. Useful Helm Commands

    helm lint .
    helm template enterprise-app .
    helm upgrade --install enterprise-app .
    helm list
    helm status enterprise-app
    helm test enterprise-app
    helm history enterprise-app

## 12. Useful Kubernetes Commands

    kubectl get pods
    kubectl get svc
    kubectl get deployments
    kubectl rollout status deployment/enterprise-app-enterprise-chart
    kubectl get endpoints
    kubectl exec -it <pod> -- sh

## 13. Useful Git Commands

    git fetch origin
    git status
    git log --oneline --decorate --graph --all
    git rebase origin/main
    git add Jenkinsfile
    git rebase --continue
    git add enterprise-chart
    git commit -m "Add Helm chart for Kubernetes deployment"
    git push origin main

## Key Learnings

- Created and validated a Helm chart.
- Learned how Helm packages Kubernetes resources.
- Used helm lint to validate the chart.
- Used helm template to render Kubernetes manifests.
- Used helm upgrade --install for deployment.
- Integrated Helm into Jenkins.
- Automated Kubernetes deployment through Jenkins.
- Verified Kubernetes rollout from Jenkins.
- Tested Kubernetes Service connectivity from inside the cluster.
- Learned that deployment files required by Jenkins must exist in the Git repository.
- Troubleshot Jenkinsfile merge conflicts.
- Used Git rebase to synchronize local and remote branches.
- Integrated Jenkins, SonarQube, Docker, Trivy, Docker Hub, Helm, and Kubernetes into one DevSecOps pipeline.

## Final Architecture

GitHub
  ↓
Jenkins
  ↓
SonarQube Analysis
  ↓
Docker Build
  ↓
Trivy Security Scan
  ↓
Docker Hub
  ↓
Helm Deployment
  ↓
Kubernetes
  ↓
Flask Application

## Day 12 Result

Successfully completed the Jenkins + Helm CI/CD integration.

The complete pipeline is now:

Code → SonarQube → Docker Build → Trivy → Docker Hub → Helm → Kubernetes → Verification

Final Jenkins status: SUCCESS