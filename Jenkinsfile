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
                // תיוג גרסה נוכחית, תיוג latest, ושמירת תגית rollback
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh "docker compose down || true"
                sh "docker compose up -d"
            }
        }

        stage('Health Check') {
            steps {
                sleep 3
                // בדיקת בריאות קפדנית - נכשל מיד אם אין HTTP 200
                sh "curl -f http://host.docker.internal:8000/health"
            }
        }
    }

    post {
        success {
            sh '''
                # שומרים עותק של הגרסה שעברה בהצלחה לצורך Rollback עתידי
                docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:stable-backup || true

                # שליחת התראה מוצלחת לטלגרם
                curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
                    -d "chat_id=${TELEGRAM_CHAT_ID}" \
                    -d "text=✅ Pipeline Succeeded!%0AProject: Holiday Events%0ABuild: #${BUILD_NUMBER}%0AStatus: App is Healthy!"
            '''
        }
        failure {
            sh '''
                echo "⚠️ Health Check or Build failed! Initiating Automatic Rollback..."
                
                # בדיקה האם קיים אימג' יציב קודם לביצוע Rollback
                if docker image inspect ${IMAGE_NAME}:stable-backup > /dev/null 2>&1; then
                    docker compose down || true
                    docker run -d --name holiday-app -p 8000:3000 ${IMAGE_NAME}:stable-backup
                    ROLLBACK_MSG="%0A🔄 Rollback executed: Restored previous stable version."
                else
                    ROLLBACK_MSG="%0A⚠️ No previous stable version found for rollback."
                fi

                # שליחת התראת כישלון לטלגרם עם סטטוס ה-Rollback
                curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
                    -d "chat_id=${TELEGRAM_CHAT_ID}" \
                    -d "text=❌ Pipeline Failed!%0AProject: Holiday Events%0ABuild: #${BUILD_NUMBER}${ROLLBACK_MSG}%0APlease inspect Jenkins logs."
            '''
        }
    }
}