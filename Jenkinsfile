pipeline {
    agent any
    tools {
        // nodejs 'node12'
        jdk 'jdk17'
    }
    environment {         
        SCANNER_HOME = tool 'sonar-scanner'        
        DOCKERHUB_CREDENTIALS = credentials('716274dc-f41e-4e10-9f58-f501c9063a39')
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()                
            }
        }
        stage('Git Checkout') {
            steps {               
                git branch: 'main', url: 'https://github.com/Godhasdoneit/Amazonproject.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {                
                withSonarQubeEnv('sonar-server') {                    
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=amazon-app-deployment -Dsonar.projectName=amazon-app-deployment"
                    sh "pwd"
                }
            }
        }
        stage('quality gate') {
            steps {
              script {
                 withSonarQubeEnv('sonar-server') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
              }
            }           
        }
        stage('Install Dependencies') {
            steps {
              sh "npm install"
            }
        }
        stage('Login to DockerHUB'){
            steps{
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                echo "login succeeded"
            }
        }
        stage('Trivy File Scan'){
            steps{
                script{
                    sh 'trivy fs . > trivy_result.txt'
                }
            }
        }
        stage('Docker Build'){
            steps{
                script{                   
                   sh "docker build -t blesseddocker/amazon-app:latest ."
                   echo "Image Build Successfully"
                }
            }
        }
        stage('Trivy Image Scan'){
            steps{
                script{                    
                    sh "trivy image blesseddocker/amazon-app:latest"
                }
            }
        }
        stage('Docker push'){
            steps{
                script{                    
                    sh "docker push blesseddocker/amazon-app:latest"
                    echo "Push Image to Registry"
                }
            }
        }       
    }
}
