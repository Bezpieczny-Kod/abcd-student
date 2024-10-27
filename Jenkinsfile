pipeline {
    agent any
    options {
        skipDefaultCheckout(true) // Pomijanie domyślnego checkoutu
    }

    stages {
        stage('Step 1: Code checkout for GitHub') {
            steps {
                script {
                    cleanWs() // Czyszczenie workspace
                    echo "Checking out code from GitHub repository..."
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'main'
                    echo "Code checked out. Listing workspace contents..."
                    sh 'ls -al ${WORKSPACE}'  // Wyświetlenie zawartości katalogu roboczego
                    echo "Waiting for 5 seconds..."
                    sleep(5) // Pauza 5 sekund
                }
            }
        }

        stage('Step 2: Prepare') {
            steps{
                sh 'mkdir -p results'
            }
            }
        }

        stage('Step 3: Dast') {
            steps {
        
                sh '''
                    mkdir -p /tmp/reports
                    chmod -R 777 /tmp/reports
                '''
                echo "Directory created. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }

        stage('Step 4: Copy passive.yaml File') {
            steps {
                echo "Copying passive.yaml file from repository to workspace..."
                // Kopiowanie pliku passive.yaml do zap-results
                sh '''
                    cp ${WORKSPACE}/passive.yaml ./zap-results/passive.yaml
                '''
                echo "File copied. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }

        stage('Step 5: Run OWASP ZAP for Passive Scanning') {
            steps {
                echo "Starting OWASP ZAP container..."
                sh '''
                    docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v /tmp:/zap/wrk:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate; \
                    zap.sh -cmd -addoninstall communityScripts; \
                    zap.sh -cmd -addoninstall pscanrulesAlpha; \
                    zap.sh -cmd -addoninstall pscanrulesBeta; \
                    zap.sh -cmd -autorun /zap/wrk/passive.yaml" || true
                '''
                echo "OWASP ZAP scan complete. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }

        stage('Step 6: Archive Scan Results') {
            steps {
                echo "Verifying scan results..."
                sh 'ls -al /tmp/reports' // Sprawdzanie zawartości katalogu z wynikami

                echo "Archiving scan results..."
                archiveArtifacts artifacts: '/tmp/reports/**/*', fingerprint: true, allowEmptyArchive: true
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

                echo "Checking if ZAP XML report exists..."
                if (fileExists('./zap-results/reports/zap_xml_report.xml')) {
                    echo "Sending ZAP XML report to DefectDojo..."
                    defectDojoPublisher(artifact: './zap-results/reports/zap_xml_report.xml',
                                        productName: 'Juice Shop',
                                        scanType: 'ZAP Scan',
                                        engagementName: 'mario360x@gmail.com')
                } else {
                    echo "ZAP XML report not found, skipping DefectDojo upload."
                }
            }

            // Archiwizowanie wyników w Jenkinsie
            archiveArtifacts artifacts: '/tmp/reports/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }

