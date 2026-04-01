pipeline {
    // Use controller/agent label; run steps inside a Docker container when needed.
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/p1neapplechoco/demo_test_ci-main'
            }
        }

        stage('Install, Lint, Test') {
            steps {
                script {
                    docker.image('python:3.10-slim').inside('-u') {
                        sh '''
                            python -m pip install --upgrade pip
                            if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
                            pip install ruff pytest coverage
                        '''

                        // Ruff outputs GitHub format for inline annotations if supported by Jenkins plugins
                        sh 'ruff --format=github --target-version=py310 .'

                        sh '''
                            coverage run -m pytest -v -s --junitxml=junit-pytests.xml
                            coverage report -m
                            coverage xml -o coverage.xml
                        '''
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '.ruff_cache/**', allowEmptyArchive: true
                    junit allowEmptyResults: true, testResults: '**/junit*.xml'
                    publishCoverage adapters: [coberturaAdapter('coverage.xml')], failNoReports: false
                }
            }
        }
    }
}
