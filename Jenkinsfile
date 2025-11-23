pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/pvssvinay/sms'
            }
        }

        stage('Build JAR') {
            steps {
                echo "Building Spring Boot JAR..."
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker Image..."
                sh 'docker build -t smsimg:latest .'
            }
        }

        stage('Push to Local Docker Registry (Optional)') {
            when { expression { return false } }
            steps {
                echo "Skipping push — using local image for Kubernetes"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."

                sh '''
                    kubectl apply -f namespace.yaml
                    kubectl apply -f deployement.yaml
                    kubectl apply -f service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "kubectl get pods -n sms-namespace"
                sh "kubectl get svc -n sms-namespace"
            }
        }
    }

    post {
        success {
            echo "Deployment Successful! Access at: http://localhost:30080/"
        }
        failure {
            echo "Deployment Failed!"
        }
    }
}
