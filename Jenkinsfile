pipeline {
    agent any
    environment {
        DOCKER_CREDS = credentials('dockerhub-credentials-id')
        IMAGE_NAME = 'afif729/lms-portal'
        TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Clone Code') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build("${IMAGE_NAME}:${TAG}", ".")
                    docker.image("${IMAGE_NAME}:${TAG}").tag("latest")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
                        app.push("${TAG}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Apply Kubernetes deployment with version tag update
                    sh "sed -i 's|image:.*|image: ${IMAGE_NAME}:${TAG}|g' deployment.yaml"
                    sh "kubectl apply -f deployment.yaml"
                }
            }
        }
        stage('Health Check & Rollback Verification') {
            steps {
                script {
                    try {
                        // Verify that pods roll out successfully
                        sh "kubectl rollout status deployment/lms-portal-deployment --timeout=60s"
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        echo "Deployment failed! Initiating automatic rollback..."
                        sh "kubectl rollout undo deployment/lms-portal-deployment"
                        error("Rollout failed and was automatically reverted to the previous stable release.")
                    }
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline executed successfully! Application updated and verified."
        }
        failure {
            echo "Pipeline failed. Rollback procedure executed."
        }
    }
}
