pipeline {
    agent any
    tools{
        nodejs 'node21'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Arijit094/3-Tier-DevOps-CICD-Pipeline-Project.git'
            }
        }
        stage('install package dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Unit Tests') {
            steps {
                sh "npm test"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs  --format table -o fs-report.html ."
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"
                }
            }
        }
        stage('Docker Build & Tag') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-secret', toolName: 'docker') {
                    sh "docker build -t arijit094/campa:latest ."
                 }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image  --format table -o fs-report.html arijit094/campa:latest"
            }
        }
        stage('Docker push image') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker-secret', toolName: 'docker') {
                    sh "docker push  arijit094/campa:latest"
                  }
               }
            }
       }
        stage('Docker Deployment') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker-secret', toolName: 'docker') {
                    sh "docker run -d -p 3000:3000  arijit094/campa:latest"
                 }
            
                }
            }
        }
    }
}
