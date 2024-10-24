pipeline {
    agent any

    stages {
        stage('DAST') {
            steps {
                // Uruchomienie aplikacji Juice Shop w kontenerze
                sh '''
                    docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    sleep 10
                '''

                // Uruchomienie skanowania OWASP ZAP
                sh '''
                    docker run --name zap --rm \
                    -v ${WORKSPACE}/results:/zap/wrk/:rw \
                    ghcr.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts; zap.sh -cmd -autorun /zap/wrk/passive.yaml" || true
                '''

                // Wyświetlenie logów kontenera ZAP (przydatne do debugowania)
                sh '''
                    docker logs zap || true
                '''

                // Listowanie plików w kontenerze ZAP, aby upewnić się, że raporty zostały wygenerowane
                sh '''
                    docker exec zap ls -l /zap/wrk/reports || true
                '''

                // Testowanie dostępu do adresu host.docker.internal
                sh '''
                    docker exec zap curl http://host.docker.internal:3000/ || true
                '''
            }
        }
    }

    post {
        always {
            // Sprawdzenie, czy kontener zap istnieje przed próbą kopiowania
            script {
                def zapContainerExists = sh(script: "docker ps -a --filter 'name=zap' --format '{{.Names}}'", returnStdout: true).trim()
                if (zapContainerExists == "zap") {
                    // Kopiowanie wyników skanowania OWASP ZAP do workspace
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml || true
                    '''
                } else {
                    echo "Kontener ZAP nie istnieje, raporty nie zostaną skopiowane."
                }
            }

            // Zatrzymywanie i usuwanie kontenerów
            sh '''
                docker stop zap juice-shop || true
            '''

            // Sprawdzanie, czy raport ZAP XML istnieje
            script {
                def reportExists = fileExists("${WORKSPACE}/results/zap_xml_report.xml")
                if (reportExists) {
                    // Archiwizowanie wyników w Jenkinsie
                    archiveArtifacts artifacts: '${WORKSPACE}/results/**/*', fingerprint: true, allowEmptyArchive: true

                    // Wysyłanie raportów do DefectDojo
                    defectDojoPublisher(artifact: '${WORKSPACE}/results/zap_xml_report.xml',
                                        productName: 'Juice Shop',
                                        scanType: 'ZAP Scan',
                                        engagementName: 'mario360x@gmail.com')
                } else {
                    echo "Raport ZAP XML nie został znaleziony, nie można go wysłać do DefectDojo."
                }
            }
        }
    }
}
