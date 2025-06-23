@Library('git_utils') _  // Подключаем вашу библиотеку

pipeline {
    agent any
    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/user/repo.git', description: 'URL Git-репозитория')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Ветка для анализа')
        string(name: 'SINCE_DATE', defaultValue: '1 week ago', description: 'Период (e.g., "2023-01-01" или "2 weeks ago")')
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
                    // Получаем и анализируем историю коммитов
                    def gitLog = sh(
                        script: """
                            git log \
                            --since="${params.SINCE_DATE}" \
                            --pretty=format:'%h | %an | %ad | %s' \
                            --date=short
                        """,
                        returnStdout: true
                    ).trim()

                    // Форматируем вывод
                    git_utils.formatGitLog(gitLog)

                    // Сохраняем в артефакт
                    writeFile file: 'git_history.txt', text: gitLog
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'git_history.txt', fingerprint: true
        }
    }
}
