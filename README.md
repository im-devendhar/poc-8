<img width="272" height="117" alt="{51174B3B-1668-4352-B53F-3389BD2815B6}" src="https://github.com/user-attachments/assets/08c3ebf0-df6e-4047-99c8-e71164e78182" />

***

# Setup Steps

This section documents the exact steps followed to set up the CI/CD pipeline using **Jenkins, SonarQube, Docker, and GitHub** on an EC2 instance for the `poc‑8` project.

***

## 1. Launch EC2 Instance

*   Launched an Ubuntu-based EC2 instance.
*   Installed required packages:
    ```bash
    sudo apt update
    sudo apt install -y openjdk-17-jdk docker.io git unzip
    ```
*   Added Jenkins/Docker permissions later as part of the pipeline setup.

***

## 2. Install and Configure Jenkins

*   Installed Jenkins on EC2.
*   Enabled and started the Jenkins service:
    ```bash
    sudo systemctl enable jenkins
    sudo systemctl start jenkins
    ```
*   Opened port `8080` in EC2 Security Group.
*   Accessed Jenkins in browser:
        http://<EC2_PUBLIC_IP>:8080
*   Followed the onboarding steps and installed recommended plugins.

***

## 3. Install Docker and Give Jenkins Access

*   Installed Docker:
    ```bash
    sudo apt install docker.io -y
    ```
*   Added Jenkins user to Docker group:
    ```bash
    sudo usermod -aG docker jenkins
    sudo systemctl restart jenkins
    ```
*   Verified Docker access:
    ```bash
    docker ps
    ```

***

## 4. Install and Configure SonarQube

*   Launched SonarQube on a separate EC2 instance using Docker:
    ```bash
    docker run -d -p 9000:9000 --name sonarqube sonarqube:latest
    ```
*   Opened port `9000` in EC2 Security Group.
*   Accessed SonarQube dashboard:
        http://<SONARQUBE_IP>:9000
*   Logged in with default credentials:
        username: admin  
        password: admin
*   Updated admin password.

***

## 5. Create SonarQube Project

*   Created new project in SonarQube: **POC-8-DevOps**.
*   Copied the project key from the URL:
        /dashboard?id=POC-8-DevOps
*   Created a **new token** under:  
    `User Account → Security → Generate Tokens`
*   Stored this token for Jenkins integration.

***

## 6. Add SonarQube Server in Jenkins

*   Go to `Manage Jenkins → Configure System`.
*   Under **SonarQube Servers**, added:
    *   Name: `Sonar-Server`
    *   Server URL: `http://<SONARQUBE_IP>:9000`
    *   Authentication Token: Added via Jenkins Credentials.

***

## 7. Add SonarScanner in Jenkins

*   Go to `Manage Jenkins → Tools → SonarQube Scanner`.
*   Added scanner with name:
        POC-8-Scan
*   Enabled “Install automatically”.

***

## 8. Create Jenkins Credentials for SonarQube

*   Go to `Manage Jenkins → Credentials → Global → Add Credentials`.
*   Selected **Secret Text**, pasted the SonarQube token.
*   Gave it an ID:
        poc-8-devops-token
*   Mapped this credential in SonarQube server configuration.

***

## 9. Create Jenkins Pipeline Job

*   Created a **Pipeline type** job in Jenkins.
*   Connected it to GitHub repository: `https://github.com/im-devendhar/poc-8`.
*   Pointed Jenkins to use the `Jenkinsfile` inside the repository.

***

## 10. Configure Jenkinsfile

*   Set the environment parameters correctly:
    *   Correct SonarQube Server name
    *   Correct SonarScanner name
    *   Correct project key
    *   Docker image name and container name
*   Removed Quality Gate stage to simplify the pipeline.

***

## 11. Run Pipeline

The pipeline performs the following stages:

1.  Checkout code from GitHub
2.  Run SonarQube analysis
3.  Build Docker image
4.  Deploy Docker container on the same EC2 instance

After final fixes, the pipeline executed successfully.

***

## 12. Application Access

*   Application is deployed using Nginx container.
*   Exposed through port `8085`:
        http://<JENKINS_EC2_PUBLIC_IP>:8085

***


