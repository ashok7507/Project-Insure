pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ashok7507/newinsure:latest"
        DOCKER_CREDENTIALS_ID = 'dockerhub-cred'
        DOCKER_USER = 'ashok7507'
        DOCKER_PASS = 'ashok-bhosale7#7507$@'
        GIT_REPO_URL = 'https://github.com/ashok7507/Project-Insure.git'
        GIT_CREDENTIALS_ID = 'github-cred'
        KUBE_CREDENTIALS_ID = 'k8s-creds'
        KUBE_NAMESPACE = '' // Set your Kubernetes namespace here
        EMAIL_RECIPIENT = 'aashok.sbhosale@gmail.com'
    }

    stages {
        stage('Code Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        url: "${GIT_REPO_URL}"
                    ]]
                )
            }
        }

        stage('Code Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Containerize the Application') {
            steps {
                echo 'Creating Docker image'
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: "${DOCKER_USER}",
                    passwordVariable: "$DOCKER_PASS"
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Code Deploy with Ansible') {
            steps {
                ansiblePlaybook(
                    credentialsId: 'ansible',
                    installation: 'ansible',
                    playbook: 'playbook.yml',
                    vaultTmpPath: ''
                )
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                withKubeConfig([credentialsId: "${KUBE_CREDENTIALS_ID}", namespace: "${KUBE_NAMESPACE}"]) {
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
            emailext(
                body: "Build Failed: ${env.BUILD_URL}",
                subject: "CI/CD Pipeline Failure - Build #${env.BUILD_NUMBER}",
                to: "${EMAIL_RECIPIENT}"
            )
        }
    }
}
