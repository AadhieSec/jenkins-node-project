pipeline {
    agent any

    environment {
	DOCKER = "/usr/local/bin/docker"
        IMAGE_NAME = "aadhiesec/node-app"
        IMAGE_TAG = "latest"
        EC2_HOST = "13.50.105.72"
        EC2_USER = "ubuntu"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-credentials', toolName: 'docker') {
                    sh """
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF

                    docker pull ${IMAGE_NAME}:${IMAGE_TAG}

                    docker stop node-app || true

                    docker rm node-app || true

                    docker run -d \
                        --name node-app \
                        --restart unless-stopped \
                        -p 3000:3000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }

        failure {
            echo '❌ Deployment Failed!'
        }
    }
}
