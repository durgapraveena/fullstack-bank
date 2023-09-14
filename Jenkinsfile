pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        nodejs 'node16'
        
    }
    
    environment{
        HOME = '/var/lib/jenkins'
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/durgapraveena/fullstack-bank.git'
            }
        }
        
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./app/backend --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('SONARQUBE ANALYSIS') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh " $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Bank -Dsonar.projectKey=Bank "
                }
            }
        }
        
        
         stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('Backend') {
            steps {
                dir('/var/lib/jenkins/workspace/Bank/app/backend') {
                    sh "npm install"
                }
            }
        }
        
        stage('frontend') {
            steps {
                dir('/var/lib/jenkins/workspace/Bank/app/frontend') {
                    sh "npm install"
                }
            }
        }
        
        stage('Deploy to Conatiner') {
            steps {
                script {    
                    docker.image('docker:20.10').inside('-v /var/lib/jenkins/workspace/Bank:/workspace') {
                        sh """
                            apt-get update
                            apt-get install -y docker-compose
                            cd /workspace/app
                            docker-compose up -d
                        """
                    }
                 }
            }
         }
    }
}
    
