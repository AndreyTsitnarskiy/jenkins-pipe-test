@Library('git_utils') _

properties([
    parameters([
        string(name: 'REPO_URL', defaultValue: 'https://github.com/user/repo.git', description: 'Git repo URL'),
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to analyze'),
        string(name: 'SINCE_DATE', defaultValue: '', description: 'Опционально. Период (например, "2024-01-01" или "2 weeks ago")'),
        booleanParam(name: 'RAW_LOG', defaultValue: false, description: 'Вывести сырой лог (без обработки)'),
        choice(name: 'OUTPUT_FORMAT', choices: ['JSON', 'YAML', 'TEXT'], description: 'Формат вывода (если RAW_LOG=false)')
    ])
])


node {
    try {
        // Параметры (можно задать через Jenkins UI)
        def repoUrl = params.REPO_URL ?: 'https://github.com/user/repo.git'
        def branch = params.BRANCH ?: 'main'
        def sinceDate = params.SINCE_DATE ?: ''
        def rawLogMode = params.RAW_LOG ?: false
        def outputFormat = params.OUTPUT_FORMAT ?: 'TEXT'

        stage('Checkout') {
            checkout([
                $class: 'GitSCM',
                branches: [[name: branch]],
                userRemoteConfigs: [[url: repoUrl, credentialsId: 'git_key']]
            ])
        }

        stage('Get Git Log') {
            // Формируем команду git log
            def gitCmd = "git log"
            if (sinceDate && !rawLogMode) {
                gitCmd += " --since='${sinceDate}'"
            }

            // Получаем сырые данные
            def rawLog = sh(script: gitCmd, returnStdout: true).trim()

            if (!rawLog) {
                currentBuild.result = 'UNSTABLE'
                error("Нет коммитов${sinceDate ? ' за указанный период' : ''}")
            }

            // Режим сырого лога
            if (rawLogMode) {
                def result
                switch (outputFormat.toUpperCase()) {
                case 'JSON':
                        result = convertRawLogToJson(rawLog)
                        break
                    case 'YAML':
                        result = convertRawLogToYaml(rawLog)
                        break
                    default:  // TEXT
                        result = rawLog
                }
                writeFile file: "git_log.${outputFormat.toLowerCase()}", text: result
                archiveArtifacts artifacts: "git_log.${outputFormat.toLowerCase()}"
                echo "Результат (RAW):\n${result}"
                return
            }

            // Режим парсинга (форматированный вывод)
            def formattedLog = sh(
                script: "${gitCmd} --pretty=format:'%h | %an | %ad | %s' --date=short",
                returnStdout: true
            ).trim()

            def report = gitParser(rawLog: formattedLog, format: outputFormat)
            writeFile file: "report.${outputFormat.toLowerCase()}", text: report
            archiveArtifacts artifacts: "report.${outputFormat.toLowerCase()}"
            echo "Отчёт:\n${report}"
        }

        currentBuild.result = 'SUCCESS'

    } catch (Exception e) {
        if (currentBuild.result != 'UNSTABLE') {
            currentBuild.result = 'FAILURE'
        }
        echo "❌ Ошибка: ${e.message}"
    } finally {
        echo "Завершено со статусом: ${currentBuild.result}"
    }
}

// Конвертация сырого лога в JSON
def convertRawLogToJson(String rawLog) {
    def commits = []
    def currentCommit = [:]
    rawLog.eachLine { line ->
        if (line.startsWith('commit ')) {
        if (currentCommit) commits.add(currentCommit)
            currentCommit = [hash: line.substring(7).trim()]
        } else if (line.startsWith('Author:')) {
        currentCommit.author = line.substring(8).trim()
        } else if (line.startsWith('Date:')) {
        currentCommit.date = line.substring(5).trim()
        } else if (line.trim()) {
        currentCommit.message = (currentCommit.message ?: '') + line.trim() + '\n'
        }
    }
    if (currentCommit) commits.add(currentCommit)
    return groovy.json.JsonOutput.prettyPrint(groovy.json.JsonOutput.toJson(commits))
}

// Конвертация сырого лога в YAML
def convertRawLogToYaml(String rawLog) {
    def yaml = "commits:\n"
    def currentCommit = [:]
    rawLog.eachLine { line ->
        if (line.startsWith('commit ')) {
        if (currentCommit) {
            yaml += "  - hash: ${currentCommit.hash}\n"
                yaml += "    author: ${currentCommit.author}\n"
                yaml += "    date: ${currentCommit.date}\n"
                yaml += "    message: |\n      ${currentCommit.message?.trim()?.replace('\n', '\n      ') ?: ''}\n"
            }
            currentCommit = [hash: line.substring(7).trim()]
        } else if (line.startsWith('Author:')) {
        currentCommit.author = line.substring(8).trim()
        } else if (line.startsWith('Date:')) {
        currentCommit.date = line.substring(5).trim()
        } else if (line.trim()) {
        currentCommit.message = (currentCommit.message ?: '') + line.trim() + '\n'
        }
    }
    return yaml
}