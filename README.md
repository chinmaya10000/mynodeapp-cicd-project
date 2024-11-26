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
- Install Necessary Plugins in Jenkins:
- <b>Go to Jenkins Master and click on <mark> Manage Jenkins --> Plugins --> Available plugins</mark> install the below plugins:</b>
  - OWASP Dependency-Check
  - NodeJs
  - SonarQube Scanner
  - Pipeline: Stage View

