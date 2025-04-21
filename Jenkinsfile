pipeline {
    agent any
    
    stages {
        stage('Code-Checkout') {
            steps {
               checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/ashok7507/Project-Insure.git']])
            }
        }
        
        stage('Code-Build') {
            steps {
               sh 'mvn clean package'
            }
        }
        
        stage('Containerize the application'){
            steps { 
               echo 'Creating Docker image'
               sh "docker build ashok7507/newinsure:latest "
            }
        }
        
        stage('Docker Push') {
    	agent any
          steps {
       	withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            	sh "docker login -u $dockerHubUser -p $dockerHubPassword"
                sh 'docker push ashok7507/newinsure:latest'
        }
      }
    }
    
      
     stage('Code-Deploy') {
        steps {
           ansiblePlaybook credentialsId: 'ansible', installation: 'ansible', playbook: 'playbook.yml', vaultTmpPath: ''       
        }
      }
   }
}

