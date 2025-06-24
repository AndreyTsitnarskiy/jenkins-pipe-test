@Library('git_utils') _

properties([
    parameters([
        string(name: 'REPO_URL', defaultValue: 'https://github.com/user/repo.git', description: 'Git repo URL'),
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to analyze'),
        string(name: 'SINCE_DATE', defaultValue: '1 week ago', description: 'Period (e.g., "2023-01-01" or "2 weeks ago")'),
        choice(name: 'OUTPUT_FORMAT', choices: ['JSON', 'YAML', 'TEXT'], description: 'Output format')
    ])
])

node {
    try {
        stage('Checkout') {
            script {
                try {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: params.BRANCH]],
                        userRemoteConfigs: [[url: params.REPO_URL, credentialsId: 'git_key']]
                    ])
                } catch (Exception e) {
                    unstable("Checkout failed: ${e.message}")
                    return
                }
            }
        }

        stage('Get Git Log') {
            script {
                def rawLog = ""
                try {
                    rawLog = sh(
                        script: "git log --since='${params.SINCE_DATE}' --pretty=format:'%h | %an | %ad | %s' --date=short",
                        returnStdout: true
                    ).trim()

                    if (!rawLog) {
                        unstable("No commits found since ${params.SINCE_DATE}")
                        return
                    }

                } catch (Exception e) {
                    unstable("Git log error: ${e.message}")
                    return
                }

                def report = gitParser(
                    rawLog: rawLog,
                    format: params.OUTPUT_FORMAT
                )

                writeFile file: "report.${params.OUTPUT_FORMAT.toLowerCase()}", text: report
                archiveArtifacts artifacts: "report.${params.OUTPUT_FORMAT.toLowerCase()}"
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        error("Pipeline failed: ${e.message}")
    }
}