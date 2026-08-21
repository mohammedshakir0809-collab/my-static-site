pipeline {
    agent any

    environment {
        IMAGE_NAME = "shakir0809/my-static-site"
        CONTAINER_NAME = "my-static-site"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker') {
            steps {
                bat 'where docker'
                bat 'docker --version'
                bat 'docker info'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                    docker build -t %IMAGE_NAME%:%BUILD_NUMBER% -t %IMAGE_NAME%:latest .
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'ef099e221526488ebf79929a62d76f5f',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat """
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin

                        docker push %IMAGE_NAME%:%BUILD_NUMBER%
                        docker push %IMAGE_NAME%:latest
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                bat """
                    docker pull %IMAGE_NAME%:latest

                    docker stop %CONTAINER_NAME% 2>nul || exit /b 0
                    docker rm %CONTAINER_NAME% 2>nul || exit /b 0

                    docker run -d -p 84:80 --restart unless-stopped --name %CONTAINER_NAME% %IMAGE_NAME%:latest
                """
            }
        }
    }

    post {
        always {
            bat 'docker logout || exit /b 0'
        }
    }
}