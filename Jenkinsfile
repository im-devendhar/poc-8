pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    environment {
        // Docker settings
        IMAGE_NAME = "poc-8"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        CONTAINER  = "poc-8"
        APP_PORT   = "8085"

        // SonarQube (must match Jenkins configuration names)
        SONARQUBE_SERVER = "Sonar-Server"   // Manage Jenkins -> Configure System -> SonarQube servers -> Name
        SONAR_SCANNER    = "POC-8-Scan"      // Manage Jenkins -> Tools -> SonarQube Scanner installations -> Name

        // SonarQube project (IMPORTANT: use the real existing Project Key)
        // Put the exact key shown in SonarQube dashboard URL: .../dashboard?id=<KEY>
        SONAR_PROJECT_KEY  = "POC-8-DevOps"
        SONAR_PROJECT_NAME = "POC-8-DevOps"

        // If your repo has spaces in workspace path, scanner still works, but this is safe:
        SONAR_SOURCES = "."
    }

    stages {

        stage('Checkout (GitHub)') {
            steps {
                echo "Checking out source code from GitHub..."
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: 'https://github.com/im-devendhar/poc-8.git']]
                ])
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo "Running SonarQube analysis..."
                script {
                    // Explicit type improves tool resolution reliability
                    def scannerHome = tool name: "${SONAR_SCANNER}", type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh """
                          ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                            -Dsonar.sources=${SONAR_SOURCES}
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
                echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy (Docker)') {
            steps {
                echo "Deploying application container on port ${APP_PORT}..."
                sh """
                  docker rm -f ${CONTAINER} || true
                  docker run -d --restart unless-stopped \
                    -p ${APP_PORT}:80 \
                    --name ${CONTAINER} \
                    ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Cleanup (Optional)') {
            steps {
                echo "Cleaning unused Docker images..."
                sh "docker image prune -f || true"
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline executed successfully!"
            echo "App should be available at: http://<JENKINS_SERVER_PUBLIC_IP>:${APP_PORT}"
        }
        failure {
            echo "❌ Pipeline failed. Please check the console output logs."
        }
        always {
            echo "Pipeline finished at: ${new Date()}"
        }
    }
}
``
