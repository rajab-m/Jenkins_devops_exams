pipeline {
    agent any  // Run on any available Jenkins agent

    stages {
        stage('Setup') {
            steps {
                echo 'Setting up environment...'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                // You can add build commands here, e.g.:
                // sh 'make build' or sh 'python setup.py build'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                // You can replace this with real test commands
                // sh 'pytest tests/'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                // This is just a placeholder
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Cleaning up...'
        }
    }
}
