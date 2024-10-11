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
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v ${WORKSPACE}/:/zap/wrk/:rw \
                        -v ${WORKSPACE}/reports/:/zap/wrk/reports/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate \
                        && zap.sh -cmd -addoninstall communityScripts \
                        -addoninstall pscanrulesAlpha \
                        -addoninstall pscanrulesBeta \
                        -autorun /zap/wrk/active_scan.yaml"
                '''
            }
        }

        stage('[ZAP] Copy scan result') {
            steps {
                sh '''
                    mkdir -p ${REPORT_DIR}
                '''

                sh '''
                    docker cp zap:/zap/wrk/reports/zap_*_report* ${REPORT_DIR}/ 
                '''
            }
        }

        stage('[ZAP] Upload report to Defect Dojo') {
            steps {
                echo 'Archiving results...'
                archiveArtifacts artifacts: '${REPORT_DIR}/*', fingerprint: true, allowEmptyArchive: true
                sh '''
                    echo Send report to DefectDojo from: ${EMAIL}
                '''
                defectDojoPublisher(artifact: '${REPORT_DIR}/zap_xml_report_active.xml', 
                    productName: 'Juice Shop', 
                    scanType: 'ZAP Scan', 
                    engagementName: '${EMAIL}') 
            }
        }
   }

    post {
        always {
           sh '''
                docker stop zap juice-shop
                docker rm zap
            '''
        }
    }
}