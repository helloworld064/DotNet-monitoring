![dotnet_with_ssl](https://github.com/user-attachments/assets/81c0ba81-b249-41de-ad8e-376fbca53e89)

![donet_with_jenkins](https://github.com/user-attachments/assets/d1f6d1dc-212a-43d9-af0c-f5a42005601d)

![dotnet_with_sonarqube](https://github.com/user-attachments/assets/bb86366c-8073-435d-940c-82b28b95945b)



# 🚀 DotNet Monitoring CI/CD with Jenkins, Docker, SonarQube, Trivy & EKS

This project demonstrates how to build a full CI/CD pipeline using:

* **EC2 (Ubuntu)**
* **Jenkins**
* **SonarQube**
* **Trivy**
* **OWASP Dependency Check**
* **Docker**
* **EKS (Elastic Kubernetes Service)**

---

## 📋 CI/CD Pipeline Steps

### ✅ Step 1 — Provision Ubuntu EC2 Instance

Create an EC2 instance with:

* **Type**: `t2.large`
* **OS**: Ubuntu 22.04
* Open ports: `8080` (Jenkins), `9000` (SonarQube), `22` (SSH)

---

### ✅ Step 2 — Install Jenkins, Docker & Trivy on EC2

#### 🔹 Jenkins Installation

```bash
sudo apt update && sudo apt install -y openjdk-17-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
echo "deb http://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword


##  Docker Installation
``` bash
sudo apt install -y ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker jenkins
sudo usermod -aG docker $USER
sudo systemctl enable docker
sudo systemctl restart docker
```
##  Trivy Installation
```bash
sudo apt install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] \
  https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/trivy.list > /dev/null

sudo apt update
sudo apt install -y trivy
```


✅ Step 3 — Run SonarQube Container

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
✅ Step 4 — Install Jenkins Plugins

# 🔧 Jenkins Plugin Installation Guide

This guide lists all essential plugins you need to install in Jenkins to set up a complete CI/CD pipeline.

## 📥 How to Install Plugins

1. Go to **Manage Jenkins** → **Plugins**.
2. Click on the **Available Plugins** tab.
3. Use the search bar to locate and install the following plugins.

## ✅ Required Plugins

| Category                | Plugins                                                                 |
|-------------------------|-------------------------------------------------------------------------|
| **Security & Analysis** | `OWASP Dependency-Check`                                                |
| **Code Quality**        | `SonarQube Scanner`                                                     |
| **Java Support**        | `Eclipse Temurin Installer`                                             |
| **Docker Support**      | `Docker`<br>`Docker Commons`<br>`Docker Pipeline`<br>`Docker API`       |
| **Kubernetes Support**  | `Kubernetes CLI Plugin`<br>`Kubernetes Credentials Plugin`              |
| **Pipeline UI**         | `Pipeline Stage View`                                                   |

## 🛠️ Post-Installation Setup

- Restart Jenkins after plugin installation.
- Go to **Manage Jenkins → Global Tool Configuration** to set up:
  - **SonarQube Scanner**
  - **Docker**
  - **Temurin JDK**

- Add credentials via **Manage Jenkins → Credentials** for:
  - **SonarQube Token**
  - **DockerHub Username/Token**
  - **Kubernetes Kubeconfig**

## 📌 Tips

- Ensure Jenkins has access to the internet to fetch and install plugins.
- Keep your plugins updated for latest features and security patches.

## ✅ Step 5 — Global Tool Configuration

Navigate to: **Manage Jenkins** → **Global Tool Configuration**

Set up the following tools:

### ☕ JDK

- **Name:** `jdk17`
- **Installer:** Adoptium
- **Version:** `jdk-17.0.8.1+1`

### 🐳 Docker

- **Name:** `docker`
- **Install Automatically:** ✅

### 🔍 SonarQube Scanner

- **Name:** `sonar-scanner`
- **Installer:** Install from Maven Central

### 🛡️ OWASP Dependency-Check

- **Name:** `DP-Check`
- **Installer:** Install automatically from GitHub

## ✅ Step 6 — Configure SonarQube in Jenkins

### 🔗 Access SonarQube
- Open SonarQube in your browser:  
  `http://<ec2-ip>:9000`

---

### 🔐 Generate Authentication Token
- Navigate to:  
  `Administration → Security → Tokens`
- Click **Generate Token** and copy the token.

---

### 🔑 Add Token to Jenkins
1. Go to: **Manage Jenkins → Credentials → Global → Add Credentials**
2. Set the following:
   - **Kind:** Secret text  
   - **ID:** `Sonar-token`  
   - **Secret:** _Paste the token you generated from SonarQube_

---

### ⚙️ Configure SonarQube Server in Jenkins
1. Go to: **Manage Jenkins → Configure System**
2. Under **SonarQube Servers**, click **Add SonarQube** and fill in:
   - **Name:** `sonar-server`  
   - **Server URL:** `http://<sonarqube-ip>:9000`  
   - **Server Authentication Token:** Select `Sonar-token` from dropdown

---

### 🔁 Add Webhook in SonarQube
- Go to: `Administration → Configuration → Webhooks`
- Add a new webhook:
  - **Name:** `Jenkins`
  - **URL:** `http://<jenkins-ip>:8080/sonarqube-webhook/`

---

> 💡 Replace `<ec2-ip>`, `<sonarqube-ip>`, and `<jenkins-ip>` with actual IP addresses.


## ✅ Step 7 — Install Make
```bash
sudo apt install make
make -v
```

## ✅ Step 8 — Install AWS CLI & kubectl
🔹 AWS CLI
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure
```
🔹 kubectl
```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

## ✅ Step 9 — Create EKS Cluster
```bash
eksctl create cluster --name dotnet-clone --region eu-north-1 --node-type t3.medium --nodes 2
```
## 📁 Upload Kubernetes `.kube/config` to Jenkins

To allow Jenkins to interact with your Kubernetes cluster, upload your kubeconfig file:

### 🔼 Step-by-Step:

1. **Copy `.kube/config`** from your EC2 instance:
   ```bash
   scp -i <your-key.pem> ec2-user@<ec2-ip>:~/.kube/config .

## 📜 Jenkins Pipeline (Declarative)

```
pipeline {
    agent any

    tools {
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_REGION = 'eu-north-1'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/helloworld064/DotNet-monitoring.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=Dotnet-Webapp \
                        -Dsonar.projectKey=Dotnet-Webapp
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh 'trivy fs . > trivy-fs_report.txt'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'make image'
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image sevenajay/dotnet-monitoring:latest > trivy.txt'
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'make push'
                    }
                }
            }
        }

        stage('Deploy to Docker Container') {
            steps {
                sh '''
                    docker rm -f dotnet || true
                    docker run -d --name dotnet -p 5000:5000 sevenajay/dotnet-monitoring:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'aws-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )]) {
                        withEnv(["AWS_REGION=${env.AWS_REGION}"]) {
                            dir('K8S') {
                                withKubeConfig(credentialsId: 'k8s') {
                                    sh '''
                                        kubectl delete --all pods || true
                                        kubectl apply -f deployment.yml
                                    '''
                                }
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline execution failed.'
        }
    }
}
```


## 📁 Directory Structure
.
├── K8S/
│   └── deployment.yml
├── Makefile
├── README.md



# 🧹 Cleanup
Delete EKS Cluster

```bash
eksctl delete cluster --name dotnet-clone --region=eu-north-1
```



