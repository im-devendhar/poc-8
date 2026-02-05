
***

# CI/CD Pipeline Troubleshooting Notes

### Jenkins + SonarQube + Docker (Project: poc‑8)

This document captures all the troubleshooting steps and fixes implemented while building the CI/CD pipeline using Jenkins, SonarQube, Docker, and GitHub.

It serves as a personal reference for future projects and interview preparation.

***

## 1. Jenkins Could Not Detect SonarScanner Tool

**Error:**

    ERROR: No tool named POC-8-scan found

**Cause:**  
The SonarScanner tool name configured in Jenkins did not match the name used in the Jenkinsfile.

**Fix:**  
Updated Jenkinsfile with the exact name used in Jenkins Tools:

```groovy
SONAR_SCANNER = "POC-8-Scan"
```

***

## 2. Wrong SonarQube Server Name

**Error:**  
`withSonarQubeEnv("sonarqube") not found`

**Cause:**  
The name configured in Jenkins → Configure System for SonarQube server was `Sonar-Server`, not `sonarqube`.

**Fix:**  
Updated Jenkinsfile:

```groovy
SONARQUBE_SERVER = "Sonar-Server"
```

***

## 3. Unauthorized SonarQube Analysis

**Error:**

    You're not authorized to analyze this project

**Causes:**

*   Jenkinsfile used the wrong project key.
*   Token was linked to a different project.
*   Token did not have "Execute Analysis" permission.

**Fixes:**

*   Confirmed actual project key from the URL:
        /dashboard?id=POC-8-DevOps
*   Updated Jenkinsfile:
    ```groovy
    SONAR_PROJECT_KEY = "POC-8-DevOps"
    ```
*   Deleted the old token and generated a new one.
*   Provided "Execute Analysis" permission for the token user.

***

## 4. Workspace Directory Contained Spaces

Example path:

    /var/lib/jenkins/workspace/POC -8 basic CICD workflow with Sonarqube integration

**Impact:**  
Potential for scan issues, though scanner still worked.  
No change required, but recommended to avoid spaces in job names in future.

***

## 5. Quality Gate Stage Not Required

The Quality Gate step was removed to keep the pipeline simple and avoid waiting for webhook events.

***

## 6. SonarQube Warning: Outdated baseline-browser-mapping

**Warning:**

    The data in this module is over two months old.

**Fix:**  
Optional. Could be resolved with:

```bash
npm i baseline-browser-mapping@latest -D
```

***

## 7. Git Installation Warning

**Warning:**

    Selected Git installation does not exist. Using Default

**Fix:**  
No action needed since system Git was available and functional.

***

## 8. Final Working Jenkinsfile (Clean Version)

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "poc-8"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        CONTAINER  = "poc-8"

        SONARQUBE_SERVER = "Sonar-Server"
        SONAR_SCANNER    = "POC-8-Scan"
        SONAR_PROJECT_KEY = "POC-8-DevOps"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/im-devendhar/poc-8.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scanner = tool name: SONAR_SCANNER, type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh """
                            ${scanner}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker rm -f ${CONTAINER} || true
                    docker run -d -p 8085:80 --name ${CONTAINER} ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success { 
            echo "Pipeline completed successfully" 
        }
        failure { 
            echo "Pipeline failed" 
        }
    }
}
```

***

## 9. Final Outcome

*   SonarQube analysis completed successfully.
*   Docker image built and tagged with build numbers.
*   Application deployed on port 8085 of the EC2 instance.
*   CI/CD pipeline completed with no errors.

***


