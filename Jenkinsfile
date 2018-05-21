def notifySuccessful(String stageName) {
  slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})" + "\n  Stage -- " + stageName)
}

def notifyFailed(String stageName) {
  slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})" + "\n  Stage -- " + stageName)
}

pipeline {
    agent any

    stages {
        stage('shellcheck') {
            steps {
                echo 'running shellcheck..'
                sh -f checkstyle bootstrap-salt.sh | tee checkstyle.xml
                // publishIssues issues:[checkstyle]
            }
        }
    }
}
