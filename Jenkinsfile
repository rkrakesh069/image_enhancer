pipeline {

    agent any

    environment {
        DOCKER_IMAGE_BACKEND  = 'rkrakesh069/image-enhancer-backend'
        DOCKER_IMAGE_FRONTEND = 'rkrakesh069/image-enhancer-frontend'
        IMAGE_TAG = "${BUILD_NUMBER}"

        DEPLOY_USER = "ubuntu"
        DEPLOY_HOST = "172.31.2.203"
        DEPLOY_PATH = "/opt/image-enhancer"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }
    
    stage('Validate Compose') {
            steps {
                sh '''
                    docker compose config
                '''
            }
        }


        stage('Build Backend') {
            steps {
                sh '''
                    docker build \
                        -t ${DOCKER_IMAGE_BACKEND}:${IMAGE_TAG} \
                        -t ${DOCKER_IMAGE_BACKEND}:latest \
                        ./backend
                '''
            }
        }


        stage('Build Frontend') {
            steps {
                sh '''
                    docker build \
                        -t ${DOCKER_IMAGE_FRONTEND}:${IMAGE_TAG} \
                        -t ${DOCKER_IMAGE_FRONTEND}:latest \
                        ./frontend
                '''
            }
        }


        stage('Login Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }


        stage('Push Backend') {
            steps {
                sh '''
                    docker push ${DOCKER_IMAGE_BACKEND}:${IMAGE_TAG}
                    docker push ${DOCKER_IMAGE_BACKEND}:latest
                '''
            }
        }


        stage('Push Frontend') {
            steps {
                sh '''
                    docker push ${DOCKER_IMAGE_FRONTEND}:${IMAGE_TAG}
                    docker push ${DOCKER_IMAGE_FRONTEND}:latest
                '''
            }
        }

        stage('Deploy') {
            steps {

                sshagent(['deploy-server-ssh']) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            ubuntu@172.31.2.203 \
                            "cd /opt/image-enhancer && \
                             IMAGE_TAG=${IMAGE_TAG} docker compose pull && \
                             IMAGE_TAG=${IMAGE_TAG} docker compose up -d"
                    '''
                }
            }
        }
    }


    post {
        always {
            sh 'docker logout || true'
        }

        success {
            echo "======================================"
            echo "Deployment successful"
            echo "Build: ${BUILD_NUMBER}"
            echo "Backend: ${DOCKER_IMAGE_BACKEND}:${IMAGE_TAG}"
            echo "Frontend: ${DOCKER_IMAGE_FRONTEND}:${IMAGE_TAG}"
            echo "======================================"
        }

        failure {
            echo "Build or deployment failed."
        }
    }
}