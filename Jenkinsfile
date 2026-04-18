pipeline {
    agent any

    environment {
        // Имя бинарного файла, который мы создадим
        BINARY_NAME = 'my-go-app'
        // Версия для Nexus (можно использовать номер сборки Jenkins)
        VERSION = "1.0.${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Репозиторий склонирован"
            }
        }

        stage('Run Go Tests') {
            steps {
                sh 'go test .'
                echo "Тесты пройдены"
            }
        }

        stage('Build Go Binary') {
            steps {
                // Эта команда должна быть из вашего Dockerfile
                // Убедитесь, что путь к исходникам правильный
                sh """
                    CGO_ENABLED=0 GOOS=linux go build -o ${BINARY_NAME} .
                """
                echo "Бинарный файл ${BINARY_NAME} собран"
            }
        }

        stage('Archive Artifact') {
            steps {
                // Сохраняем бинарный файл как артефакт Jenkins
                // Он будет доступен на странице сборки
                archiveArtifacts artifacts: "${BINARY_NAME}", fingerprint: true
                echo "Артефакт заархивирован"
            }
        }

        stage('Upload to Nexus') {
            steps {
                // Загружаем файл в raw-hosted репозиторий
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'localhost:8081',
                    repository: 'my-raw-repo',        // имя вашего репозитория в Nexus
                    credentialsId: 'nexus-login',     // ID креденшелов в Jenkins (настроим позже)
                    groupId: 'com.example',
                    version: "${VERSION}",
                    artifacts: [
                        [
                            artifactId: "${BINARY_NAME}",
                            type: 'bin',
                            file: "${BINARY_NAME}"
                        ]
                    ]
                )
                echo "Файл загружен в Nexus"
            }
        }
    }

    post {
        success {
            echo "Pipeline успешно выполнен! Бинарный файл загружен в Nexus."
        }
        failure {
            echo "Pipeline завершился с ошибкой."
        }
    }
}
