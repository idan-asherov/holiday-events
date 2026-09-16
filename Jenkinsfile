pipeline {
    agent any

    environment {
        IMAGE_NAME         = "holiday-events"
        TELEGRAM_BOT_TOKEN = "8553298889:AAFmWYcGPwiU5aoviwVJ5aiJgMPPAUAard8"
        TELEGRAM_CHAT_ID   = "6839491229"
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

        stage('Deploy Container') {
            steps {
                sh "docker stop holiday-app || true"
                sh "docker rm holiday-app || true"
                sh "docker run -d --name holiday-app -p 8000:3000 ${IMAGE_NAME}:latest"
            }
        }

        stage('Health Check') {
            steps {
                sleep 3
                sh "curl -f http://host.docker.internal:8000/health"
            }
        }
    }

    post {
        success {
            sh '''
                docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:stable-backup || true

                curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
                    -d "chat_id=${TELEGRAM_CHAT_ID}" \
                    -d "text=✅ Pipeline Succeeded!%0AProject: Holiday Events%0ABuild: #${BUILD_NUMBER}%0AStatus: App is Healthy!"
            '''
        }
        failure {
            sh '''
                echo "⚠️ Health Check or Deployment failed! Initiating Automatic Rollback..."
                
                if docker image inspect ${IMAGE_NAME}:stable-backup > /dev/null 2>&1; then
                    docker stop holiday-app || true
                    docker rm holiday-app || true
                    docker run -d --name holiday-app -p 8000:3000 ${IMAGE_NAME}:stable-backup
                    ROLLBACK_MSG="%0A🔄 Rollback executed: Restored previous stable version."
                else
                    ROLLBACK_MSG="%0A⚠️ No previous stable version found for rollback."
                fi

                curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
                    -d "chat_id=${TELEGRAM_CHAT_ID}" \
                    -d "text=❌ Pipeline Failed!%0AProject: Holiday Events%0ABuild: #${BUILD_NUMBER}${ROLLBACK_MSG}%0APlease inspect Jenkins logs."
            '''
        }
    }
}