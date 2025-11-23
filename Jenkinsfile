pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        APP_NAME = "python-web-app"
        DOCKER_IMAGE = "sohail28/python-web-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                  python3 -m venv venv
                  . venv/bin/activate
                  pip install --upgrade pip
                  pip install -r requirements.txt
                '''
            }
        }

        stage('Static Code Analysis') {
            steps {
                sh '''
                  . venv/bin/activate
                  pip install pylint bandit
                  pylint app.py || true
                  bandit -r . || true
                '''
            }
        }

        stage('Unit Testing') {
            steps {
                sh '''
                  . venv/bin/activate
                  pip install pytest
                  pytest
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                  docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                      docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Python CI/CD Pipeline Successful"
        }
        failure {
            echo "❌ Build Failed - Check Logs"
        }
        always {
            cleanWs()
        }
    }
}

