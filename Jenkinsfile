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
            steps {
                script {
                    unstash 'terragoat-code'
                    
                    // Eliminar archivo anterior si existe
                    sh "rm -f terrascan-report.json || true"
                    
                    // Ejecutar terraScan capturando el exit code
                    def terraScanExitCode = sh(script: '''
                        cd terragoat
                        terrascan init scan
                        terrascan scan -o json > ../terrascan-report.json
                    ''', returnStatus: true)
                    
                    echo "TerraScan exit code: ${terraScanExitCode}"

                    // Verificar que el archivo se creó
                    sh "test -f terrascan-report.json && echo 'Archivo terrascan-report.json creado' || echo 'Archivo no existe, creando vacío...'"
                    sh "test -f terrascan-report.json || echo '{}' > terrascan-report.json"
                    sh "ls -la terrascan-report.json"
                    
                    archiveArtifacts artifacts: "terrascan-report.json", fingerprint: true

                    // Leer el archivo como texto
                    def jsonText = readFile('terrascan-report.json')
                    
                    // Parsear con Groovy JsonSlurper (no requiere plugin)
                    def jsonReport = new groovy.json.JsonSlurper().parseText(jsonText)
                    
                    // Ahora puedes acceder a los datos
                    def lowCount = jsonReport.results.scan_summary.low
                    def mediumCount = jsonReport.results.scan_summary.medium
                    def highCount = jsonReport.results.scan_summary.high
                    def totalViolations = jsonReport.results.violations.size()
                    
                    // Imprimir resumen en formato legible
                    echo """
                    ==========================================
                    RESUMEN DE ESCANEO TERRASCAN
                    ==========================================
                    Vulnerabilidades HIGH:    ${highCount}
                    Vulnerabilidades MEDIUM:  ${mediumCount}
                    Vulnerabilidades LOW:     ${lowCount}
                    ------------------------------------------
                    TOTAL:                    ${totalViolations}
                    ==========================================
                    """

                    //capturar si encontro vulnerabilidades retorna 3 bajas, si son 5 media y altas
                    //6 en caso de error
                    if (terraScanExitCode == 3 || terraScanExitCode == 5) {
                        error(message: "TerraScan encontró vulnerabilidades de seguridad")
                        echo "TerraScan encontró vulnerabilidades"
                    } else if (terraScanExitCode == 6) {
                        error("TerraScan falló con un error")
                    }
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
