pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Repository cloned successfully!'
            }
        }

        stage('Build') {
            steps {
                echo 'Starting build...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }
    }

    post {
        always {
            echo 'Pipeline Finished!'
        }
    }
}