pipeline {
    agent any
    tools {
        nodejs 'nodejs'
    }
  environment {
        SONAR_TOKEN = credentials('sonar-token') // Jenkins credential ID
        SONAR_HOST_URL = '${SONAR_HOST_URL}'
        SONAR_SCANER_HOME= tool 'SonarQube'
    }
    stages {
        stage('Git Checkout') {
            steps {
                cleanWs()
                git branch: 'develop', url: 'https://github.com/Shopping-App-Services/payment-service.git'
            }
        }
               stage('Dependencies'){
            steps{
                nodejs('nodejs') {
                    sh 'npm install @grpc/grpc-js'
                    sh 'npm install'
            }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                     sh '''
                    ${SONAR_SCANER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=PaymentService \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=. \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.verbose=true
                    '''
                    }  
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    // This step should not normally be used in your script. Consult the inline help for details.
                  withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                        sh 'ls -latr'
                        sh "docker build -t payment-service ."
                        sh "docker tag payment-service nitesh2611/payment-service:latest "
                        sh "docker push nitesh2611/payment-service:latest "
                    }
                }
            }
        }
    }
}


