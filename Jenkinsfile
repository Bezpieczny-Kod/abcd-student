pipeline {
    agent any
    options {
        skipDefaultCheckout(true) // Pomijanie domyślnego checkoutu
    }

    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs() // Czyszczenie workspace, aby pipeline zaczynał w czystym stanie
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'main'
                }
            }
        }

        stage('Prepare') {
            steps {
                // Tworzenie katalogu results, aby mieć pewność, że istnieje
                sh 'mkdir -p ${WORKSPACE}/results'
            }
        }

        stage('Check passive.yaml exists in workspace') {
            steps {
                // Sprawdzenie, czy plik passive.yaml został pobrany z repozytorium i istnieje w workspace
                sh 'ls -l ${WORKSPACE}/passive.yaml'
            }
        }

        stage('DAST - OWASP ZAP scan') {
            steps {
                // Montowanie całego katalogu workspace, w którym znajduje się passive.yaml
                sh '''
                    docker run --name zap --rm \
                    -v ${WORKSPACE}:/zap/wrk/:rw \
                    ghcr.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts; zap.sh -cmd -autorun /zap/wrk/passive.yaml" || true
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
