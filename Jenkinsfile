pipeline {
    agent any

    tools {
        sonarRunner 'sonar-scanner'
    }

    environment {
        IMAGE_NAME = "billa1108/enterprise-devsecops:v1"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    ${tool 'sonar-scanner'}/bin/sonar-scanner \
                      -Dsonar.projectKey=enterprise-devsecops-aws \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://13.203.219.97:9000
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('apps') {
                    sh 'docker build -t enterprise-devsecops:v1 .'
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image enterprise-devsecops:v1'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker tag enterprise-devsecops:v1 $IMAGE_NAME
                    docker push $IMAGE_NAME
                    docker logout
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline Finished'
        }
    }
}