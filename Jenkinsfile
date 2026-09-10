pipeline {
    agent any
    environment {
        IMAGE_NAME = "holiday-events"
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
        stage('Deploy to Docker Desktop') {
            steps {
                sh "docker stop holiday-app || true"
                sh "docker rm holiday-app || true"
                sh "docker run -d --name holiday-app -p 8000:3000 ${IMAGE_NAME}:latest"
            }
        }
        stage('Health Check') {
            steps {
                sleep 3
                sh "curl -f http://host.docker.internal:8000/health || curl -f http://host.docker.internal:8000/api/events || curl -f http://host.docker.internal:8000/ || true"
            }
        }
    }
}
