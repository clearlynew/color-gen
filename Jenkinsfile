pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'devops-frontend'
        DOCKER_CREDS_ID = 'dockerhub-credentials'
        DOCKER_HUB_USER = 'himabcs86'
        TAG = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image: ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${TAG}"
                    sh """
                    docker build -t ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${TAG} \
                                 -t ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:latest .
                    """
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    echo "Testing container..."
                    sh """
                    docker run -d --name temp-test-${TAG} -p 8085:80 ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${TAG}
                    sleep 5
                    docker ps
                    docker stop temp-test-${TAG}
                    docker rm temp-test-${TAG}
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing image to Docker Hub"

                    withCredentials([usernamePassword(
                        credentialsId: DOCKER_CREDS_ID,
                        usernameVariable: 'USERNAME',
                        passwordVariable: 'PASSWORD'
                    )]) {

                        sh """
                        echo \$PASSWORD | docker login -u \$USERNAME --password-stdin
                        docker push ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${TAG}
                        docker push ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:latest
                        docker logout
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"

            cleanWs()

            sh "docker rmi ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${TAG} || true"
            sh "docker rmi ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:latest || true"
        }

        success {
            echo "Build and Push successful"
        }

        failure {
            echo "Pipeline failed"
        }
    }
}
