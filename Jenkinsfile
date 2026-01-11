pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'bhauvana'
        DOCKER_CREDS   = credentials('DOCKER_HUB')
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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube1') {
                sh "sonar-scanner -Dsonar.projectKey=ept-dashboard -Dsonar.sources=src"


                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
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
                sh 'docker build -t $DOCKERHUB_USER/ept-dashboard:latest .'
            }
        }

        stage('Docker Login') {
            steps {
                sh 'echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $DOCKERHUB_USER/ept-dashboard:latest'
            }
        }

        stage('Deploy on Application EC2') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no ubuntu@40.192.119.196 '
                    docker pull $DOCKERHUB_USER/ept-dashboard:latest
                    docker stop ept-dashboard || true
                    docker rm ept-dashboard || true
                    docker run -d \
                        --name ept-dashboard \
                        -p 3000:80 \
                        $DOCKERHUB_USER/ept-dashboard:latest
                '
                """
            }
        }
    }
}      
      
            
