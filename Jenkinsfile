pipeline {
    agent any

    tools {
        sonarQube 'sonar-scanner'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
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
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=enterprise-devsecops-aws \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://13.203.219.97:9000
                    """
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