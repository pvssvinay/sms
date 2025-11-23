pipeline {
    agent any

    environment {
        APP_NAME        = "sms"
        APP_NAMESPACE   = "sms-namespace"
        IMAGE_NAME      = "smsimg"
        IMAGE_TAG       = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/pvssvinay/sms'
            }
        }

        stage('Cleanup Old Container (if exists)') {
            steps {
                bat """
                    for /f "tokens=*" %%i in ('docker ps -aq -f name=%APP_NAME%') do (
                        echo Stopping container...
                        docker stop %APP_NAME%
                        echo Removing container...
                        docker rm %APP_NAME%
                    )
                    echo No old container found OR cleanup complete.
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE_NAME%:%IMAGE_TAG% -f Dockerfile ."
            }
        }

        stage('K8s Deployment') {
            steps {
                script {

                        echo "Applying Kubernetes manifests..."

                        bat "kubectl apply -f namespace.yaml --validate=false"
                        bat "kubectl apply -f deployment.yaml --validate=false"
                        bat "kubectl apply -f service.yaml --validate=false"

                        bat "kubectl rollout status deployment/sms-deployment -n sms-namespace"
                }
            }
        }
    }

    post {
        success {
            echo "✔ Completed: Build and Deploy on Windows"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}