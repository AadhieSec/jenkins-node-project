pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo "Source code checked out from GitHub"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Application Test') {
            steps {
                sh 'npm test'
            }
        }

    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
