pipeline {
    agent any

    environment {
        IMAGE_NAME = "sohail28/python-web-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:${TAG} .
                    docker tag ${IMAGE_NAME}:${TAG} ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Run Tests Inside Container') {
            steps {
                script {
                    sh """
                    docker run --rm ${IMAGE_NAME}:${TAG} pytest tests/
                    docker run --rm ${IMAGE_NAME}:${TAG} flake8 app/
                    """
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${TAG}
                        docker push ${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Enterprise Pipeline Completed Successfully"
        }
        failure {
            echo "❌ Pipeline Failed - Inspect Stage Logs"
        }
    }
}

