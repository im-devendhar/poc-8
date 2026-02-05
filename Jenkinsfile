pipeline {
    agent any

    environment {
        IMAGE_NAME       = "poc-8"
        IMAGE_TAG        = "${BUILD_NUMBER}"
        CONTAINER        = "poc-8"

        // Must match: Manage Jenkins -> Configure System -> SonarQube servers -> Name
        SONARQUBE_SERVER = "Sonar-Server"

        // Must match: Manage Jenkins -> Tools -> SonarQube Scanner installations -> Name
        SONAR_SCANNER    = "POC-8-Scan"
    }

    stages {

        stage('Pull Code from GitHub') {
            steps {
                echo 'Cloning source code from GitHub'
                git branch: 'main', url: 'https://github.com/im-devendhar/poc-8.git'
            }
        }

        stage('Code Quality Analysis with SonarQube') {
            steps {
                echo 'Running SonarQube analysis'
                script {
                    def scannerHome = tool "${SONAR_SCANNER}"
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh """
                          ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=poc-8 \
                          -Dsonar.projectName=poc-8 \
                          -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Waiting for SonarQube Quality Gate result..."
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying application using Docker'
                sh """
                  docker rm -f ${CONTAINER} || true
                  docker run -d -p 8085:80 --name ${CONTAINER} ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Cleanup Old Images (Optional)') {
            steps {
                echo "Cleaning unused Docker images"
                sh "docker image prune -f"
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
