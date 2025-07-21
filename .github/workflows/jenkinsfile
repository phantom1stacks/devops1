pipeline {
    agent any

    environment {
        DOCKER_HOST = "unix:///var/run/docker.sock"
        DOCKER_IMAGE = "prity/my-web-app"
        DOCKER_TAG = "${env.BUILD_ID ?: 'latest'}"
        CONTAINER_NAME = "my-web-app-${env.BUILD_NUMBER}"
        DEPLOYMENT_URL = "http://localhost:8088"
        HOST_PORT = "8088"
        GOOGLE_CHAT_WEBHOOK = credentials('google-chat-webhook') // 🔐 stored securely in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/prity/html-demo.git'
                // ❌ No credentialsId needed if public
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh """
                            docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        # Stop and remove container using the same port
                        EXISTING_CONTAINER=\$(docker ps --format '{{.Names}}' --filter "publish=${HOST_PORT}" | head -n 1)
                        if [ -n "\$EXISTING_CONTAINER" ]; then
                            echo "Stopping existing container on port ${HOST_PORT}: \$EXISTING_CONTAINER"
                            docker stop \$EXISTING_CONTAINER || true
                            docker rm \$EXISTING_CONTAINER || true
                        fi

                        # Remove old container with same name
                        if docker ps -a --format '{{.Names}}' | grep -w ${CONTAINER_NAME}; then
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                        fi

                        # Deploy new container
                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            -p ${HOST_PORT}:80 \
                            ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
    }

    post {
        always {
            sh """
                docker logout || true
            """
        }

        success {
            script {
                def message = """
                🚀 *Deployment Successful* 
                *Build*: #${env.BUILD_NUMBER}
                *Image*: ${DOCKER_IMAGE}:${DOCKER_TAG}
                *URL*: ${DEPLOYMENT_URL}
                """
                sendGoogleChatNotification(message)
            }
        }

        failure {
            script {
                def logs = sh(
                    script: "docker logs --tail 50 ${CONTAINER_NAME} 2>&1 || echo 'No logs available'",
                    returnStdout: true
                ).trim()

                def message = """
                🔴 *Deployment Failed*
                *Build*: #${env.BUILD_NUMBER}
                *Error*: ${currentBuild.currentResult}
                *Logs*:
                ${logs}
                """
                sendGoogleChatNotification(message)
            }
        }
    }
}

// Function to send message to Google Chat
def sendGoogleChatNotification(String message) {
    def payload = """{ "text": "${message.replace('"', '\\"').replace('\n', '\\n')}" }"""
    sh """
        curl -X POST -H 'Content-Type: application/json' \
        -d '${payload}' \
        '${GOOGLE_CHAT_WEBHOOK}' || echo "Notification failed"
    """
}
