pipeline {
    agent any
    environment {
        DOCKER_CREDS = credentials('dockerhub')
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
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            bat "powershell -Command \"$pass = '%DOCKER_PASS%'; $user = '%DOCKER_USER%'; $sec = ConvertTo-SecureString $pass -AsPlainText -Force; $BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($sec); $Plain = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR); & docker login -u $user --password-string $Plain\""
            bat "docker push afif729/lms-portal:latest"
        }
    }
}
        stage('Deploy to Kubernetes') {
            steps {
                bat "kubectl apply -f deployment.yaml"
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
