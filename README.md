# The Ultimate CI/CD/K8s Pipeline on Kali Linux - Complete Setup Guide

A comprehensive, battle-tested guide documenting the complete journey of building a production-grade Jenkins pipeline with Maven, Docker, SonarQube, and Kubernetes integration on Kali Linux. This guide includes all the troubleshooting fixes discovered through 50+ build attempts.

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Prerequisites](#prerequisites)
3. [Phase 1: Core Infrastructure Setup](#phase-1-core-infrastructure-setup)
4. [Phase 2: Jenkins Configuration](#phase-2-jenkins-configuration)
5. [Phase 3: Docker Integration](#phase-3-docker-integration)
6. [Phase 4: Kubernetes (Minikube) Setup](#phase-4-kubernetes-minikube-setup)
7. [Phase 5: SonarQube Integration](#phase-5-sonarqube-integration)
8. [Phase 6: The Complete Pipeline](#phase-6-the-complete-pipeline)
9. [Common Issues & Solutions](#common-issues--solutions)
10. [Testing & Verification](#testing--verification)
11. [Project Structure](#project-structure)

---

## 🎯 Project Overview

This pipeline implements a complete DevOps workflow:

```
Code Push → Jenkins Trigger → Maven Build → SonarQube Analysis → 
Docker Image Build → Kubernetes Deployment → Service Exposure
```

**Key Features:**
- Automated CI/CD with Jenkins
- Code quality analysis with SonarQube
- Containerization with Docker
- Orchestration with Kubernetes (Minikube)
- Parameterized deployment (Deploy/Destroy)

---

## 🔧 Prerequisites

- **OS**: Kali Linux / Debian-based distribution
- **RAM**: Minimum 8GB (16GB recommended)
- **Disk**: 20GB free space
- **Network**: Internet connection for downloading packages

---

## 📦 Phase 1: Core Infrastructure Setup

### 1.1 System Update & Java Installation

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Java Development Kit 17 (Required for Jenkins & Maven)
sudo apt install openjdk-17-jdk -y

# Verify installation
java -version
javac -version
```

**Why Java 17?** Jenkins 2.x requires Java 11 or 17. Java 17 is the LTS version and provides better performance.

### 1.2 Maven Installation

```bash
# Install Maven (Build automation tool)
sudo apt install maven -y

# Verify installation
mvn -version
```

### 1.3 Jenkins Installation

```bash
# Add Jenkins repository key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io.key

# Add Jenkins repository
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update and install Jenkins
sudo apt update
sudo apt install jenkins -y

# Start and enable Jenkins service
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Check Jenkins status
sudo systemctl status jenkins
```

---

## 🔐 Phase 2: Jenkins Configuration

### 2.1 Initial Jenkins Setup

1. **Access Jenkins Web Interface:**
   ```bash
   # Open browser to: http://localhost:8080
   ```

2. **Unlock Jenkins:**
   ```bash
   # Get initial admin password
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
   Copy the password and paste it in the browser.

3. **Install Plugins:**
   - Select "Install suggested plugins"
   - Wait for installation to complete

4. **Create Admin User:**
   - Username: `admin` (or your choice)
   - Password: Create a strong password
   - Full Name: Your name
   - Email: Your email

### 2.2 Configure Jenkins Tools

Navigate to: **Manage Jenkins → Global Tool Configuration**

#### Configure JDK:
- Click "Add JDK"
- Name: `JDK-17`
- Uncheck "Install automatically"
- JAVA_HOME: `/usr/lib/jvm/java-17-openjdk-amd64`
  
  (Find path with: `sudo update-alternatives --config java`)

#### Configure Maven:
- Click "Add Maven"
- Name: `MyLocalMaven`
- Uncheck "Install automatically"
- MAVEN_HOME: `/usr/share/maven`

### 2.3 Install Required Jenkins Plugins

Navigate to: **Manage Jenkins → Manage Plugins → Available**

Install these plugins:
- **Pipeline** (should already be installed)
- **Docker Pipeline**
- **Kubernetes CLI**
- **SonarQube Scanner**
- **Git** (should already be installed)

After installation, restart Jenkins:
```bash
sudo systemctl restart jenkins
```

---

## 🐳 Phase 3: Docker Integration

### 3.1 Docker Installation

```bash
# Update package index
sudo apt update

# Install Docker
sudo apt install docker.io -y

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker --version
```

### 3.2 **CRITICAL**: Grant Jenkins Docker Permissions

This solves the infamous "permission denied" error when Jenkins tries to use Docker.

```bash
# Add jenkins user to docker group
sudo usermod -aG docker jenkins

# Verify jenkins is in docker group
groups jenkins

# Restart Jenkins to apply changes
sudo systemctl restart jenkins

# Restart Docker service
sudo systemctl restart docker
```

**Why this matters:** Jenkins runs as the `jenkins` user, which by default cannot access the Docker daemon. Adding it to the `docker` group grants the necessary permissions.

### 3.3 Test Docker Access

```bash
# Switch to jenkins user and test
sudo -u jenkins docker ps
```

If this works without errors, you're good to proceed!

---

## ☸️ Phase 4: Kubernetes (Minikube) Setup

### 4.1 Minikube Installation

```bash
# Download Minikube binary
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Install Minikube
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Verify installation
minikube version
```

### 4.2 kubectl Installation

```bash
# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client
```

### 4.3 Start Minikube Cluster

```bash
# Start Minikube (as root or with sudo)
minikube start --driver=docker --force

# Verify cluster is running
minikube status
kubectl cluster-info
```

### 4.4 **CRITICAL**: Fix Minikube Profile Isolation

This is THE solution that fixes Kubernetes integration with Jenkins. The problem: Jenkins (running as `jenkins` user) cannot access Minikube configuration created by root.

```bash
# Step 1: Copy Minikube profile from root to jenkins home
sudo cp -R /root/.minikube /var/lib/jenkins/

# Step 2: Create .kube directory if it doesn't exist
sudo mkdir -p /var/lib/jenkins/.kube

# Step 3: Copy kubeconfig file
sudo cp /root/.kube/config /var/lib/jenkins/.kube/config

# Step 4: Change ownership to jenkins user
sudo chown -R jenkins:jenkins /var/lib/jenkins/.minikube
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

# Step 5: Verify jenkins user can access kubectl
sudo -u jenkins kubectl get nodes
```

**Expected output:**
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   5m    v1.xx.x
```

### 4.5 Additional Kubernetes Setup

```bash
# Enable ingress addon (optional, for advanced routing)
minikube addons enable ingress

# Enable metrics server (optional, for monitoring)
minikube addons enable metrics-server
```

---

## 🔍 Phase 5: SonarQube Integration

### 5.1 Launch SonarQube Server

```bash
# Run SonarQube as Docker container
sudo docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  sonarqube:lts

# Wait 2-3 minutes for SonarQube to start
# Check logs: docker logs -f sonarqube
```

### 5.2 SonarQube Initial Configuration

1. **Access SonarQube:** Open browser to `http://localhost:9000`
2. **Login:** 
   - Username: `admin`
   - Password: `admin`
3. **Change Password:** You'll be prompted to change the default password
4. **Generate Token:**
   - Go to: Administration → Security → Users → Tokens
   - Generate a new token (copy and save it!)
   - Example format: `squ_xxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### 5.3 Manual SonarScanner Installation

Jenkins' automatic SonarScanner installer often fails. We install it manually.

```bash
# Download SonarScanner CLI
sudo wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip \
  -O /tmp/sonar-scanner.zip

# Extract to /opt
sudo unzip /tmp/sonar-scanner.zip -d /opt/

# Rename for easier access
sudo mv /opt/sonar-scanner-5.0.1.3006-linux /opt/sonar-scanner

# Grant jenkins user access
sudo chown -R jenkins:jenkins /opt/sonar-scanner

# Verify installation
/opt/sonar-scanner/bin/sonar-scanner --version
```

### 5.4 Configure SonarQube in Jenkins

#### Add SonarQube Token to Jenkins:
1. Navigate to: **Manage Jenkins → Manage Credentials → (global) → Add Credentials**
2. Kind: `Secret text`
3. Secret: Paste your SonarQube token
4. ID: `sonar-token`
5. Description: `SonarQube Authentication Token`
6. Click "Create"

#### Configure SonarQube Server:
1. Navigate to: **Manage Jenkins → Configure System**
2. Scroll to "SonarQube servers"
3. Check "Environment variables"
4. Click "Add SonarQube"
   - Name: `sonar-server`
   - Server URL: `http://172.17.0.1:9000` (Docker bridge IP)
   - Server authentication token: Select `sonar-token`
5. Save

**Why `172.17.0.1`?** This is Docker's default bridge network IP, allowing containers to communicate with the host.

#### Configure SonarScanner Tool:
1. Navigate to: **Manage Jenkins → Global Tool Configuration**
2. Scroll to "SonarQube Scanner"
3. Click "Add SonarQube Scanner"
   - Name: `SonarScanner`
   - Uncheck "Install automatically"
   - SONAR_RUNNER_HOME: `/opt/sonar-scanner`
4. Save

---

## 🚀 Phase 6: The Complete Pipeline

### 6.1 Project Structure

Create this structure in your Git repository:

```
your-project/
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── example/
│                   └── Application.java
├── k8s/
│   └── deployment.yaml
├── pom.xml
├── Jenkinsfile
└── README.md
```

### 6.2 Sample Spring Boot Application

**pom.xml:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>spring-boot-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**src/main/java/com/example/Application.java:**
```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class Application {
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    
    @GetMapping("/")
    public String hello() {
        return "Hello from Kubernetes! Pipeline Version 1.0";
    }
    
    @GetMapping("/health")
    public String health() {
        return "Application is healthy!";
    }
}
```

### 6.3 Kubernetes Deployment Configuration

**k8s/deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-app
  labels:
    app: spring-boot-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-boot-app
  template:
    metadata:
      labels:
        app: spring-boot-app
    spec:
      containers:
      - name: spring-boot-app
        image: spring-boot-app:latest
        imagePullPolicy: Never  # Important: Use local image
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 20
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-service
spec:
  type: NodePort
  selector:
    app: spring-boot-app
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080
```

### 6.4 The Complete Jenkinsfile

Create `Jenkinsfile` in your project root:

```groovy
pipeline {
    agent any

    parameters {
        choice(
            name: 'ACTION',
            choices: ['Deploy', 'Destroy'],
            description: 'Choose whether to Deploy or Destroy the application'
        )
    }

    tools {
        maven 'MyLocalMaven'
        jdk 'JDK-17'
    }

    environment {
        IMAGE_NAME = "spring-boot-app"
        K8S_DEPLOY_FILE = "k8s/deployment.yaml"
        SONAR_TOKEN = credentials('sonar-token')
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {
        
        stage('01. Checkout Code') {
            steps {
                echo '📥 Checking out source code from SCM...'
                checkout scm
            }
        }

        stage('02. Build with Maven') {
            when {
                expression { params.ACTION == 'Deploy' }
            }
            steps {
                echo '🔨 Building application with Maven...'
                sh 'mvn clean package -DskipTests'
                echo '✅ Build completed successfully'
            }
        }

        stage('03. SonarQube Code Analysis') {
            when {
                expression { params.ACTION == 'Deploy' }
            }
            steps {
                echo '🔍 Running SonarQube code analysis...'
                script {
                    def scannerHome = '/opt/sonar-scanner'
                    withSonarQubeEnv('sonar-server') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -Dsonar.projectKey=${IMAGE_NAME} \
                            -Dsonar.projectName=${IMAGE_NAME} \
                            -Dsonar.sources=src \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.language=java
                        """
                    }
                }
                echo '✅ Code analysis completed'
            }
        }

        stage('04. Build Docker Image') {
            when {
                expression { params.ACTION == 'Deploy' }
            }
            steps {
                echo "🐳 Building Docker image: ${IMAGE_NAME}:latest"
                script {
                    def dockerfileContent = """
FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
                    """
                    writeFile file: 'Dockerfile', text: dockerfileContent
                }
                
                sh "docker build -t ${IMAGE_NAME}:latest ."
                echo '✅ Docker image built successfully'
            }
        }

        stage('05. Load Image to Minikube') {
            when {
                expression { params.ACTION == 'Deploy' }
            }
            steps {
                echo "📦 Loading image into Minikube's Docker daemon..."
                sh """
                    export KUBECONFIG=${KUBECONFIG}
                    minikube image load ${IMAGE_NAME}:latest
                """
                echo '✅ Image loaded to Minikube'
            }
        }

        stage('06. Deploy to Kubernetes') {
            when {
                expression { params.ACTION == 'Deploy' }
            }
            steps {
                echo "☸️ Deploying application to Kubernetes..."
                sh """
                    export KUBECONFIG=${KUBECONFIG}
                    export KUBE_INSECURE_SKIP_TLS_VERIFY=true
                    
                    kubectl apply -f ${K8S_DEPLOY_FILE} --context=minikube
                    
                    echo "Waiting for deployment to be ready..."
                    kubectl rollout status deployment/spring-boot-app --context=minikube --timeout=5m
                    
                    echo "Deployment status:"
                    kubectl get deployments --context=minikube
                    kubectl get pods --context=minikube
                    kubectl get services --context=minikube
                """
                echo '✅ Application deployed successfully'
            }
        }

        stage('07. Destroy Resources') {
            when {
                expression { params.ACTION == 'Destroy' }
            }
            steps {
                echo "🗑️ Destroying Kubernetes resources..."
                sh """
                    export KUBECONFIG=${KUBECONFIG}
                    export KUBE_INSECURE_SKIP_TLS_VERIFY=true
                    
                    kubectl delete -f ${K8S_DEPLOY_FILE} --context=minikube || true
                    
                    echo "Verifying deletion..."
                    kubectl get deployments --context=minikube || true
                    kubectl get services --context=minikube || true
                """
                echo '✅ Resources destroyed successfully'
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline completed successfully!'
            script {
                if (params.ACTION == 'Deploy') {
                    echo """
                    ==========================================
                    🚀 APPLICATION DEPLOYED SUCCESSFULLY! 🚀
                    ==========================================
                    
                    Access your application:
                    → minikube service spring-boot-service --url
                    
                    Or directly at:
                    → http://\$(minikube ip):30080
                    
                    Check deployment status:
                    → kubectl get all -n default
                    ==========================================
                    """
                }
            }
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
        always {
            echo '🧹 Cleaning up workspace...'
            cleanWs()
        }
    }
}
```

### 6.5 Create Jenkins Pipeline Job

1. **Create New Job:**
   - Click "New Item"
   - Enter name: `K8s-Spring-Boot-Pipeline`
   - Select "Pipeline"
   - Click "OK"

2. **Configure Pipeline:**
   - Scroll to "Pipeline" section
   - Definition: `Pipeline script from SCM`
   - SCM: `Git`
   - Repository URL: Your Git repository URL
   - Branch: `*/main` (or your branch name)
   - Script Path: `Jenkinsfile`
   - Click "Save"

3. **Run Pipeline:**
   - Click "Build with Parameters"
   - Select ACTION: `Deploy`
   - Click "Build"

---

## 🔧 Common Issues & Solutions

### Issue 1: "Permission denied" when accessing Docker

**Symptom:** Jenkins fails with "Got permission denied while trying to connect to the Docker daemon socket"

**Solution:**
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo systemctl restart docker
```

### Issue 2: kubectl cannot connect to Minikube

**Symptom:** "The connection to the server localhost:8080 was refused"

**Solution:**
```bash
# Copy Minikube config to jenkins home
sudo cp -R /root/.minikube /var/lib/jenkins/
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp /root/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.minikube
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
```

### Issue 3: "x509: certificate signed by unknown authority"

**Symptom:** TLS certificate verification errors when using kubectl

**Solution:** Add to your pipeline:
```groovy
environment {
    KUBE_INSECURE_SKIP_TLS_VERIFY = "true"
}
```

### Issue 4: SonarQube "Invalid tool type" error

**Symptom:** Jenkins cannot find SonarScanner

**Solution:** Install SonarScanner manually and point Jenkins to `/opt/sonar-scanner`

### Issue 5: Image not found in Minikube

**Symptom:** Kubernetes shows "ImagePullBackOff" or "ErrImagePull"

**Solution:**
```bash
# Load image to Minikube
minikube image load spring-boot-app:latest

# Set imagePullPolicy to Never in deployment.yaml
imagePullPolicy: Never
```

### Issue 6: Minikube won't start

**Symptom:** "Exiting due to GUEST_MISSING_CONNTRACK"

**Solution:**
```bash
sudo apt install conntrack -y
minikube delete
minikube start --driver=docker --force
```

### Issue 7: Jenkins build hangs on kubectl commands

**Symptom:** Pipeline gets stuck indefinitely

**Solution:** Ensure KUBECONFIG is set and user has permissions:
```bash
sudo -u jenkins kubectl get nodes
```

---

## ✅ Testing & Verification

### 1. Verify Jenkins Access

```bash
# Check if Jenkins is running
sudo systemctl status jenkins

# Access web interface
curl http://localhost:8080
```

### 2. Verify Docker Integration

```bash
# Test as jenkins user
sudo -u jenkins docker ps
sudo -u jenkins docker images
```

### 3. Verify Kubernetes Cluster

```bash
# Check Minikube status
minikube status

# Test kubectl access as jenkins user
sudo -u jenkins kubectl get nodes
sudo -u jenkins kubectl get pods --all-namespaces
```

### 4. Verify SonarQube

```bash
# Check if SonarQube container is running
docker ps | grep sonarqube

# Access web interface
curl http://localhost:9000
```

### 5. End-to-End Pipeline Test

1. Push code to your Git repository
2. Run Jenkins pipeline with "Deploy" action
3. Wait for all stages to complete
4. Access your application:
   ```bash
   minikube service spring-boot-service --url
   ```
5. Test the endpoint:
   ```bash
   curl $(minikube service spring-boot-service --url)
   ```

### 6. Test Destroy Action

1. Run pipeline with "Destroy" action
2. Verify resources are deleted:
   ```bash
   kubectl get deployments
   kubectl get services
   ```

---

## 📊 Monitoring & Troubleshooting Commands

### Jenkins Logs
```bash
# View Jenkins logs
sudo journalctl -u jenkins -f

# View specific job logs
tail -f /var/lib/jenkins/jobs/K8s-Spring-Boot-Pipeline/builds/lastBuild/log
```

### Docker Commands
```bash
# List all containers
docker ps -a

# View container logs
docker logs sonarqube

# Remove unused images
docker image prune -a
```

### Kubernetes Commands
```bash
# Get all resources
kubectl get all

# Describe pod (for debugging)
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# Get events
kubectl get events --sort-by='.lastTimestamp'

# Delete stuck resources
kubectl delete pod <pod-name> --force --grace-period=0
```

### Minikube Commands
```bash
# SSH into Minikube VM
minikube ssh

# List loaded images
minikube image ls

# Get Minikube IP
minikube ip

# Access dashboard
minikube dashboard
```

---

## 🎓 What You've Learned

After completing this guide, you've successfully:

1. ✅ Set up a complete CI/CD pipeline from scratch
2. ✅ Configured Jenkins with proper permissions and tools
3. ✅ Integrated Docker for containerization
4. ✅ Deployed applications to Kubernetes
5. ✅ Implemented code quality checks with SonarQube
6. ✅ Troubleshot 50+ common DevOps issues
7. ✅ Created a production-ready deployment workflow

---

## 🚀 Next Steps & Enhancements

### Beginner Level
- [ ] Add unit tests to your application
- [ ] Configure Jenkins email notifications
- [ ] Create multiple environments (dev, staging, prod)

### Intermediate Level
- [ ] Implement Helm charts for Kubernetes deployments
- [ ] Add database integration (PostgreSQL/MySQL)
- [ ] Set up monitoring with Prometheus and Grafana
- [ ] Implement secrets management with Kubernetes Secrets

### Advanced Level
- [ ] Multi-cluster Kubernetes deployment
- [ ] Implement GitOps with ArgoCD
- [ ] Add security scanning (Trivy, OWASP Dependency Check)
- [ ] Implement blue-green or canary deployments
- [ ] Set up service mesh (Istio)

---

## 📚 Additional Resources

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [Maven Documentation](https://maven.apache.org/guides/)

---

## 🤝 Contributing & Support

This guide was created through extensive troubleshooting and testing. If you encounter any issues or have suggestions for improvements, feel free to:

- Open an issue on GitHub
- Submit a pull request
- Share your experience on LinkedIn

---

## 📝 License

This guide is provided as-is for educational purposes. Feel free to use and modify it for your own projects.

---

## 👨‍💻 Author

**Saleem Ali**  
- 🎓 Studying AIOps at Al-Nafi International College
- 💼 [LinkedIn](https://www.linkedin.com/in/saleem-ali-189719325/)
- 🐱 [GitHub](https://github.com/ali4210?tab=repositories)

---

## 🙏 Acknowledgments

Special thanks to:
- The DevOps community for extensive documentation
- Google Gemini for assistance during initial troubleshooting
- All the Stack Overflow contributors whose solutions helped solve critical issues

---

**Last Updated:** December 2024  
**Pipeline Version:** 1.0  
**Build Status:** ✅ Tested with 50+ builds

---

## 📸 Screenshots Section (Optional - Add Your Own)

You can add screenshots of:
- Jenkins dashboard
- Successful pipeline execution
- Kubernetes deployment
- Running application
- SonarQube analysis results

---

*Remember: Every error you encounter is a learning opportunity. This guide documents 50+ builds worth of troubleshooting. Use it wisely!* 🚀
