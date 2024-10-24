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
                // Tworzenie katalogu results, aby mieć pewność, że istnieje
                sh 'mkdir -p ${WORKSPACE}/zap-results'
            }
        }

        stage('Check passive_scan.yaml exists in workspace') {
            steps {
                // Sprawdzenie, czy plik passive_scan.yaml został pobrany z repozytorium i istnieje w workspace
                sh 'ls -l ${WORKSPACE}/passive_scan.yaml'
            }
        }

        stage('DAST - OWASP ZAP scan') {
            steps {
                // Sprawdzenie i usunięcie istniejącego kontenera "zap", jeśli istnieje
                sh '''
                    if [ $(docker ps -a -q -f name=zap) ]; then
                        docker stop zap || true
                        docker rm zap || true
                    fi
                '''

                // Uruchomienie kontenera ZAP z wymaganymi parametrami
                sh '''
                    docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v ${WORKSPACE}/zap-results:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate; \
                    zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta; \
                    zap.sh -cmd -autorun /zap/wrk/passive_scan.yaml" || true
                '''
            }
        }
    }

    post {
        always {
            script {
                def zapContainerExists = sh(script: "docker ps -a --filter 'name=zap' --format '{{.Names}}'", returnStdout: true).trim()
                if (zapContainerExists == "zap") {
                    // Kopiowanie wyników skanowania OWASP ZAP do katalogu results
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/zap-results/zap_html_report.html || true
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/zap-results/zap_xml_report.xml || true
                    '''
                } else {
                    echo "Kontener ZAP nie istnieje, raporty nie zostaną skopiowane."
                }
            }

            // Zatrzymywanie i usuwanie kontenerów
            sh '''
                docker stop zap juice-shop || true
            '''

            script {
                def reportExists = fileExists("${WORKSPACE}/zap-results/zap_xml_report.xml")
                if (reportExists) {
                    // Archiwizowanie wyników w Jenkinsie
                    archiveArtifacts artifacts: '${WORKSPACE}/zap-results/**/*', fingerprint: true, allowEmptyArchive: true

                    // Wysyłanie raportów do DefectDojo
                    defectDojoPublisher(artifact: '${WORKSPACE}/zap-results/zap_xml_report.xml',
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
