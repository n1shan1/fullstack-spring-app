pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        EC2_KEY = credentials('jenkins-ec2-key')
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning repository from GitHub..."
                checkout scm
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('spring-boot-server') {
                    sh 'docker build -t niishantdev/backend:latest .'
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('angular-17-client') {
                    sh 'docker build -t niishantdev/frontend:latest .'
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push niishantdev/backend:latest
                    docker push niishantdev/frontend:latest
                    '''
                }
            }
        }

        stage('Deploy via Ansible') {
            steps {
                dir('/home/ubuntu/ansible-ws') {
                    sh 'ansible-playbook -i inventory.ini site.yml'
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
