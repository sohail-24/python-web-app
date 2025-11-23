pipeline {
    agent any

    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        DOCKER_REPO = 'sohail28/python-web-app'
        DOCKER_CREDENTIALS = 'docker-hub-creds'
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
                venv/bin/pip install --upgrade pip
                venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '''
                venv/bin/pip install pytest
                venv/bin/pytest tests/
                '''
            }
        }

        stage('Static Code Analysis') {
            steps {
                sh '''
                venv/bin/pip install flake8
                venv/bin/flake8 app/
                '''
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def tag = BUILD_NUMBER
                    def image = "${DOCKER_REPO}:${tag}"
                    env.DEPLOY_IMAGE = image
                    docker.build(image)
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        docker.image(env.DEPLOY_IMAGE).push()
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
            echo "✅ Python CI/CD Pipeline Completed Successfully"
        }
        failure {
            echo "❌ Pipeline Failed - Fix Required"
        }
    }
}

