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
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh 'mkdir -p ${WORKSPACE}/results/'
                sh '''
                pwd
                ls -la
                docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v /home/lolszowy/git/abcd/abcd-student/plan-testow-zap/:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "mkdir /zap/wrk/reports; zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" 
                '''
            }
        }
    }
    post {
        success {
            sh '''            
            docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
            docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
            '''
            archiveArtifacts artifacts: "results/zap_html_report.html", allowEmptyArchive: true
            defectDojoPublisher(artifact: '${WORKSPACE}/results/zap_xml_report.xml',
                productName: 'Juice Shop',
                scanType: 'ZAP Scan',
                engagementName: 'lolszowy@gmail.com'
            )
            sh '''
            docker stop zap juice-shop || true
            docker rm zap juice-shop || true
            '''   
        }
        failure {            
            sh '''
            docker stop zap juice-shop || true
            docker rm zap juice-shop || true
            '''            
        }
    }
}
// OSV Scan
// Semgrep JSON Report
// Trufflehog Scan