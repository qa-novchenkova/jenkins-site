pipeline {
    // Запускать pipeline можно на любом доступном Jenkins-агенте.
    // Jenkins стоит локально на Windows, поэтому команды bat выполняются на этой машине.
    agent any

    environment {
        // NODE_HOME берется из параметров Jenkins job.
        // Пример значения: C:\Program Files\nodejs
        //
        // Здесь мы добавляем папку Node.js в PATH.
        // Это нужно, чтобы команды node и npm были доступны внутри Jenkins.
        PATH = "${params.NODE_HOME};${env.PATH}"

        // SERVER_WAIT_SECONDS берется из параметров Jenkins job.
        // Это число секунд, сколько Jenkins будет ждать после запуска сервера
        // перед тем как запускать тесты.
        WAIT_SECONDS = "${params.SERVER_WAIT_SECONDS}"
    }

    stages {
        stage('Versions') {
            steps {
                bat '''
                    @echo off
                    chcp 65001 >nul

                    :: Проверяем, что Jenkins видит Node.js
                    echo [STEP] Node version
                    node -v

                    :: Проверяем, что Jenkins видит npm
                    :: call нужен для npm-команд в Windows batch,
                    :: иначе .bat-скрипт может завершиться раньше времени
                    echo [STEP] NPM version
                    call npm -v
                '''
            }
        }

        stage('Install') {
            // Этот этап выполнится только если в параметрах сборки
            // RUN_NPM_INSTALL = true.
            when {
                expression {
                    return params.RUN_NPM_INSTALL
                }
            }

            steps {
                bat '''
                    @echo off
                    chcp 65001 >nul

                    :: Устанавливаем зависимости из package.json
                    :: --no-fund убирает сообщения про финансирование пакетов
                    :: --no-audit отключает проверку уязвимостей, чтобы сборка была быстрее
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

                    :: Запускаем команду build из package.json
                    :: Обычно она создает готовую сборку проекта
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

                    :: Говорим Jenkins не убивать процесс сервера автоматически.
                    :: Сервер нужен временно, чтобы тесты могли открыть сайт.
                    :: В конце pipeline мы сами остановим процесс по порту.
                    set BUILD_ID=dontKillMe

                    :: PORT берется из параметров Jenkins job.
                    :: Перед запуском сервера очищаем этот порт,
                    :: если после прошлой сборки там остался старый процесс.
                    echo [STEP] Cleanup port %PORT%
                    for /f "tokens=5" %%a in ('netstat -aon ^| findstr ":%PORT%" ^| findstr "LISTENING"') do (
                        taskkill /f /pid %%a /t 2>nul
                    )

                    :: Запускаем сайт в фоне на нужном порту.
                    :: Логи сервера пишем в server_log.txt в workspace Jenkins.
                    :: Если сервер не стартует, этот файл поможет понять причину.
                    echo [STEP] Start server on port %PORT%
                    start /B cmd /c "call npm run start -- --port %PORT% > server_log.txt 2>&1"

                    :: Ждем, чтобы сервер успел подняться перед тестами.
                    :: Используем PowerShell sleep, потому что timeout иногда
                    :: некрасиво работает в Jenkins console на Windows.
                    echo [STEP] Wait server %WAIT_SECONDS% seconds
                    powershell -NoProfile -Command "Start-Sleep -Seconds %WAIT_SECONDS%"
                '''
            }
        }

        stage('Test') {
            // Этот этап выполнится только если в параметрах сборки
            // RUN_TESTS = true.
            when {
                expression {
                    return params.RUN_TESTS
                }
            }

            steps {
                bat '''
                    @echo off
                    chcp 65001 >nul

                    :: Запускаем тесты из package.json
                    :: Если тесты упадут, Jenkins пометит сборку как failed.
                    echo [STEP] Run tests
                    call npm test
                '''
            }
        }
    }

    post {
        // Блок always выполняется всегда:
        // и при успешной сборке, и при ошибке.
        // Это важно, чтобы временный сервер не остался висеть после падения тестов.
        always {
            bat '''
                @echo off
                chcp 65001 >nul

                :: Финальная очистка порта.
                :: Ищем процесс, который слушает PORT, и завершаем только его.
                :: Так мы не убиваем все node.exe, а трогаем только сервер этой сборки.
                echo [STEP] Final cleanup port %PORT%
                for /f "tokens=5" %%a in ('netstat -aon ^| findstr ":%PORT%" ^| findstr "LISTENING"') do (
                    taskkill /f /pid %%a /t 2>nul
                )
            '''
        }
    }
}