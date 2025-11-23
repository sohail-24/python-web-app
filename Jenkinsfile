pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "sohail28/python-web-app"
        IMAGE_TAG   = "${BUILD_NUMBER}"
        DOCKER_CREDS = "docker-hub-creds"   // Jenkins credentials ID
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/sohail-24/python-web-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Run Tests Inside Container') {
            steps {
                script {
                    sh "docker run --rm ${DOCKER_IMAGE}:${IMAGE_TAG} pytest || true"
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    withDockerRegistry(
                        url: 'https://index.docker.io/v1/',
                        credentialsId: DOCKER_CREDS
                    ) {
                        sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline Completed Successfully'
        }
        failure {
            echo '❌ Pipeline Failed - Check Logs'
        }
    }
}
