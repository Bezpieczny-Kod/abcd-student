pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
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
            steps {
                echo 'ZAP Passive Scan'
                sh 'mkdir -p results/'
                
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /home/lolszowy/git/abcd/abcd-student/plan-testow-zap/passive_scan.yaml:/zap/wrk/:rw
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" \
                        || true
                '''
            }
        }
    }
    post {
        always {
            sh '''
                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml || true
                docker stop zap juice-shop || true
                docker rm zap
                docker stop juice-shop || true
                docker rm juice-shop
            '''
        }
    }
}
