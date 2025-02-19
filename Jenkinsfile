pipeline {
    agent any
    
    environment {
        DOCKER_HUB_REPO = 'shrutisharma1152/flask-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                bat "docker build -t %DOCKER_HUB_REPO%:%BUILD_NUMBER% ."
            }
        }
        
        stage('Test') {
            steps {
                // Changed to unittest for your specific test file
                bat "docker run %DOCKER_HUB_REPO%:%BUILD_NUMBER% python -m unittest tests/test_app.py"
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                usernameVariable: 'DOCKER_USER', 
                                                passwordVariable: 'DOCKER_PASS')]) {
                    bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
                    bat "docker push %DOCKER_HUB_REPO%:%BUILD_NUMBER%"
                }
                
                // Added error handling and wait time
                bat """
                    docker stop flask-app || true
                    docker rm flask-app || true
                    timeout 10
                    docker run -d --name flask-app -p 5000:5000 %DOCKER_HUB_REPO%:%BUILD_NUMBER%
                """
            }
        }
    }
    
    post {
        always {
            bat "docker logout"
            // Clean up old images
            bat "docker rmi %DOCKER_HUB_REPO%:%BUILD_NUMBER% || true"
        }
        failure {
            // Stop and remove container if pipeline fails
            bat "docker stop flask-app || true"
            bat "docker rm flask-app || true"
        }
    }
}