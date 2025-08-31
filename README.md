# 📺 Netflix Clone - DevSecOps Project

This project demonstrates a **DevSecOps pipeline** for deploying a Netflix Clone application with integrated CI/CD, security scans, monitoring, and Kubernetes deployment.

---

## 📖 Project Overview
The pipeline covers:
- Jenkins for CI/CD
- SonarQube for code quality
- Trivy & OWASP Dependency-Check for security scans
- Docker for containerization
- Kubernetes & ArgoCD for deployment
- Prometheus & Grafana for monitoring
- Slack for notifications

---

## ⚙️ Phase 1: Initial Setup and Deployment
### Step 1: Launch EC2 (Ubuntu 22.04)
Provision an EC2 instance and connect via SSH.

### Step 2: Clone Repo
```bash
git clone https://github.com/N4si/DevSecOps-Project.git
cd DevSecOps-Project
```

### Step 3: Install Docker
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
```

### Step 4: Run Container
```bash
docker build -t netflix .
docker run -d -p 8081:80 netflix
```

---

## 🔐 Phase 2: Security and Quality
### SonarQube Analysis
1. Install **SonarQube** on your server.  
2. Create a project and generate a token.  
3. Configure Jenkins with SonarQube plugin.  

### OWASP Dependency-Check
```groovy
dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
```

### Trivy Scan
```bash
trivy fs . > trivyfs.txt
trivy image netflix:latest > trivyimage.txt
```

---

## 🚀 Phase 3: Jenkins Pipeline
Here’s the Jenkins pipeline used in this project:

```groovy
pipeline {
    agent any

    tools {
        jdk 'jdk21'
        nodejs 'node24'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/<your-username>/<your-repo>.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('<sonar-server-name>') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=<your-project-name> \
                        -Dsonar.projectKey=<your-project-key>
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: '<dependency-check-installation-name>'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivy-fs.txt"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '<docker-credentials-id>', toolName: 'docker') {
                        sh "docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t <your-image-name> ."
                        sh "docker tag <your-image-name> <your-dockerhub-username>/<your-image-name>:latest"
                        sh "docker push <your-dockerhub-username>/<your-image-name>:latest"
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image <your-dockerhub-username>/<your-image-name>:latest > trivy-image.txt"
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                    docker rm -f <your-container-name> || true
                    docker run -d --name <your-container-name> -p 8081:80 <your-dockerhub-username>/<your-image-name>:latest
                '''
            }
        }
    }
}

```

---

## 📊 Phase 4: Monitoring and Alerts
- **Prometheus** for metrics scraping  
- **Grafana** for dashboards  
- **Slack Integration** for build and scan notifications  

---

## 🧹 Phase 5: Cleanup
When finished, remove containers, images, and stop unused services:
```bash
docker ps -a
docker stop <container_id>
docker rm <container_id>
docker rmi <image_id>
```

---

## ✅ Final Notes
- Quality gates must be configured in SonarQube.  
- Webhooks need to be added in SonarQube → Administration → Configuration → Webhooks → Jenkins URL.  
- Update DockerHub credentials in Jenkins before running the pipeline.  
