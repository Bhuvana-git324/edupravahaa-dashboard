pipeline {
    agent any

    tools {
        nodejs 'NodeJS'          // must match Jenkins → Global Tool Configuration
    }

    environment {
        SONAR_SCANNER = tool 'SonarScanner'   // SonarQube Scanner name
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                  npm install
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                  npm run build
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonarqube') {
                    sh '''
                      ${SONAR_SCANNER}/bin/sonar-scanner \
                      -Dsonar.projectKey=sonarqube \
                      -Dsonar.projectName=sonarqube \
                      -Dsonar.projectVersion=1.0 \
                      -Dsonar.sources=src \
                      -Dsonar.exclusions=node_modules/**,build/**
                    '''
                }
            }
        }
    }
}

