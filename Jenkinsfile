pipeline {
    agent any

    stages {

        stage('Pull Code from GitHub') {
            steps {
                echo 'Cloning source code from GitHub'
                git branch: 'main',
                    url: 'https://github.com/im-devendhar/poc-8.git'
            }
        }

        stage('Code Quality Analysis with SonarQube') {
            steps {
                echo 'Running SonarQube analysis'
                withSonarQubeEnv('sonarqube') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh 'docker build -t poc-8:latest .'
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying application using Docker'
                sh '''
                  docker rm -f poc-8 || true
                  docker run -d -p 8085:80 --name poc-8 poc-8:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline executed successfully'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}

