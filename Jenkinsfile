pipeline {
    agent any
    stages {
        stage('Security Scan') {
            steps {
                sh '''
                checkov -d . -o cli -o json \
                  --output-file-path console,results.json \
                  --repo-id example/mi-repo --branch main
                '''
            }
        }
    }
    post {
        always {
            // Archivar el JSON
            archiveArtifacts 'results.json'
        }
    }
}
