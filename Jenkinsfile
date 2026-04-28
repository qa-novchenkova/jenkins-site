pipeline {
    agent any

    environment {
        // Путь к папке Node.js, где лежат node.exe и npm.cmd
        NODE_HOME = 'C:\\Program Files\\nodejs'

        // Добавляем Node.js в PATH для всех bat-команд
        PATH = "${env.NODE_HOME};${env.PATH}"

        // Порт, на котором будем запускать приложение
        APP_PORT = '8082'

        // Не даем Jenkins автоматически убить сервер между stage'ами
        JENKINS_NODE_COOKIE = 'dontKillMe'
    }

    stages {
        stage('Checkout') {
            steps {
                // Забираем код из GitHub по настройкам Pipeline job
                checkout scm
            }
        }

        stage('Versions') {
            steps {
                // Проверяем, что Jenkins видит Node.js и npm
                bat '''
                    @echo off
                    chcp 65001 >nul

                    echo [STEP] Node version
                    node -v

                    echo [STEP] NPM version
                    call npm -v
                '''
            }
        }

        stage('Install') {
            steps {
                // Устанавливаем зависимости проекта
                bat '''
                    @echo off
                    chcp 65001 >nul

                    echo [STEP] Install dependencies
                    call npm install --no-fund --no-audit
                '''
            }
        }

        stage('Build') {
            steps {
                // Собираем проект
                bat '''
                    @echo off
                    chcp 65001 >nul

                    echo [STEP] Build project
                    call npm run build
                '''
            }
        }

        stage('Start Server') {
            steps {
                // Перед запуском освобождаем порт и стартуем сервер в фоне
                bat '''
                    @echo off
                    chcp 65001 >nul

                    echo [STEP] Cleanup port %APP_PORT%
                    for /f "tokens=5" %%a in ('netstat -aon ^| findstr ":%APP_PORT%" ^| findstr "LISTENING"') do (
                        taskkill /f /pid %%a /t 2>nul
                    )

                    echo [STEP] Start server
                    start /B cmd /c "call npm run start -- --port %APP_PORT% > server_log.txt 2>&1"

                    echo [STEP] Wait server
                    timeout /t 10 /nobreak >nul
                '''
            }
        }

        stage('Test') {
            steps {
                // Запускаем тесты против поднятого локального сервера
                bat '''
                    @echo off
                    chcp 65001 >nul

                    echo [STEP] Run tests
                    call npm test
                '''
            }
        }
    }

    post {
        always {
            // Этот блок выполнится всегда: и при успехе, и при падении тестов
            bat '''
                @echo off
                chcp 65001 >nul

                echo [STEP] Final cleanup port %APP_PORT%
                for /f "tokens=5" %%a in ('netstat -aon ^| findstr ":%APP_PORT%" ^| findstr "LISTENING"') do (
                    taskkill /f /pid %%a /t 2>nul
                )
            '''
        }
    }
}