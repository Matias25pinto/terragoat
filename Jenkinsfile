pipeline {
    agent any

    stages {

        stage('Checkov') {
            steps {
                script {
                    docker.image('bridgecrew/checkov:latest').inside("--entrypoint=''") {
                        try {
                           sh '''
                                checkov -d . \
                                -o cli -o junitxml \
                                --output-file-path console,results.xml \
                                --repo-id example/mi-repo \
                                --branch main
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
