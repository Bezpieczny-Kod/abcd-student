pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub main') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-student', url: 'https://github.com/tyriusz/abcd-student', branch: 'sca-scan'
                }
            }
        }
        stage('Example') {
            steps {
                echo 'Hello!!'
                sh 'whoami'
            }
        }
        stage('[OSV-Scanner] Dependency scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    docker run --name osv-scanner \
                        -v /c/Users/Piotrek/Documents/abcd-devsecops/working/abcd-student/.osv:/osv/wrk/:rw \
                        -t ghcr.io/google/osv-scanner:latest \
                         ghcr.io/google/osv-scanner:stable bash -c \
                         --lockfile=package-lock.json
                    '''
            }
            post {
                always {
                    echo 'OSV-Scanner finished.'
                }
            }
        }
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    docker run --name juice-shop -d \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /c/Users/Piotrek/Documents/abcd-devsecops/working/abcd-student/.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" \
                        || true
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                    '''
                    defectDojoPublisher(artifact: 'results/zap_xml_report.xml',
                       productName: 'Juice Shop',
                       scanType: 'ZAP Scan',
                       engagementName: 'piotr.tyrala.mail@gmail.com')
                }
            }
        }
    }
}
