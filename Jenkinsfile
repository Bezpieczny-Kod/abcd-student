pipeline {
    agent any

    environment {
        WORKSPACE='/home/ubuntu/abcd-student/resources'
        REPORT_DIR='/scan/reports'
    }

    parameters {
        string(name: 'EMAIL', defaultValue: 'email@example.com', description: 'ABC-DevSecOps user email')
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-token', url: 'https://github.com/m3m03y/abcd-student', branch: 'dast-implementation'
                }
            }
        }

        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap --rm \
                        --add-host=host.docker.internal:host-gateway \
                        -v ${WORKSPACE}/:/zap/wrk/:rw \
                        -v ${WORKSPACE}/reports/:/zap/wrk/reports/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate \
                        && zap.sh -cmd -addoninstall communityScripts \
                        -addoninstall pscanrulesAlpha \
                        -addoninstall pscanrulesBeta \
                        -autorun /zap/wrk/passive_scan.yaml"
                '''
            }
        }

        stage('[ZAP] Copy scan result') {
            steps {
                sh '''
                    mkdir -p ${REPORT_DIR}
                '''

                sh '''
                    docker cp zap:/zap/wrk/zap_xml_report.xml ${REPORT_DIR}/zap_xml_report.xml 
                '''
            }
        }

   }

    post {
        always {
            echo 'Send report from: ${EMAIL}'
            defectDojoPublisher(artifact: '${REPORT_DIR}/zap_xml_report.xml', 
                productName: 'Juice Shop', 
                scanType: 'ZAP Scan', 
                engagementName: '${EMAIL}')
            sh '''
                docker stop zap juice-shop
            '''
        }
    }
}