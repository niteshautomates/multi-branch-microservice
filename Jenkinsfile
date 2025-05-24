pipeline {
    agent any
    tools {
        dotnetsdk 'dotnet'
    }
    environment {
        SONAR_TOKEN = credentials('sonar-token') // Jenkins credential ID
        SONAR_HOST_URL = "${SONAR_HOST_URL}"
        SONAR_SCANER_HOME = tool 'SonarQube'
    }
    stages {
        stage('Git Checkout') {
            steps {
                cleanWs()
                git branch: 'develop', url: 'https://github.com/Shopping-App-Services/cart-service.git'
            }
        }

        stage('Build') {
            steps {
                dir('src') {
                    sh 'ls -latr'
               sh 'dotnet build cartservice.csproj'
                }
            }
        }
        stage('Test') {
            steps {
                dir('src') {
                sh 'dotnet test'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh '''
                    ${SONAR_SCANER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=CartService \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=. \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.verbose=true \
                        -Dsonar.sonar.cs.opencover.reportsPaths=\"./TestResults/coverage.opencover.xml\"
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
                        sh "docker build -t cart-service -f src/Dockerfile ."
                        sh "docker tag cart-service nitesh2611/cart-service:latest "
                        sh "docker push nitesh2611/cart-service:latest "
                    }
                }
            }
        }
    }
}

