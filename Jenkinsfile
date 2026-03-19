pipeline {
    // We use a docker agent with Python 3.10. 
    // You can also use 'agent any' if your Jenkins server has Python 3.9+ directly installed.
    agent {
        docker { 
            image 'python:3.10' 
            args '-u root:root' // Needed for some Jenkins setups to have permission to workspace
        }
    }

    environment {
        POETRY_VERSION = "2.2.1"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install System Dependencies') {
            steps {
                sh '''
                    python3 -m pip install --upgrade pip
                    python3 -m pip install poetry==${POETRY_VERSION} build twine tox
                '''
            }
        }

        stage('Build Package') {
            steps {
                sh '''
                    poetry install --only build
                    poetry run python -m build
                    poetry run twine check --strict dist/*
                '''
            }
        }

        stage('Install Dev Dependencies') {
            steps {
                sh '''
                    poetry config virtualenvs.in-project true
                    poetry install --only=dev
                '''
            }
        }

        stage('Type Checking (mypy)') {
            steps {
                sh '''
                    source .venv/bin/activate
                    tox -e mypy
                '''
            }
        }

        stage('Test with Tox') {
            steps {
                sh '''
                    source .venv/bin/activate
                    // Testing using Python 3.10 tox factor as an example
                    tox run-parallel -f py3.10 --parallel 4 --installpkg dist/*.whl
                '''
            }
        }
    }

    post {
        always {
            // Optional: Archive pytest coverage or testing results if you configure pytest to output junit xml
            archiveArtifacts artifacts: 'dist/*', allowEmptyArchive: true
        }
        success {
            echo "Tests passed successfully!"
        }
        failure {
            echo "pipeline failed. Please check the logs."
        }
    }
}
