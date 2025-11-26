pipeline {
    agent any

    stages {

        stage('Checkov') {
            steps {
                script {
                    docker.image('bridgecrew/checkov:latest').inside("--entrypoint=''") {
                        try {
                           sh '''
                                checkov --soft-fail -o cli -o json --output-file-path console,results.json -d .
                            '''
                            junit skipPublishingChecks: true, testResults: 'results.xml'
                        } catch (err) {
                            junit skipPublishingChecks: true, testResults: 'results.xml'
                            throw err
                        }
                    }
                }
            }
        }
        stage('Compilation') {
            steps {
                sh 'echo "Compilación..."'
            }
        }
    }

    options {
        timestamps()
    }
}
