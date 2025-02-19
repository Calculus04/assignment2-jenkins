pipeline {
    agent any
    
    environment {
        DOCKER_HUB_REPO = 'shrutisharma1152/flask-app'
        REMOTE_HOST = '13.49.75.233'  
        REMOTE_USER = 'ubuntu'        
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
        
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                usernameVariable: 'DOCKER_USER', 
                                                passwordVariable: 'DOCKER_PASS')]) {
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    bat "docker push %DOCKER_HUB_REPO%:%BUILD_NUMBER%"
                }
            }
        }
        
        stage('Deploy to Remote Server') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'remote-server-ssh',
                                                 keyFileVariable: 'SSH_KEY')]) {
                    
                    bat '''
                        icacls %SSH_KEY% /inheritance:r
                        icacls %SSH_KEY% /grant:r "SYSTEM:R"
                    '''
                   
                    bat "scp -o StrictHostKeyChecking=no -i %SSH_KEY% docker-compose.yml %REMOTE_USER%@%REMOTE_HOST%:/app"
                    bat "ssh -o StrictHostKeyChecking=no -i %SSH_KEY% %REMOTE_USER%@%REMOTE_HOST% \"cd /app && docker stop flask-app || true && docker rm flask-app || true && export BUILD_NUMBER=%BUILD_NUMBER% && docker-compose up -d\""
            }
        }
    }
}