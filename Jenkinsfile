pipeline {
    agent any
    environment {
        IMAGE = "shopeasy-app"
        TAG   = "v${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build Image') {
            steps { sh 'docker build -t $IMAGE:$TAG -t $IMAGE:latest .' }
        }
        stage('Trivy Security Scan') {
            steps {
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity HIGH,CRITICAL --exit-code 0 $IMAGE:$TAG'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/blue-deployment.yaml'
                sh 'kubectl apply -f k8s/green-deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
                sh 'kubectl get pods,svc'
            }
        }
    }
    post {
        success { echo 'ShopEasy DevSecOps Pipeline SUCCESS!' }
        failure { echo 'Pipeline FAILED' }
    }
}
