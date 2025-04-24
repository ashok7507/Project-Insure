pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "ashok7507/project-insure:${env.BUILD_NUMBER}"
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
                    docker.build(DOCKER_IMAGE).push()
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

