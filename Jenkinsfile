pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
        timestamps()
        ansiColor("xterm")
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'GitHub_PAT', url: 'https://github.com/lolszowy/abcd-student.git', branch: 'main'
                }
            }
        }
        stage('Run Juice Shop') {
            steps {
                echo 'Run Juice Shop'
                sh '''
                    docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    sleep 5
                '''
            }
        }
        stage('ZAP Passive Scan') {
            agent {
                docker {
                    image "ghcr.io/zaproxy/zaproxy:stable"
                    args "--name zap --add-host=host.docker.internal:host-gateway"
                }
            }
            steps {
                echo 'ZAP Passive Scan'
                sh 'mkdir -p /zap/wrk/reports/'
                sh 'zap.sh -cmd -addonupdate'
                sh '''
                cat ${WORKSPACE}/plan-testow-zap/passive_scan.yaml
                zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun ${WORKSPACE}/plan-testow-zap/passive_scan.yaml
                '''
            }
        }
    }
    post {
        always {
            sh '''
            ls -la /zap/wrk/reports/zap_html_report.html
            ls -la /zap/wrk/reports/zap_xml_report.xml
            docker stop juice-shop || true
            docker rm juice-shop || true              
            '''
        }
    }
}
