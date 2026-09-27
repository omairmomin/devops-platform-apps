pipeline {
    agent any

    environment {
        IMAGE_NAME = "frontend"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('src/frontend') {
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Verify Image') {
            steps {
                sh 'docker images ${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }
    }

    post {
        success {
            echo "Build succeeded: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Build failed"
        }
    }
}
