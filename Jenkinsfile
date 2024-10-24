pipeline {
    agent any
    options {
        skipDefaultCheckout(true) // Pomijanie domyślnego checkoutu
    }

    stages {
        stage('Step 1: Prepare Juice Shop Application for Testing') {
            steps {
                script {
                    cleanWs() // Czyszczenie workspace
                    echo "Starting Juice Shop application..."
                    sh '''
                        docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    '''
                    echo "Juice Shop is running. Waiting for 5 seconds..."
                    sleep(5) // Pauza 5 sekund
                }
            }
        }

        stage('Step 2: Prepare Directory for Scan Results') {
            steps {
                echo "Creating directory for scan results..."
                sh '''
                    mkdir -p ./zap-results/reports
                '''
                echo "Directory created. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }

        stage('Step 3: Copy passive_scan.yaml File') {
            steps {
                echo "Copying passive_scan.yaml file from repository to workspace..."
                // Kopiowanie pliku passive_scan.yaml do zap-results
                sh '''
                    cp ${WORKSPACE}/passive_scan.yaml ./zap-results/passive_scan.yaml
                '''
                echo "File copied. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }

        stage('Step 4: Run OWASP ZAP for Passive Scanning') {
            steps {
                echo "Starting OWASP ZAP container..."
                sh '''
                    docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v $(pwd)/zap-results:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate; \
                    zap.sh -cmd -addoninstall communityScripts; \
                    zap.sh -cmd -addoninstall pscanrulesAlpha; \
                    zap.sh -cmd -addoninstall pscanrulesBeta; \
                    zap.sh -cmd -autorun /zap/wrk/passive_scan.yaml" || true
                '''
                echo "OWASP ZAP scan complete. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }

        stage('Step 5: Archive Scan Results') {
            steps {
                echo "Archiving scan results..."
                // Archiwizowanie wyników w Jenkinsie
                archiveArtifacts artifacts: './zap-results/reports/**/*', fingerprint: true, allowEmptyArchive: true
                echo "Scan results archived. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }
    }

    post {
        always {
            script {
                echo "Cleaning up Docker containers..."
                sh '''
                    docker stop zap juice-shop || true
                    docker rm zap juice-shop || true
                '''
                echo "Containers stopped and removed."

                echo "Sending ZAP XML report to DefectDojo..."
                // Wysyłanie raportów do DefectDojo
                defectDojoPublisher(artifact: './zap-results/reports/zap_xml_report.xml',
                                    productName: 'Juice Shop',
                                    scanType: 'ZAP Scan',
                                    engagementName: 'mario360x@gmail.com')
            }

            // Archiwizowanie wyników w Jenkinsie
            archiveArtifacts artifacts: './zap-results/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}
