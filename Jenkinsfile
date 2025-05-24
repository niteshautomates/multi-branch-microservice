pipeline {
    agent any

  environment {
        SONAR_TOKEN = credentials('sonar-token') // Jenkins credential ID
        SONAR_HOST_URL = '${SONAR_HOST_URL}'
        SONAR_SCANER_HOME= tool 'SonarQube'
    }
    stages {
        stage('Git Checkout') {
            steps {
                cleanWs()
                git branch: 'develop', url: 'https://github.com/Shopping-App-Services/email-service.git'
            }
        }
        stage('Setup Virtual Environment') {
            steps {
                sh '''
                # Remove any existing virtual environments
                #rm -rf venv
                # Create a new virtual environment
                #python3 -m venv venv
                #chmod -R 755 venv
                # Activate the virtual environment and install dependencies
                #. venv/bin/activate && \
                #pip install --upgrade pip && \
                pip install -r requirements.txt
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                     sh '''
                    ${SONAR_SCANER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EmailService \
                        -Dsonar.exclusions=venv/** \
                        -Dsonar.sources=. \
                        -Dsonar.python.coverage.reportPaths=coverage.xml \
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
                sh '''
                  # Activate the virtual environment and run tests
                #. venv/bin/activate && \
                pip install pytest && \
                pip install pytest-cov && \
                pytest --cov=app --cov-report=xml && \
                pytest --cov=app --cov-report=term-missing --disable-warnings
                '''
            }
        }
        stage('Build') {
            steps {
                sh '''
                # Remove any existing virtual environments
                #rm -rf venv
                # Create a new virtual environment
                #python3 -m venv venv
                #chmod -R 755 venv
                # Activate the virtual environment and install dependencies
                #. venv/bin/activate && \
                #pip install --upgrade pip && \
                pip install -r requirements.txt
            '''
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                    // This step should not normally be used in your script. Consult the inline help for details.
                   withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                        sh 'ls -latr'
                        sh "docker build -t email-service ."
                        sh "docker tag email-service nitesh2611/email-service:latest "
                        sh "docker push nitesh2611/email-service:latest "
                    }
                }
            }
        }
    }
}

