pipeline {
    agent any

    environment {
        IMAGE_REPO = "billa1108/enterprise-devsecops"
        IMAGE_TAG = "${BUILD_NUMBER}"
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
                sh '''
                    trivy image \
                    --severity HIGH,CRITICAL \
                    --exit-code 1 \
                    enterprise-devsecops:v1
                '''
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

                        docker tag enterprise-devsecops:v1 $IMAGE_REPO:$IMAGE_TAG
                        docker push $IMAGE_REPO:$IMAGE_TAG
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    helm upgrade --install enterprise-app ./enterprise-chart \
                    --set image.repository=$IMAGE_REPO \
                    --set image.tag=$IMAGE_TAG
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