pipeline {
    agent any

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
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('sonarqube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=enterprise-devsecops-aws \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://localhost:9000
                        """
                    }
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
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin

                        docker tag enterprise-devsecops:v1 $IMAGE_NAME
                        docker push $IMAGE_NAME
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    helm upgrade --install enterprise-app ./enterprise-chart
                '''
            }
        }

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
    }

    post {
        always {
            echo 'Pipeline Finished'
        }
    }
}
