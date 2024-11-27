#### Node.js Application with CI/CD Pipeline

This repository contains a Node.js application along with a Jenkins CI/CD pipeline to automate the process from code checkout to deployment.

## Setup Instructions:

### 1. Server Provisioning
- I am using AWS as the Cloud Provider
- Provision an EC2 instance on AWS with a secure Linux distribution (e.g., Amazon Linux 2023).
- Configure security groups to allow necessary incoming traffic and restrict unnecessary access.
- Generate and configure SSH key pairs for secure remote acces.

### 2. Install Docker and run Jenkins as a docker container
- Install Docker
```
sudo yum install docker -y
sudo usermod -aG docker ec2-user
sudo systemctl enable --now docker
```
- Run Jenkins as a docker container
```
docker run -p 8080:8080 -p 50000:50000 -d \
                    -v jenkins_home:/var/jenkins_home \
                    -v /var/run/docker.sock:/var/run/docker.sock \
                    -v $(which docker):/usr/bin/docker jenkins/jenkins:lts
```
- Go to Inside Jenkins Container as a root user
```
docker exec -u 0 -it <container-id> bash
```
- Give the permission to /var/run/docker.sock
```
chmod 666 /var/run/docker.sock
```
- Run SonarQube as a docker container and access in browser
```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
---
publicip:9000
```
### 3. Access Jenkins in web browser using publicip of your EC2 instance. Install some plugin and configure in Global tool configuration
- Access Jenkins
```
publicip:8080
```
## Prerequisites

- Jenkins installed with required plugins:
  - **Pipeline**
  - **Git**
  - **SonarQube Scanner**
  - **Slack Notification**
  - **SSH Agent**
- Access to the following tools:
  - Docker
  - Node.js
  - Git
  - SonarQube server
  - Slack webhook
- Credentials setup in Jenkins:
  - **Docker Registry**: `docker-cred`
  - **Staging Server**: `staging-server-credentials`
- A staging server with Docker and Docker Compose installed.
- SSH access to the staging server.

---

## Environment Variables

The pipeline uses the following environment variables:

| Variable         | Description                                     |
|------------------|-------------------------------------------------|
| `IMAGE_NAME`     | Name of the Docker image.                      |
| `IMAGE_TAG`      | Tag for the Docker image.                      |
| `SCANNER_HOME`   | Path to the SonarQube scanner installation.    |
| `SEMGREP_RULES`  | Ruleset for Semgrep analysis.                  |
| `SLACK_CHANNEL`  | Slack channel for notifications.               |

---

## Pipeline Stages Details

### Clone Repo
- **Description**: Clones the main branch of the repository.
- **Command**: `git branch: 'main', url: '<repo-url>'`

### Secret Scanning (Gitleaks)
- **Description**: Uses Gitleaks to scan for sensitive information.
- **Output**: Generates a `gitleaks-report.json`.

### Install Dependencies
- **Description**: Installs Node.js dependencies using `npm install`.
- **Path**: `app` folder.

### Run Tests
- **Description**: Executes the application's test suite using `npm test`.
- **Path**: `app` folder.

### SonarQube Analysis (SAST)
- **Description**: Performs static code analysis using SonarQube.
- **Command**: `sonar-scanner`

### Quality Gate Check
- **Description**: Waits for SonarQube quality gate results.
- **Timeout**: 1 hour.

### Run Semgrep
- **Description**: Scans the codebase for vulnerabilities using Semgrep.
- **Tool**: Dockerized Semgrep.

### Run Retire.js (SCA)
- **Description**: Scans for outdated libraries and security issues.
- **Tool**: Retire.js.

### Build Image
- **Description**: Builds the Docker image with the application code.
- **Command**: `docker build`

### Image Security Scan
- **Description**: Scans the Docker image for vulnerabilities using Trivy.
- **Output**: Generates a `trivy.json`.

### Push Image
- **Description**: Pushes the Docker image to the registry.

### Deploy to Staging
- **Description**: Deploys the Docker container to a staging server.
- **Commands**:
  - Copies deployment files to the server.
  - Executes a shell script to deploy the application.

---

## Deployment Details

- **Server IP**: `3.129.42.205`
- **Files Transferred**:
  - `server-cmds.sh`: Deployment script.
  - `docker-compose.yaml`: Docker Compose configuration.

---

## Notifications

- Slack notifications are sent to the specified channel:
  - **Success**: `Job completed successfully.`
  - **Failure**: `Job failed.`

---

## Troubleshooting

1. **Pipeline Fails in Gitleaks**:
   - Ensure Gitleaks is installed and configured correctly.
   - Check the `gitleaks-report.json` for details.

2. **Docker Image Not Pushed**:
   - Verify Docker credentials are set up in Jenkins.

3. **Deployment Fails**:
   - Ensure SSH access is properly configured.
   - Verify `docker-compose.yaml` and `server-cmds.sh` are correctly implemented.

