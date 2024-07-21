pipeline {
    agent any
    stages {
        stage('Code checkout from GitHub') {
            steps {
                git credentialsId: '24468485-9389-43ce-b007-fad88c304d7e', url: 'https://github.com/krzysztofkorozej/abcd-student-poc', branch: 'main'
            }
        }
        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'echo "$GIT_COMMIT"'
            }
        }
        stage('[TruffleHog] Scan for secrets') {
            steps { 
                sh 'trufflehog git file://. --no-update'
            }
        }
    }
}