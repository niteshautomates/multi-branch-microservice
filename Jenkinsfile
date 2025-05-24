pipeline {
    agent any
    tools {
        go 'Go 1.24.3'
    }
  environment {
        SONAR_TOKEN = credentials('sonar-token') // Jenkins credential ID
        SONAR_HOST_URL = '${SONAR_HOST_URL}'
        SONAR_SCANER_HOME= tool 'SonarQube'
        GOCACHE = "${WORKSPACE}/.go-cache"
    }
    stages {
        stage('Git Checkout') {
            steps {
                cleanWs()
                git branch: 'develop', url: 'https://github.com/Shopping-App-Services/frontend-service.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                     sh '''
                    ${SONAR_SCANER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=FrontendService \
                        -Dsonar.sources=. \
                        -Dsonar.language=go \
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
        stage('Test') {
            steps {
              
                sh 'go test ./...'
            }
        }
         stage('Build') {
            steps {
                sh 'go version'
                sh 'go build -o ./...'
                
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                    // This step should not normally be used in your script. Consult the inline help for details.
                  withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                        sh 'ls -latr'
                        sh "docker build -t frontend-service ."
                        sh "docker tag frontend-service nitesh2611/frontend-service:latest "
                        sh "docker push nitesh2611/frontend-service:latest "
                    }
                }
            }
        }
    }
}


