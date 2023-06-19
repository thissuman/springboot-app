pipeline {
    agent any
    
    tools{
        maven 'Maven3'
    }
    environment {
        registry = "433581047482.dkr.ecr.us-east-2.amazonaws.com/jenkins-suman00"
        
    }
    

    stages {
        stage('checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/thissuman/springboot-app.git']])
            }
        }
         stage('build jar') {
            steps {
                sh "mvn clean install"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    script {
                        def scannerHome = tool 'sonar4.8'
                        withEnv(["SONAR_USER_HOME=${env.WORKSPACE}/.sonar"]) {
                            sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=ba2d8119e478328dbc3d1620b51100ab98940414 -Dsonar.exclusions=**/*.java"

                            
                        }
                        
                    }
                    
                }
                
            }
            
        }
        
        
         stage('build image'){
            steps{
                script {
                    docker.build registry
                }
            }
        }
        stage('push into registry'){
            steps{
                script {
                    sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 433581047482.dkr.ecr.us-east-2.amazonaws.com'
                    sh 'docker push 433581047482.dkr.ecr.us-east-2.amazonaws.com/jenkins-suman00:latest'
                }
            }
        }
        stage('Deploy on EKS'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                    sh 'kubectl apply -f eks-deploy-k8s.yaml'
                }
            }
        }
    }
}
