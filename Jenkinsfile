pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        IMAGE_NAME = "aadhiesec/node-app"
        IMAGE_TAG = "latest"
        EC2_HOST = "13.50.105.72"
        EC2_USER = "ubuntu"
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
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
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

docker pull ${IMAGE_NAME}:${IMAGE_TAG}

docker stop node-app || true
docker rm node-app || true

docker run -d \
    --name node-app \
    --restart unless-stopped \
    -p 3000:3000 \
    ${IMAGE_NAME}:${IMAGE_TAG}

docker image prune -f

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

        always {
            cleanWs()
        }
    }
}
