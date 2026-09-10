pipeline {
    agent any
    environment {
        DOCKER_USER = "idanasherov"
        IMAGE_NAME = "${DOCKER_USER}/holiday-events"
        CREDS_ID = "docker-hub-credentials"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest ."
            }
        }
        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${CREDS_ID}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER_ID')]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER_ID} --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }
    }
}
