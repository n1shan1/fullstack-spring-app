pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        EC2_HOST = "ec2-3-80-177-47.compute-1.amazonaws.com"
        EC2_USER = "ubuntu"
        APP_DIR = "/home/ubuntu/app"
        KEY_CREDENTIAL_ID = "ec2-ssh-key"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Cloning repository from GitHub..."
                checkout scm
            }
        }

        stage('Build Backend Docker Image') {
            agent {
                docker {
                    image 'docker:latest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo "Building backend Docker image..."
                dir('spring-boot-server') {
                    sh '''
                        docker build -t ${DOCKERHUB_CREDENTIALS_USR}/backend:latest .
                    '''
                }
            }
        }

        stage('Build Frontend Docker Image') {
            agent {
                docker {
                    image 'docker:latest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo "Building frontend Docker image..."
                dir('angular-17-client') {
                    sh '''
                        docker build -t ${DOCKERHUB_CREDENTIALS_USR}/frontend:latest .
                    '''
                }
            }
        }

        stage('Push Images to DockerHub') {
            agent {
                docker {
                    image 'docker:latest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo "Logging in and pushing images to DockerHub..."
                sh '''
                    echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                    docker push ${DOCKERHUB_CREDENTIALS_USR}/backend:latest
                    docker push ${DOCKERHUB_CREDENTIALS_USR}/frontend:latest
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "Deploying to EC2 instance..."
                sshagent([env.KEY_CREDENTIAL_ID]) {
                    sh '''
                        scp -o StrictHostKeyChecking=no docker-compose.yml ${EC2_USER}@${EC2_HOST}:${APP_DIR}/
                    '''

                    sh '''
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                            cd ${APP_DIR}
                            echo "🧹 Cleaning old containers..."
                            docker compose down
                            echo "Pulling latest images..."
                            docker pull ${DOCKERHUB_CREDENTIALS_USR}/backend:latest
                            docker pull ${DOCKERHUB_CREDENTIALS_USR}/frontend:latest
                            echo "Starting updated containers..."
                            docker compose up -d
                            exit
                            EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Application running on EC2."
        }
        failure {
            echo "❌ Deployment failed. Please check logs."
        }
    }
}
