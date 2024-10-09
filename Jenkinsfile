pipeline {
    agent any

    environment {
        WORKSPACE="/home/ubuntu/abcd-student/resources"
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
                    docker run --name juice-shop -d --rm \\
                        -p 3000:3000 \\
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap --rm \\
                        --add-host=host.docker.internal:host-gateway \\
                        -v ${WORKSPACE}/:/zap/wrk/:rw \\
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \\
                        "zap.sh -cmd -addonupdate \\
                        && zap.sh -cmd -addoninstall communityScripts \\
                        -addoninstall pscanrulesAlpha \\
                        -addoninstall pscanrulesBeta \\
                        -autorun /zap/wrk/passive_scan.yaml"
                '''
                sh '''
                    docker cp zap:/zap/wrk/zap_html_report.html ${WORKSPACE}/reports/zap_html_report.html
                    docker cp zap:/zap/wrk/zap_xml_report.xml ${WORKSPACE}/reports/zap_xml_report.xml
                '''
            }
        }
        // stage('Upload raport to DefectDojo') {
        //     post {
        //         always {
        //             defectDojoPublisher(artifact: '${WORKSPACE}/results/zap_xml_report.xml', 
        //                 productName: 'Juice Shop', 
        //                 scanType: 'ZAP Scan', 
        //                 engagementName: 'change-me')
        //         }
        //     }
        // }
    }

    post {

        always {
            sh '''
                docker stop zap juice-shop
            '''
        }

    }
}