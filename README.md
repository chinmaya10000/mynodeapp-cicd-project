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
- Run SonarQube as a docker container and access in browser
```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
---
publicip:9000
```