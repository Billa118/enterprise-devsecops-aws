# Day 6 - Jenkins + SonarQube Integration

## Objective

Integrate SonarQube with Jenkins and execute the first static code analysis.

---

## Tasks Completed

- Installed SonarQube Scanner plugin in Jenkins
- Configured SonarScanner tool
- Created SonarQube project
- Generated project authentication token
- Added SonarQube token to Jenkins Credentials
- Configured SonarQube Server in Jenkins
- Updated Jenkinsfile to include SonarQube Analysis stage
- Executed first static code analysis from Jenkins
- Successfully published analysis report to SonarQube

---

## Jenkins Pipeline

Checkout Source Code

↓

SonarQube Analysis

↓

Publish Report

---

## Key Learnings

- Jenkins integrates with SonarQube using authentication tokens.
- SonarScanner performs static code analysis.
- Jenkins automatically downloads configured tools.
- SonarQube provides centralized code quality reports.
- Static code analysis executes before application deployment.

---

## Result

Pipeline Status:

SUCCESS

SonarQube Status:

ANALYSIS SUCCESSFUL

---

## Next Step

- Install Docker
- Build Docker image
- Scan Docker image using Trivy
- Push image to Docker Hub