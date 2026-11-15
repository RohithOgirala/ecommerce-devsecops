pipeline {
    agent any
    environment {
        KUBECONFIG = '/var/jenkins_home/.kube/config'
        IMAGE = "shopeasy-app:b${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Build Image') { steps { sh 'docker build -t $IMAGE .' } }
        stage('Trivy Security Scan') {
            steps { sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity HIGH,CRITICAL $IMAGE' }
        }
        stage('Load Image to Cluster') {
            steps { sh 'docker save $IMAGE | docker exec -i desktop-control-plane ctr -n k8s.io images import -' }
        }
        stage('Deploy Green') {
            steps {
                sh 'kubectl apply -f k8s/'
                sh 'kubectl set image deployment/shopeasy-green shopeasy=$IMAGE'
                sh 'kubectl rollout status deployment/shopeasy-green --timeout=120s'
            }
        }
        stage('Switch Traffic to Green') {
            steps {
                sh 'kubectl set selector service shopeasy-service app=shopeasy,version=green'
                sh 'kubectl get pods,svc'
            }
        }
    }
    post {
        success { echo 'Deployed to Kubernetes - Blue/Green switch complete!' }
        failure { echo 'Pipeline FAILED' }
    }
}
