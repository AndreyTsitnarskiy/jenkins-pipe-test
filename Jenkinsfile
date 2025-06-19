@Library('git_utils') _  // Подключаем вашу библиотеку

pipeline {
    agent any
    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/user/repo.git', description: 'URL Git-репозитория')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Ветка для анализа')
        string(name: 'SINCE_DATE', defaultValue: '1 week ago', description: 'Период (e.g., "2023-01-01" или "2 weeks ago")')
        choice(name: 'OUTPUT_FORMAT', choices: ['JSON', 'YAML', 'TEXT'], description: 'Формат вывода')
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Клонируем репозиторий
                    git branch: params.BRANCH,
                         url: params.REPO_URL,
                         credentialsId: 'git-creds'  // ID SSH-ключа или логина/пароля в Jenkins
                }
            }
        }
        stage('Analyze Git Log') {
            steps {
                script {
                    def report = gitParser(
                        format: params.OUTPUT_FORMAT,
                        since: params.SINCE_DATE
                    )

                    writeFile file: "report.${params.OUTPUT_FORMAT.toLowerCase()}", text: report
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: "report.${params.OUTPUT_FORMAT.toLowerCase()}"
        }
    }
}