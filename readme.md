# Spring PetClinic Sample Application [![Build Status](https://github.com/spring-projects/spring-petclinic/actions/workflows/maven-build.yml/badge.svg)](https://github.com/spring-projects/spring-petclinic/actions/workflows/maven-build.yml)

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/spring-projects/spring-petclinic) [![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=main&repo=7517918)

## Understanding the Spring Petclinic application with a few diagrams

[See the presentation here](https://speakerdeck.com/michaelisvy/spring-petclinic-sample-application)

## Run Petclinic locally

Spring Petclinic is a [Spring Boot](https://spring.io/guides/gs/spring-boot) application built using [Maven](https://spring.io/guides/gs/maven/) or [Gradle](https://spring.io/guides/gs/gradle/). You can build a jar file and run it from the command line (it should work just as well with Java 17 or newer):

```bash
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
./mvnw package
java -jar target/*.jar
```

You can then access the Petclinic at <http://localhost:8080/>.

<img width="1042" alt="petclinic-screenshot" src="https://cloud.githubusercontent.com/assets/838318/19727082/2aee6d6c-9b8e-11e6-81fe-e889a5ddfded.png">

Or you can run it from Maven directly using the Spring Boot Maven plugin. If you do this, it will pick up changes that you make in the project immediately (changes to Java source files require a compile as well - most people use an IDE for this):

```bash
./mvnw spring-boot:run
```

> NOTE: If you prefer to use Gradle, you can build the app using `./gradlew build` and look for the jar file in `build/libs`.
---

# Spring PetClinic Application Deployment Guide
## Table of Contents
- [Prerequisites](#prerequisites)
- [1. Repository Setup](#1-repository-setup)
- [2. EKS Cluster Setup](#2-eks-cluster-setup)
- [3. Jenkins Installation](#3-jenkins-installation)
- [4. Tools and Plugins Configuration](#4-tools-and-plugins-configuration)
- [5. Docker Setup](#5-docker-setup)
- [6. Trivy Installation](#6-trivy-installation)
- [7. Kubernetes Configuration](#7-kubernetes-configuration)
- [8. Helm Charts](#8-helm-charts)
- [9. MySQL Deployment](#9-mysql-deployment)
- [10. CI/CD Pipeline](#10-cicd-pipeline)
- [11. Post-deployment Verification](#11-post-deployment-verification)
- [12. Troubleshooting Guide](#12-troubleshooting-guide)

## Prerequisites
- AWS Account with appropriate permissions
- AWS CLI installed and configured
- kubectl installed
- Helm installed
- Git installed

## 1. Repository Setup
```bash
# Clone the repository
git clone https://github.com/SubbuTechOps/spring-petclinic.git
cd spring-petclinic

# Verify the contents
ls -la
```

## 2. EKS Cluster Setup
# EKS Cluster Setup Guide

This document provides access to the comprehensive guide for setting up an Amazon EKS (Elastic Kubernetes Service) cluster.

## Guide Access

📚 [EKS Cluster Setup Guide](https://drive.google.com/file/d/1mouXxkZ6kYjeL5KtRJp9BK6SLNTZOYEr/view?usp=sharing)

## Contents Overview

This guide includes:
* EKS cluster creation steps
* Networking configuration
* Security group setup
* Node group management
* Access control configuration
* Best practices and recommendations

## Jenkins Server Installation Script

📚 [Download Jenkins Installation Script](https://drive.google.com/file/d/1wxs6RNi8qInij8WUtwIUzYETtCaDAtfE/view?usp=sharing)

This shell script automates the installation and configuration of Jenkins server and required tools.

### Script Usage
```bash
chmod +x install-jenkins.sh
sudo ./install-jenkins.sh
```

### Components Installed:
* Essential system utilities (wget, curl, unzip, etc.)
* Jenkins server
* Docker
* kubectl
* AWS CLI
* Trivy scanner
* Helm
* Required permissions and configurations

### Access Details:
* URL: http://YOUR_SERVER_IP:8080
* Default Port: 8080
* Initial Admin Password: Found in script output or at `/var/lib/jenkins/secrets/initialAdminPassword`

### Configure kubectl
```bash
aws eks update-kubeconfig --name your-cluster-name --region your-region
```

## SonarQube Server Installation Script

📚 [Download SonarQube Installation Script](https://drive.google.com/file/d/1pl3PxQx9urAapolsf5KM94JoYVMInBE2/view?usp=sharing)

This shell script automates the installation and configuration of SonarQube server.

### Script Usage
```bash
chmod +x install-sonarqube.sh
sudo ./install-sonarqube.sh
```

### Components and Configurations:
* SonarQube server (Latest LTS version)
* Dedicated sonar user
* Systemd service for automatic startup
* System limits and requirements
* Firewall rules

### Default Access:
* URL: http://YOUR_SERVER_IP:9000
* Default credentials: admin/admin
* Default port: 9000

### Verification Commands:
```bash
# Check service status
sudo systemctl status sonarqube

# View logs
sudo tail -f /opt/sonarqube/logs/sonar.log

# Check if port is listening
sudo netstat -tlpn | grep 9000
```

### Service Management:
```bash
# Stop SonarQube
sudo systemctl stop sonarqube

# Start SonarQube
sudo systemctl start sonarqube

# Restart SonarQube
sudo systemctl restart sonarqube

# Check logs
sudo journalctl -u sonarqube -f
```

### System Requirements:
* Minimum 2GB RAM
* 1GB free space
* Java 11 or higher


## 4. Tools and Plugins Configuration
### Required Jenkins Plugins
- Docker Pipeline
- Kubernetes CLI
- GitHub Integration
- Pipeline AWS Steps
- CloudBees AWS Credentials

### Configure Credentials
1. Navigate to Jenkins > Manage Jenkins > Manage Credentials
2. Add the following credentials:
   - GitHub credentials
   - Docker Hub credentials
   - AWS credentials
   - Kubernetes configuration


## 7. Kubernetes Configuration
### Create ConfigMap
```yaml
# petclinic-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: petclinic-config
data:
  application.properties: |
    spring.profiles.active=mysql
    spring.datasource.url=jdbc:mysql://mysql:3306/petclinic
    spring.datasource.initialization-mode=always
```

### Create Secrets
```yaml
# petclinic-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
data:
  username: cGV0Y2xpbmlj
  password: cGV0Y2xpbmljX3Bhc3N3b3Jk
```

## 8. Helm Charts
### Create Helm Chart Structure
```bash
helm create petclinic
```

### Update values.yaml
```yaml
# petclinic/values.yaml
image:
  repository: your-docker-hub-username/spring-petclinic
  tag: latest
  pullPolicy: Always

service:
  type: LoadBalancer
  port: 8080

resources:
  limits:
    cpu: 1000m
    memory: 1024Mi
  requests:
    cpu: 500m
    memory: 512Mi

configMap:
  name: petclinic-config
```

## 9. MySQL Deployment
```yaml
# mysql-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-credentials
              key: password
        ports:
        - containerPort: 3306
```

## 10. CI/CD Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'your-docker-hub-username/spring-petclinic'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/SubbuTechOps/spring-petclinic.git'
            }
        }
        
        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }
        
        stage('Test') {
            steps {
                sh './mvnw test'
            }
        }
        
        stage('Security Scan') {
            steps {
                sh "trivy image ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.withRegistry('', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "helm upgrade --install petclinic ./petclinic \
                        --set image.tag=${DOCKER_TAG} \
                        --namespace petclinic"
                }
            }
        }
    }
}
```

## 11. Post-deployment Verification
```bash
# Check deployment status
kubectl get deployments -n petclinic

# Check pods
kubectl get pods -n petclinic

# Check services
kubectl get svc -n petclinic

# View logs
kubectl logs -f deployment/petclinic -n petclinic

# Test the application
curl http://$(kubectl get svc petclinic -n petclinic -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'):8080
```

## 12. Troubleshooting Guide

### Common Issues and Solutions

1. **Jenkins Pipeline Failures**
   - Issue: Maven build fails
   - Solution: Check Java version compatibility and Maven settings

2. **Docker Issues**
   - Issue: Permission denied
   - Solution: Ensure Jenkins user is in docker group
   ```bash
   sudo usermod -aG docker jenkins
   sudo service jenkins restart
   ```

3. **Kubernetes Deployment Issues**
   - Issue: Pods not starting
   - Solution: Check pod events and logs
   ```bash
   kubectl describe pod <pod-name> -n petclinic
   kubectl logs <pod-name> -n petclinic
   ```

4. **Database Connection Issues**
   - Issue: Application can't connect to MySQL
   - Solution: Verify MySQL service and credentials
   ```bash
   kubectl exec -it <mysql-pod> -- mysql -u root -p
   ```

### Health Check Commands
```bash
# Check node status
kubectl get nodes

# Check pod health
kubectl get pods -n petclinic -o wide

# Check logs
kubectl logs -f deployment/petclinic -n petclinic

# Check service endpoints
kubectl get endpoints -n petclinic
```

Remember to replace placeholder values such as `your-region`, `your-docker-hub-username`, and adjust resource limits based on your requirements.

For additional support or specific error resolution, consult the project's GitHub issues or create a new issue with detailed information about the problem encountered.
