pipeline {
    agent any

    environment {
        // Путь к Node.js на Windows
        NODE_HOME = 'C:\\Program Files\\nodejs'

        // Добавляем Node.js и npm в PATH
        PATH = "${env.NODE_HOME};${env.PATH}"

        // Порт для запуска сайта
        // 8080 не используем, потому что на нем работает Jenkins
        PORT = '8082'
    }

    stages {
        stage('Versions') {
            steps {
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
                bat '''
                    @echo off
                    chcp 65001 >nul

                    :: Не даем Jenkins автоматически убить сервер сразу после старта
                    set BUILD_ID=dontKillMe

                    echo [STEP] Cleanup port %PORT%
                    for /f "tokens=5" %%a in ('netstat -aon ^| findstr ":%PORT%" ^| findstr "LISTENING"') do (
                        taskkill /f /pid %%a /t 2>nul
                    )

                    echo [STEP] Start server
                    start /B cmd /c "call npm run start -- --port %PORT% > server_log.txt 2>&1"

                    echo [STEP] Wait server
                    powershell -NoProfile -Command "Start-Sleep -Seconds 10"
                '''
            }
        }

        stage('Test') {
            steps {
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
            bat '''
                @echo off
                chcp 65001 >nul

                echo [STEP] Final cleanup port %PORT%
                for /f "tokens=5" %%a in ('netstat -aon ^| findstr ":%PORT%" ^| findstr "LISTENING"') do (
                    taskkill /f /pid %%a /t 2>nul
                )
            '''
        }
    }
}