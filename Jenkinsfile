pipeline {
    agent any
    options {
        skipDefaultCheckout(true) // Pomijanie domyślnego checkoutu
    }

    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs() // Czyszczenie workspace
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'main'
                }
            }
        }

        stage('Prepare') {
            steps {
                // Tworzenie folderu na wyniki w workspace
                sh 'mkdir -p ${WORKSPACE}/results/'
            }
        }

        stage('DAST') {
            steps {
                // Uruchomienie aplikacji Juice Shop w kontenerze
                sh '''
                    docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    # Zwiększenie czasu oczekiwania, aby aplikacja mogła się w pełni załadować
                    sleep 10
                '''

                // Uruchomienie skanowania OWASP ZAP
                sh '''
                    docker run --name zap --rm \
                    -v ${WORKSPACE}/results:/zap/wrk/:rw \
                    ghcr.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts; zap.sh -cmd -autorun /zap/wrk/passive.yaml" || true
                '''
            }
        }
    }

    post {
        always {
            // Kopiowanie wyników skanowania OWASP ZAP do workspace
            sh '''
                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml || true
                docker stop zap || true
                docker stop juice-shop || true
            '''

            // Archiwizowanie wyników w Jenkinsie
            archiveArtifacts artifacts: '${WORKSPACE}/results/**/*', fingerprint: true, allowEmptyArchive: true

            // Wysyłanie raportów do DefectDojo
            defectDojoPublisher(artifact: '${WORKSPACE}/results/zap_xml_report.xml',
                                productName: 'Juice Shop',
                                scanType: 'ZAP Scan',
                                engagementName: 'mario360x@gmail.com')
        }
    }
}
