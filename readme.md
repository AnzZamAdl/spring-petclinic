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
- [4. AWS Credentials Configuration](#4-aws-credentials-configuration)
- [5. SonarQube Server Installation](#5-sonarqube-server-installation)
- [6. Tools and Plugins Configuration](#6-tools-and-plugins-configuration)
- [7. Kubernetes Configuration](#7-kubernetes-configuration)
- [8. Helm Charts](#8-helm-charts)
- [9. CI/CD Pipeline](#9-cicd-pipeline)
- [10. Post-deployment Verification](#10-post-deployment-verification)
- [11. Troubleshooting Guide](#11-troubleshooting-guide)

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
---

## 2. EKS Cluster Setup
### EKS Cluster Setup Guide

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

---

## 3. Jenkins Installation
### Jenkins Server Installation Script

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
---

## 4. AWS Credentials Configuration

### Generate Access Keys
1. Log in to AWS Management Console
2. Navigate to IAM (Identity and Access Management)
3. Select your IAM user
4. Go to "Security credentials" tab
5. Click "Create access key"
6. Download and securely store the credentials file

### Configure AWS CLI
```bash
# Configure AWS CLI with your credentials
aws configure

AWS Access Key ID [None]: <Your-Access-Key>
AWS Secret Access Key [None]: <Your-Secret-Key>
Default region name [None]: ap-south-1
Default output format [None]: table

# Verify AWS CLI configuration
aws sts get-caller-identity
```

### Configure kubectl for EKS on Jenkins Server
```bash
# List available EKS clusters
aws eks list-clusters

# Update kubeconfig for your EKS cluster
aws eks update-kubeconfig --name <your-cluster-name> --region <your-cluster-region>

# Verify kubeconfig
cat ~/.kube/config

# Test kubectl configuration
kubectl get nodes
kubectl get ns
```
---
## 5. SonarQube Server Installation
### SonarQube Server Installation Script

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

### Create PetClinic Project
1. Login to SonarQube (http://YOUR_SERVER_IP:9000)
2. Navigate to "Projects" → "Create Project"
3. Select "Manual" setup
4. Fill in project details:
   - Project key: `PetClinic`
   - Display name: `PetClinic`
   - Main branch name: `main`

### Generate Authentication Token
1. Go to User → My Account → Security
2. Generate a new token:
   - Token name: `jenkins-integration`
   - Expiration: Choose appropriate expiration (e.g., 30 days)
3. Copy and securely store the generated token

### Configure Project Permissions
1. Go to Administration → Security → Users
2. Click on your user
3. Under "Permissions", ensure the following are enabled:
   - Execute Analysis
   - Create Projects
   - Create Applications
   - Create Portfolios
----
### Configure Jenkins Integration

#### Add SonarQube Token to Jenkins
1. Navigate to Jenkins → Manage Jenkins → Credentials → System
2. Click on "Global credentials" → "Add Credentials"
3. Configure the credentials:
   ```
   Kind: Secret text
   Scope: Global
   Secret: <Your-SonarQube-Token>
   ID: sonar-credentials
   Description: SonarQube Authentication Token
   ```
#### Configure SonarQube Server in Jenkins
1. Navigate to Jenkins Dashboard → Manage Jenkins → System
2. Scroll down to the "SonarQube servers" section
3. Configure the following settings:
   - Check "Environment variables" if you want administrators to inject SonarQube configuration as environment variables
   - Under "SonarQube installations":
     - Name: `SonarQube`
     - Server URL: `http://3.92.82.78:9000` (replace with your SonarQube server IP)
     - Server authentication token: 
       - Click "Add" → Select "Jenkins"
       - Kind: Secret text
       - Secret: Paste your SonarQube authentication token
       - ID: `sonar-credentials`
       - Description: "SonarQube Authentication Token"
       - Click "Add"
     - Select the created credential from the dropdown
   - Click "Advanced" if you need to configure additional settings
   - Click "Save" or "Apply" to preserve the configuration

### Verify SonarQube Configuration
1. After saving the configuration, return to the Jenkins dashboard
2. Create a test pipeline job
3. Add the following environment variables in your Jenkinsfile:
```groovy
environment {
    SONARQUBE_HOST_URL = 'http://<sonar-ip>:9000'
    SONARQUBE_PROJECT_KEY = 'PetClinic'
    SONARQUBE_TOKEN = credentials('sonar-credentials')
}
```
### Troubleshooting
```bash
# Check SonarQube logs
sudo tail -f /opt/sonarqube/logs/sonar.log

# Verify SonarQube service status
sudo systemctl status sonarqube

# Check Jenkins logs for integration issues
sudo tail -f /var/log/jenkins/jenkins.log

# Test SonarQube connectivity from Jenkins
curl -v http://<sonar-ip>:9000
```
---

## 6. Tools and Plugins Configuration
### Essential Jenkins Plugins
1. **Pipeline and SCM**
   - Pipeline
   - Git
   - GitHub Integration

2. **Build Tools**
   - Maven Integration
   - JDK Tool

3. **Docker & Kubernetes**
   - Docker
   - Docker Pipeline
   - Kubernetes
   - Kubernetes CLI

4. **SonarQube**
   - SonarQube Scanner
   - Quality Gates

5. **Credentials**
   - Credentials Binding
   - Credentials

## Installation Steps
1. Go to Jenkins Dashboard → Manage Jenkins → Plugins → Available plugins
2. Search and install the above plugins
3. Select "Install without restart"
4. Restart Jenkins after installation completes

## Verification
After restart, verify in Manage Jenkins → Plugins → Installed plugins that all required plugins are active.


### Configure Credentials
1. Navigate to Jenkins > Manage Jenkins > Manage Credentials
2. Add the following credentials:
   - GitHub credentials
   - Docker Hub credentials
   - AWS credentials
   - SonarQube Token
   - Kubernetes configuration

---

## 7. Kubernetes Configuration
### Create ConfigMap
```yaml
# app-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: petclinic-dev
data:
  MYSQL_DATABASE: petclinic
  MYSQL_URL: jdbc:mysql://mysql-service:3306/petclinic
```

### Create Secrets
```yaml
# db-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secrets
  namespace: petclinic-dev
type: Opaque
data:
  MYSQL_USER: cGV0Y2xpbmlj          # Base64 encoded value for 'petclinic'
  MYSQL_PASSWORD: cGV0Y2xpbmlj      # Base64 encoded value for 'petclinic'
  MYSQL_ROOT_PASSWORD: cm9vdA==     # Base64 encoded value for 'root'
```

### Create PersistentVolume(PV)
```yaml
# mysql-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  awsElasticBlockStore:
    volumeID: vol-00ad22c086bb5817f
    fsType: ext4
```
**Note:**
### Guide: Setting Up EBS Volumes for Kubernetes PersistentVolumes
[EBS Volume Setup Guide](https://github.com/SubbuTechOps/storages-guide/blob/main/EBS/ebs-pv-guide.md)

---

## 8. Helm Charts

### Directory Structure
```
petclinic-chart/
├── templates/
│   ├── deployment-app.yaml
│   ├── deployment-db.yaml
│   ├── pvc.yaml
│   ├── service-app.yaml
│   └── service-db.yaml
├── Chart.yaml
└── values.yaml
```

#### Check the following directory for all charts used in the project:
- 🗂️ Petclinic Chart Directory - Contains main chart configuration and values
- 📑 Templates Directory - Contains Kubernetes manifests and service definitions

[Petclinic Chart Directory](https://github.com/SubbuTechOps/spring-petclinic/tree/develop/petclinic-chart)

[Templates Directory](https://github.com/SubbuTechOps/spring-petclinic/tree/develop/petclinic-chart/templates)

---

## 9. CI/CD Pipeline

🔄 Jenkins Pipeline - Contains CICD pipeline configuration and build steps:

[Jenkins Pipeline File](https://github.com/SubbuTechOps/spring-petclinic/blob/develop/Jenkinsfile)


---

## 10. Post-deployment Verification

📋 Deployment Resources:

* 🔍 [Post-deployment Verification](https://github.com/SubbuTechOps/spring-petclinic/blob/develop/documentation/deployment-verification.md) - Complete verification steps and checks after deployment

```bash
# Check deployment status
kubectl get deployments -n petclinic-dev

# Check pods
kubectl get pods -n petclinic-dev

# Check services
kubectl get svc -n petclinic-dev

# View logs
kubectl logs -f deployment/petclinic -n petclinic-dev

# Test the application
curl http://$(kubectl get svc petclinic -n petclinic-dev -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'):8080
```

---

## 11. Troubleshooting Guide

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
   
   * 🛠️ [Kubernetes Troubleshooting Guide](https://github.com/SubbuTechOps/spring-petclinic/blob/develop/documentation/k8s-troubleshooting-guide.md) - Common issues and solutions for Kubernetes deployments
  

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
kubectl get pods -n petclinic-dev -o wide

# Check logs
kubectl logs -f deployment/petclinic -n petclinic-dev

# Check service endpoints
kubectl get endpoints -n petclinic-dev
```

Remember to replace placeholder values such as `your-region`, `your-docker-hub-username`, and adjust resource limits based on your requirements.

For additional support or specific error resolution, consult the project's GitHub issues or create a new issue with detailed information about the problem encountered.
