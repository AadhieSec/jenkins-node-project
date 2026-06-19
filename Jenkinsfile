pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        IMAGE_NAME = "aadhiesec/node-app"
        IMAGE_TAG = "latest"

        EC2_HOST = "13.50.105.72"
        EC2_USER = "ubuntu"

        APP_PORT = "3000"
    }

    tools {
        nodejs 'NodeJS'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build \
                      --platform linux/amd64 \
                      -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-key']) {
                    sh """
ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
set -e

echo "Pulling latest image..."
docker pull ${IMAGE_NAME}:${IMAGE_TAG}

echo "Stopping old container..."
docker stop node-app || true
docker rm node-app || true

echo "Starting new container..."
docker run -d \
    --name node-app \
    --restart unless-stopped \
    -p ${APP_PORT}:3000 \
    ${IMAGE_NAME}:${IMAGE_TAG}

echo "Cleaning unused images..."
docker image prune -f

echo ""
echo "Running containers:"
docker ps

sleep 5

echo ""
echo "===================================="
echo "Deployment Successful!"
echo "Application URL:"
echo "http://${EC2_HOST}:${APP_PORT}"
echo "===================================="

EOF
"""
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
            echo "🌐 Application URL: http://${EC2_HOST}:${APP_PORT}"
        }

        failure {
            echo "❌ Deployment Failed!"
        }

        always {
            cleanWs()
        }
    }
}
