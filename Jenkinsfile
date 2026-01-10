pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'bhauvana'
        IMAGE_NAME = 'ept-dashboard'
        DOCKER_CREDS = credentials('DOCKER_HUB')
        EMAIL_TO = 'kbhuvaneswari474@gmail.com'
        APP_EC2_IP = '54.183.131.143'

        SONAR_HOST_URL = 'http://54.183.107.136:9000'
        SONAR_SCANNER_HOME = tool 'SonarScanner'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests with Coverage') {
            steps {
                sh 'npm test -- --coverage --watchAll=false'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    sonar-scanner \
                    -Dsonar.projectKey=ept-dashboard \
                    -Dsonar.projectName=ept-dashboard \
                    -Dsonar.sources=src \
                    -Dsonar.exclusions=node_modules/**,build/** \
                    -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                    -Dsonar.host.url=${SONAR_HOST_URL}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('Docker Login') {
            steps {
                sh """
                echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy on Application EC2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'EC2_SSH_KEY',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                    ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no "$SSH_USER"@${APP_EC2_IP} << 'ENDSSH'
                    set -e
                    docker pull bhauvana/ept-dashboard:latest
                    docker stop ept-dashboard || true
                    docker rm ept-dashboard || true
                    docker run -d --name ept-dashboard -p 3000:80 bhauvana/ept-dashboard:latest
                    ENDSSH
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: "kbhuvaneswari474@gmail.com",
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build Successful - ${env.BUILD_URL}"
            )
        }
        failure {
            emailext(
                to: "kbhuvaneswari474@gmail.com",
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build Failed - ${env.BUILD_URL}"
            )
        }
    }
}
