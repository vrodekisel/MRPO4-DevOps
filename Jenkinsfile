
pipeline {
    agent any
    
    environment {
        PROJECT_NAME = 'MRPO4-DevOps'
        BUILD_MODE = 'training'
    }

    stages {
        stage('Build') {
            steps {    
            echo "Проект: ${env.PROJECT_NAME}, режим: ${env.BUILD_MODE}, сборка №${env.BUILD_NUMBER}"

                echo 'Сборка приложения'
                sh 'mkdir -p build && cp index.html build/index.html'
                archiveArtifacts artifacts: 'build/index.html'
            }
        }

        
    stage('Test') {
        steps {
            echo 'Тестирование приложения'

            sh 'test -s build/index.html'
            sh 'grep -q "MRPO4 DevOps" build/index.html'
            sh 'test "$BUILD_MODE" = "training"'

            withCredentials([string(
                credentialsId: 'mrpo4-demo-token',
                variable: 'DEMO_TOKEN'
            )]) {
                sh 'test -n "$DEMO_TOKEN"'
                echo 'Проверка Jenkins Credentials успешна'
            }
        }
    }


        stage('Deploy to Staging') {
            steps {
                echo 'Развертывание в тестовой среде'
                sh 'mkdir -p staging && cp build/index.html staging/index.html'
            }
        }

        stage('Deploy to Production') {
            steps {
                echo 'Развертывание в рабочей среде'
                sh 'mkdir -p production && cp staging/index.html production/index.html'
            }
        }
    }

    
    post {
        success {
            echo 'Конвейер успешно выполнен'
        }

        failure {
            echo 'Ошибка выполнения конвейера'

            withCredentials([string(
                credentialsId: 'mrpo4-notification-email',
                variable: 'NOTIFY_EMAIL'
            )]) {
                mail(
                    to: env.NOTIFY_EMAIL,
                    subject: "Ошибка сборки №${env.BUILD_NUMBER}: ${env.JOB_NAME}",
                    body: """Сборка Jenkins завершилась ошибкой.

    Проект: ${env.JOB_NAME}
    Номер сборки: ${env.BUILD_NUMBER}
    Статус: FAILURE

    Подробности: ${env.BUILD_URL}
    """
                )
            }
        }
    }

}
