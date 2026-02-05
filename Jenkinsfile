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
                        args '--entrypoint="" -u root:root'
                    }
                }
            environment {
                TERRASCAN_HOME = "${WORKSPACE}/.terrascan"
            }
            steps {
                script {
                        // Recuperar el código stasheado
                        unstash 'terragoat-code'
                        
                        // Cear TERRASCAN_HOME Eliminar archivo anterior si existe
                        sh '''
                            mkdir -p $TERRASCAN_HOME
                            rm -f terrascan-report.json || true
                        '''

                        // Ejecutar Terrascan capturando el exit code
                        def terrascanExitCode = sh(script: '''
                            cd terragoat
                            terrascan scan -i terraform -d . -o json \
                                --config-path="$TERRASCAN_HOME" \
                                --policy-path="$TERRASCAN_HOME/policies" \
                                > ../terrascan-report.json 2>&1
                        ''', returnStatus: true)
                        
                        echo "Terrascan exit code: ${terrascanExitCode}"

                    // Verificar que el archivo se creó
                        sh "test -f terrascan-report.json && echo 'Archivo terrascan-report.json creado' || echo 'Archivo no existe, creando vacío...'"
                        sh "test -f terrascan-report.json || echo '{}' > terrascan-report.json"
                        sh "ls -la terrascan-report.json"

                    // Archivar resultados
                    archiveArtifacts artifacts: "terrascan-report.json", fingerprint: true, allowEmptyArchive: true
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
