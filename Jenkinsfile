pipeline {
    agent any

    environment {
        CONTAINER_NAME = 'my-container'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Starting build for container: ${env.CONTAINER_NAME}"
                sh 'echo "Building project..."'
            }
        }

        stage('Test') {
            steps {
                echo "Running tests..."
                sh 'echo "Tests passed!"'
            }
        }
    }

    post {
        always {
            // ✅ Removed node block
            sh 'echo "Running post-build cleanup..."'

            script {
                try {
                    googlechatSend message: "Build completed for ${env.JOB_NAME} (#${env.BUILD_NUMBER})",
                                   webhookUrl: 'https://chat.googleapis.com/your-webhook-url'
                } catch (err) {
                    echo "Google Chat notification failed: ${err}"
                }
            }
        }

        failure {
            script {
                echo "Build failed. Container: ${env.CONTAINER_NAME}"
            }
        }

        success {
            script {
                echo "Build succeeded. Container: ${env.CONTAINER_NAME}"
            }
        }
    }
}
