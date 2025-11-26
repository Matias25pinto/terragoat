pipeline {
    agent any

    stages {

        stage('Checkov') {
            steps {
                script {
                    docker.image('bridgecrew/checkov:latest').inside("--entrypoint=''") {
                        try {
                           sh '''
                                checkov -d . -o cli -o junitxml --output-file-path console,results.xml \
                                --repo-id example/mi-repo --branch main \
                                --skip-check CKV2_AWS_8,CKV_AWS_18,CKV2_AWS_12,CKV2_AZURE_27
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
