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
                bat "docker run %DOCKER_HUB_REPO%:%BUILD_NUMBER% python -m pytest"
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                usernameVariable: 'DOCKER_USER', 
                                                passwordVariable: 'DOCKER_PASS')]) {
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "docker push %DOCKER_HUB_REPO%:%BUILD_NUMBER%"
                }
                
                // Run the new container
                bat """
                    docker stop flask-app || true
                    docker rm flask-app || true
                    docker run -d --name flask-app -p 5000:5000 %DOCKER_HUB_REPO%:%BUILD_NUMBER%
                """
            }
        }
    }
    
    post {
        always {
            bat "docker logout"
        }
    }
}