pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "ashok7507/project-insure:${env.BUILD_NUMBER}"
        IMAGE_NAME = 'ashok7507/project-insure'
        IMAGE_TAG = '3'
        KUBE_NAMESPACE = "insure-app"
        REPO_URL = "https://github.com/ashok7507/Project-Insure.git"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: env.REPO_URL,
                credentialsId: 'github-cred'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'  
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                    def app = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    app.push()
                }
            }
        }
        
        stage('K8s Deployment') {
            steps {
                withKubeConfig([credentialsId: 'k8s-creds', namespace: KUBE_NAMESPACE]) {
                    sh """
                    kubectl apply -f kubernetes/deployment.yaml
                    kubectl apply -f kubernetes/service.yaml
                    """
                }
            }
        }
    }
    
    post {
        failure {
            emailext body: 'Build Failed: ${BUILD_URL}',
                    subject: 'CI/CD Pipeline Failure',
                    to: 'your-email@example.com'
        }
    }
}

