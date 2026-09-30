pipeline {
    agent any

    environment {
        REGISTRY    = "ghcr.io/omairmomin"
        IMAGE_NAME  = "frontend"
        IMAGE_TAG   = "${env.BUILD_NUMBER}"
        CONFIG_REPO = "https://github.com/omairmomin/devops-platform-config.git"
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

        stage('Security Scan (Trivy)') {
            steps {
                sh '''
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --exit-code 0 \
                      --format table \
                      ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push to GHCR') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'ghcr-credentials',
                    usernameVariable: 'GHCR_USER',
                    passwordVariable: 'GHCR_TOKEN'
                )]) {
                    sh '''
                        echo "$GHCR_TOKEN" | docker login ghcr.io -u "$GHCR_USER" --password-stdin
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout ghcr.io
                    '''
                }
            }
        }

        stage('Update Config Repo') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-credentials',
                    usernameVariable: 'GH_USER',
                    passwordVariable: 'GH_TOKEN'
                )]) {
                    sh '''
                        rm -rf config-repo
                        git clone https://${GH_USER}:${GH_TOKEN}@github.com/omairmomin/devops-platform-config.git config-repo
                        cd config-repo
                        sed -i "s|ghcr.io/omairmomin/frontend:[^\\"[:space:]]*|${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}|" manifests/online-boutique.yaml
                        git add manifests/online-boutique.yaml
                        git commit -m "Update frontend image to ${IMAGE_TAG} [ci skip]" || echo "No changes to commit"
                        git push
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pushed ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} and updated config repo"
        }
        failure {
            echo "Build failed"
        }
    }
}
