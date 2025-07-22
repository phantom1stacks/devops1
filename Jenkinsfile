pipeline {
    agent any

    environment {
        CONTAINER_NAME = 'my-container'  // Define your environment variable here
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
                // Add your actual build commands here
            }
        }

        // Optional stage: Test
        stage('Test') {
            steps {
                echo "Running tests..."
                sh 'echo "Tests passed!"'
                // Add your test commands here
            }
        }
    }

    post {
        always {
            node {
                // Ensure this runs in a workspace
                sh 'echo "Running post-build cleanup..."'
            }

            script {
                try {
                    // Google Chat notification (make sure plugin is installed and webhook is correct)
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
