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
