pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'niishantdev'
        DOCKER_HUB_PASS = credentials('dockerhub')
        ANSIBLE_DIR = '/home/ubuntu/ansible'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                    echo "Building Docker images..."
                    docker build -t $DOCKER_HUB_USER/backend:latest ./spring-boot-server
                    docker build -t $DOCKER_HUB_USER/frontend:latest ./angular-17-client
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh '''
                    echo $DOCKER_HUB_PASS | docker login -u $DOCKER_HUB_USER --password-stdin
                    docker push $DOCKER_HUB_USER/backend:latest
                    docker push $DOCKER_HUB_USER/frontend:latest
                    '''
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                script {
                    sh """
                    cd ${ANSIBLE_DIR}
                    ansible-playbook -i inventory.ini site.yml
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Deployment failed. Check logs in Jenkins.'
        }
    }
}
