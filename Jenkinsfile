pipeline {
    agent any

    environment {
        IMAGE_NAME = 'expense-tracker'
        CONTAINER_NAME = 'expense-tracker-app'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo 'GitHub source code downloaded'
            }
        }

        stage('Python Version') {
            steps {
                sh '''
                    docker --version
                    python3 --version || true
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                withCredentials([
                    string(credentialsId: 'expense-db-uri',
                           variable: 'DB_URI'),
                    string(credentialsId: 'expense-secret-key',
                           variable: 'SECRET_KEY')
                ]) {
                    sh '''
                        docker rm -f ${CONTAINER_NAME} || true

                        docker run -d \
                          --name ${CONTAINER_NAME} \
                          --restart unless-stopped \
                          -p 5000:5000 \
                          -e DB_URI \
                          -e SECRET_KEY \
                          ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 5
                    docker ps
                    curl --fail http://localhost:5000/ || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'Expense Tracker deployed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Jenkins console output.'
        }
    }
}
