pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'bhauvana'
        IMAGE_NAME = 'ept-dashboard'
        DOCKER_CREDS = credentials('DOCKER_HUB')
        EMAIL_TO = 'bhuvaneswari.k002@gmail.com'
        APP_EC2_IP = '54.183.131.143'
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

        stage('Run Tests') {
            steps {
                sh 'npm test -- --watchAll=false'
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t bhauvana/ept-dashboard:latest .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                sh '''
                    echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push bhauvana/ept-dashboard:latest
                '''
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
                    ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no "$SSH_USER"@54.183.131.143 << EOF
                        docker pull ${DOCKERHUB_USER}/ept-dashboard:latest
                        docker stop ept-dashboard || true
                        docker rm ept-dashboard || true
                        docker run -d \
                          --name ept-dashboard \
                          -p 3000:80 \
                          ${DOCKERHUB_USER}/ept-dashboard:latest
                    EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: "bhuvaneswari.k002@gmail.com",
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                <h2>Build Successful</h2>
                <p>Job: ${env.JOB_NAME}</p>
                <p>Build Number: ${env.BUILD_NUMBER}</p>
                <p>URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }

        failure {
            emailext(
                to: "bhuvaneswari.k002@gmail.com",
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                <h2>Build Failed</h2>
                <p>Job: ${env.JOB_NAME}</p>
                <p>Build Number: ${env.BUILD_NUMBER}</p>
                <p>Check logs: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }
    }
}
