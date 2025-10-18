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
                stash includes: 'docker-compose.yaml', name: 'docker-compose'
            }
        }

        stage('Build Backend Docker Image') {
            agent {
                docker {
                    image 'docker:latest'
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo "Building backend Docker image..."
                dir('spring-boot-server') {
                    sh '''
                        echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                        docker buildx create --use --name multiarch-builder --driver docker-container --driver-opt network=host || docker buildx use multiarch-builder
                        docker buildx build --platform linux/amd64,linux/arm64 -t ${DOCKERHUB_CREDENTIALS_USR}/backend:latest --push .
                    '''
                }
            }
        }

        stage('Build Frontend Docker Image') {
            agent {
                docker {
                    image 'docker:latest'
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo "Building frontend Docker image..."
                dir('angular-17-client') {
                    sh '''
                        echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                        docker buildx create --use --name multiarch-builder-frontend --driver docker-container --driver-opt network=host || docker buildx use multiarch-builder-frontend
                        docker buildx build --platform linux/amd64,linux/arm64 -t ${DOCKERHUB_CREDENTIALS_USR}/frontend:latest --push .
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "Deploying to EC2 instance..."
                unstash 'docker-compose'
                sshagent([env.KEY_CREDENTIAL_ID]) {
                    sh '''
                        scp -o StrictHostKeyChecking=no docker-compose.yaml ${EC2_USER}@${EC2_HOST}:${APP_DIR}/
                    '''

                    sh '''
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                            cd ${APP_DIR}
                            echo "🧹 Cleaning old containers..."
                            sudo docker compose down
                            echo "Pulling latest images..."
                            sudo docker pull ${DOCKERHUB_CREDENTIALS_USR}/backend:latest
                            sudo docker pull ${DOCKERHUB_CREDENTIALS_USR}/frontend:latest
                            echo "Starting updated containers..."
                            sudo docker compose up -d
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
