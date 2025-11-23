pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "sohail28/python-web-app"
        DOCKER_CREDS = "docker-hub-creds"
    }

    parameters {
        string(name: 'TAG', defaultValue: 'latest', description: 'Docker image tag')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
                sh "docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Run Tests Inside Container') {
            steps {
                sh "docker run --rm ${DOCKER_IMAGE}:${BUILD_NUMBER} pytest"
            }
        }

        stage('Push Image') {
            steps {
                withDockerRegistry([ credentialsId: DOCKER_CREDS ]) {
                    sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline Completed Successfully"
        }
        failure {
            echo "❌ Pipeline Failed - Check logs"
        }
        always {
            cleanWs()
        }
    }
}

