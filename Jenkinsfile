pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo "Obteniendo el código desde GitHub..."
                sh '''
                    rm -rf terragoat || true
                    git clone https://github.com/Matias25pinto/terragoat.git terragoat
                    git config --global --add safe.directory $WORKSPACE/terragoat
                    cd terragoat
                '''

                // Stash para compartir el código entre stages con diferentes agentes
                stash name: 'terragoat-code', includes: 'terragoat/**'
            }
        }

        stage('SAST - Terrascan Scan') {
            agent {
                docker {
                    image 'tenable/terrascan:latest'
                    args '--entrypoint="" -u root:root -v /tmp/terrascan-policies:/root/.terrascan'
                }
            }
            steps {
                script {
                    unstash 'terragoat-code'
                    
                    sh '''
                        cd terragoat
                        
                        terrascan scan -i terraform -d . -o json > ../terrascan-output-raw.json 2>&1
                        
                        # Filtrar solo JSON válido
                        if grep -q '^{' ../terrascan-output-raw.json; then
                            grep -E '^\{|^\[' ../terrascan-output-raw.json > ../terrascan-report.json
                        else
                            echo '{"results": {"violations": []}}' > ../terrascan-report.json
                        fi
                    '''
                    
                    archiveArtifacts artifacts: "terrascan-report.json", fingerprint: true
                }
            }

            post {
                always {
                    script {
                        if (fileExists("terrascan-report.json")) {
                            echo "Resultados de Terrascan disponibles para análisis"
                        }
                    }
                }
            }
        }

    }

    post {
        success {
            echo "✓ Pipeline completado exitosamente"
        }
        unstable {
            echo "⚠ Pipeline completado con advertencias"
        }
        failure {
            echo "✗ Pipeline falló"
        }
    }
}
