pipeline {
    agent any

    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        DOCKER_REPO = "sohail28/python-web-app"
        DOCKER_CREDENTIALS = "docker-hub-creds"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies & Test') {
            steps {
                script {
                    docker.image('python:3.11-slim').inside {
                        sh '''
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        pip install pytest flake8

                        echo "Running Unit Tests..."
                        pytest tests/

                        echo "Running Code Quality Checks..."
                        flake8 app/
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_REPO}:${IMAGE_TAG} .
                    docker tag ${DOCKER_REPO}:${IMAGE_TAG} ${DOCKER_REPO}:latest
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        sh """
                        docker push ${DOCKER_REPO}:${IMAGE_TAG}
                        docker push ${DOCKER_REPO}:latest
                        """
                    }
                }
            }
        }

    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "✅ Enterprise CI/CD Pipeline Completed Successfully"
        }
        failure {
            echo "❌ Pipeline Failed - Please Check Logs"
        }
    }
}

