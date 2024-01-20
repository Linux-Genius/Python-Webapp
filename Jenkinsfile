pipeline {
    agent any
    tools {
        jdk 'jdk17'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/Linux-Genius/Python-Webapp.git'
            }
        }
        
        stage('OWASP') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy Fs Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Python-Webapp \
                -Dsonar.projectKey=Python-Webapp'''
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "make image"
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image abhisharma1423/python-webapp:latest"
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "make push"
                    }
                }
            }
        }
        
        stage('Deploy to Container') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker run -d -p 5000:5000 abhisharma1423/python-webapp:latest"
                    }   
                }
            }
            
        }
        
    }
}
