pipeline {
    agent {
        docker {
            image 'python:3.9-slim'
        }
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
        
        stage('Test') {
            steps {
                sh 'python -m pytest'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building...'
                // Add your build steps here
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline Succeeded!'
        }
        failure {
            echo 'Pipeline Failed!'
        }
    }
}