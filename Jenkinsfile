@Library('git_utils') _  // Подключаем библиотеку

// Параметры (можно задать через Jenkins UI)
properties([
    parameters([
        string(name: 'REPO_URL', defaultValue: 'https://github.com/user/repo.git', description: 'Git repo URL'),
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to analyze'),
        string(name: 'SINCE_DATE', defaultValue: '1 week ago', description: 'Period (e.g., "2023-01-01" or "2 weeks ago")'),
        choice(name: 'OUTPUT_FORMAT', choices: ['JSON', 'YAML', 'TEXT'], description: 'Output format')
    ])
])

// Сам Pipeline
node {
    stage('Checkout') {
        checkout([
            $class: 'GitSCM',
            branches: [[name: params.BRANCH]],
            userRemoteConfigs: [[url: params.REPO_URL, credentialsId: 'git_key']]
        ])
    }

    stage('Get Git Log') {
        script {
            // Получаем raw-лог (опасная операция — делаем в Jenkinsfile)
            def rawLog = sh(
                script: "git log --since='${params.SINCE_DATE}' --pretty=format:'%h | %an | %ad | %s' --date=short",
                returnStdout: true
            ).trim()

            // Передаем в библиотеку для парсинга
            def report = gitParser(
                rawLog: rawLog,
                format: params.OUTPUT_FORMAT
            )

            // Сохраняем отчет
            writeFile file: "report.${params.OUTPUT_FORMAT.toLowerCase()}", text: report
            archiveArtifacts artifacts: "report.${params.OUTPUT_FORMAT.toLowerCase()}"

            // Выводим результат (опционально)
            echo "Report:\n${report}"
        }
    }
}