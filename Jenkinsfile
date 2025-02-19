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
                bat "docker run %DOCKER_HUB_REPO%:%BUILD_NUMBER% python -m unittest tests/test_app.py"
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                usernameVariable: 'DOCKER_USER', 
                                                passwordVariable: 'DOCKER_PASS')]) {
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    bat "docker push %DOCKER_HUB_REPO%:%BUILD_NUMBER%"
                }
                
                bat "docker run -d -p 5000:5000 %DOCKER_HUB_REPO%:%BUILD_NUMBER%"
            }
        }
    }
    
    post {
        always {
            bat 'docker logout'
        }
    }
}