pipeline {
    agent any
    options {
        skipDefaultCheckout(true) // Opcja pomijająca domyślne checkout w Jenkinsie
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
                // Tworzenie folderu na wyniki
                sh 'mkdir -p ~/Documents/DevSecOps/Test/results/'
            }
        }

        stage('DAST') {
            steps {
                // Uruchomienie aplikacji juice-shop w kontenerze
                sh '''
                    docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    # Zwiększono czas oczekiwania, by aplikacja mogła się w pełni załadować
                    sleep 10
                '''

                // Uruchomienie skanowania OWASP ZAP
                sh '''
                    docker run --name zap --rm \
                    -v ~/Documents/DevSecOps/Test:/zap/wrk/:rw \
                    ghcr.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts; zap.sh -cmd -autorun /zap/wrk/passive.yaml" || true
                '''
            }
        }
    }

    post {
        always {
            // Kopiowanie wyników skanowania OWASP ZAP
            sh '''
                docker cp zap:/zap/wrk/reports/zap_html_report.html ~/Documents/DevSecOps/Test/results/zap_html_report.html
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ~/Documents/DevSecOps/Test/results/zap_xml_report.xml
                docker stop zap juice-shop
                docker rm zap
            '''

            // Archiwizowanie wyników
            archiveArtifacts artifacts: '~/Documents/DevSecOps/Test/results/**/*', fingerprint: true, allowEmptyArchive: true

            // Wysyłanie raportów do DefectDojo
            defectDojoPublisher(artifact: '~/Documents/DevSecOps/Test/results/zap_xml_report.xml',
                                productName: 'Juice Shop',
                                scanType: 'ZAP Scan',
                                engagementName: 'test@test.pl')
        }
    }
}
