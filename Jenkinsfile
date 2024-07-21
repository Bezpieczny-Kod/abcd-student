pipeline {
    agent any
    environment {
        BRANCH_NAME = "${env.GIT_BRANCH}".replaceFirst(/^origin\//, '')
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                git credentialsId: '24468485-9389-43ce-b007-fad88c304d7e', url: 'https://github.com/krzysztofkorozej/abcd-student-poc', branch: $BRANCH_NAME
            }
        }
        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
            }
        }
        stage('[TruffleHog] Scan for secrets') {
            steps { 
                sh 'trufflehog git file://. --since-commit main --branch HEAD --fail --no-update'
            }
        }
    }
}