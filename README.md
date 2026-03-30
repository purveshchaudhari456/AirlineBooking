# ✈️ Galactic Airlines – Containerized Flight Management on AWS EKS

> A full-stack flight management web application with a **Frontend (Nginx)**, **Backend (Flask/Python)**, and **Database (MySQL)** — each containerized with Docker, deployed on **Amazon EKS**, and automated via a **Jenkins CI/CD pipeline**.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Infrastructure Setup](#infrastructure-setup)
- [Docker Images](#docker-images)
- [Jenkins CI/CD Pipeline](#jenkins-cicd-pipeline)
- [Kubernetes Deployment](#kubernetes-deployment)
- [API Reference](#api-reference)
- [Screenshots](#screenshots)

---

## 📌 Project Overview

**Galactic Airlines Employee Portal** is a 3-tier containerized web application that allows airline employees to:

- ➕ **Add** new flight records
- 🗑️ **Delete** existing flights by identifier
- 📋 **Fetch & View** all scheduled flights

The entire application is containerized using **Docker**, orchestrated on **Amazon EKS (Kubernetes)**, and deployed automatically through a **Jenkins CI/CD pipeline** running on an EC2 instance.

---

## 🏗️ Architecture

```
  Developer
     │
     │  git push
     ▼
┌─────────────────────────────────────────────────────────┐
│               GitHub Repository                          │
│   Frontend | Backend | Database | K8s Manifests         │
└──────────────────────┬──────────────────────────────────┘
                       │  Webhook / Poll
                       ▼
┌─────────────────────────────────────────────────────────┐
│           Jenkins CI/CD (EC2 Instance)                   │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Pipeline Stages:                                 │   │
│  │  1. Git Clone → 2. Docker Build → 3. Docker Push │   │
│  │  4. kubectl apply → Deploy to EKS                │   │
│  └──────────────────────────────────────────────────┘   │
│  Tools: Git | Docker | kubectl                          │
└──────────────────────┬──────────────────────────────────┘
                       │  kubectl apply
                       ▼
┌─────────────────────────────────────────────────────────┐
│              Amazon EKS Cluster                          │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              EC2 Worker Nodes                    │   │
│  │                                                  │   │
│  │  ┌────────────┐ ┌────────────┐ ┌─────────────┐  │   │
│  │  │  Frontend  │ │  Backend   │ │  Database   │  │   │
│  │  │  Pod       │ │  Pod       │ │  Pod        │  │   │
│  │  │  (Nginx)   │ │  (Flask)   │ │  (MySQL)    │  │   │
│  │  │  Port: 80  │ │  Port:5000 │ │  Port:3306  │  │   │
│  │  └─────┬──────┘ └─────┬──────┘ └──────┬──────┘  │   │
│  │        │              │               │          │   │
│  │  ┌─────▼──────────────▼───────────────▼──────┐  │   │
│  │  │           Kubernetes Services              │  │   │
│  │  │  frontend-service (LoadBalancer/NodePort)  │  │   │
│  │  │  backend-service  (ClusterIP :5000)        │  │   │
│  │  │  database-service (ClusterIP :3306)        │  │   │
│  │  └────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                       │
                  ┌────▼────┐
                  │  Users  │
                  │(Browser)│
                  └─────────┘
```

---

## 🧰 Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | HTML5, CSS3, JavaScript, Nginx |
| **Backend** | Python 3.9, Flask, Flask-CORS |
| **Database** | MySQL (`galactic_airlines` DB) |
| **Containerization** | Docker |
| **Orchestration** | Amazon EKS (Kubernetes) |
| **CI/CD** | Jenkins (on EC2) |
| **Version Control** | Git + GitHub |
| **Cloud** | AWS (EKS, EC2) |

---

## 📁 Project Structure

```
galactic-airlines/
├── frontend/
│   ├── index.html                # Employee portal home
│   ├── add_flight.html           # Add new flight form
│   ├── delete_flight.html        # Delete flight by ID
│   ├── fetch_flights.html        # View all flights table
│   ├── script.js                 # Frontend API calls
│   ├── style.css                 # Styles
│   └── Dockerfile                # Frontend container
│
├── backend/
│   ├── app.py                    # Flask API (add/delete/fetch flights)
│   ├── requirements.txt          # Python dependencies
│   ├── default.conf              # Nginx reverse proxy config
│   └── Dockerfile                # Backend container (Python 3.9)
│
├── database/
│   ├── init.sql                  # DB + table creation script
│   └── Dockerfile                # MySQL container
│
├── k8s/
│   ├── frontend-deployment.yaml  # Frontend K8s deployment + service
│   ├── backend-deployment.yaml   # Backend K8s deployment + service
│   └── database-deployment.yaml  # MySQL K8s deployment + service
│
├── Jenkinsfile                   # CI/CD pipeline definition
└── README.md
```

---

## 🚀 Infrastructure Setup

### Step 1: Launch EC2 Instance (Jenkins Server)

1. Launch an **EC2 instance** (Ubuntu, t2.medium recommended)
2. Open ports: `22` (SSH), `8080` (Jenkins), `80` (HTTP)

### Step 2: Install Jenkins

```bash
# Install Java
sudo apt update
sudo apt install fontconfig openjdk-17-jre -y

# Add Jenkins repo and install
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Access Jenkins at: `http://<EC2-PUBLIC-IP>:8080`

### Step 3: Install Git

```bash
sudo apt install git -y
git --version
```

### Step 4: Install Docker

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# Give Jenkins permission to use Docker
sudo usermod -aG docker jenkins
sudo chmod 666 /var/run/docker.sock
```

### Step 5: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
  https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

### Step 6: Create EKS Cluster

```bash
# Install eksctl
curl --silent --location \
  "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
  | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create EKS cluster
eksctl create cluster \
  --name galactic-airlines \
  --region ap-south-1 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2
```

### Step 7: Grant Jenkins Permissions

```bash
# Add jenkins user to docker group (if not done already)
sudo usermod -aG docker jenkins

# Copy kubeconfig to Jenkins home
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp ~/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

# Restart Jenkins to apply group changes
sudo systemctl restart jenkins
```

---

## 🐳 Docker Images

### Backend Dockerfile
```dockerfile
FROM python:3.9
WORKDIR /app
COPY default.conf /etc/nginx/conf.d/
COPY . /app
RUN pip install -r requirements.txt
EXPOSE 5000
EXPOSE 80
ENV FLASK_ENV=development
ENV FLASK_DEBUG=1
CMD ["flask", "run", "--host=0.0.0.0", "--port=5000", "--debug"]
```

### Nginx Config (default.conf)
```nginx
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Database Init (init.sql)
```sql
CREATE DATABASE IF NOT EXISTS galactic_airlines;
USE galactic_airlines;

CREATE TABLE IF NOT EXISTS flights (
    id INT AUTO_INCREMENT PRIMARY KEY,
    flight_identifier VARCHAR(50),
    departure_location VARCHAR(100),
    destination_location VARCHAR(100),
    scheduled_departure DATETIME,
    scheduled_arrival DATETIME,
    aircraft_model VARCHAR(50),
    seating_capacity INT,
    crew_count INT
);
```

---

## 🔄 Jenkins CI/CD Pipeline

### Jenkinsfile
```groovy
pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = "your-dockerhub-username"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/your-username/galactic-airlines.git'
            }
        }
        stage('Build Docker Images') {
            steps {
                sh 'docker build -t ${DOCKER_HUB_REPO}/frontend:${IMAGE_TAG} ./frontend'
                sh 'docker build -t ${DOCKER_HUB_REPO}/backend:${IMAGE_TAG} ./backend'
                sh 'docker build -t ${DOCKER_HUB_REPO}/database:${IMAGE_TAG} ./database'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push ${DOCKER_HUB_REPO}/frontend:${IMAGE_TAG}'
                    sh 'docker push ${DOCKER_HUB_REPO}/backend:${IMAGE_TAG}'
                    sh 'docker push ${DOCKER_HUB_REPO}/database:${IMAGE_TAG}'
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}
```

---

## ☸️ Kubernetes Deployment

### Services Overview

| Service | Type | Port |
|---------|------|------|
| `frontend-service` | LoadBalancer / NodePort | 80 |
| `backend-service` | ClusterIP | 5000 |
| `database-service` | ClusterIP | 3306 |

The backend connects to MySQL using the Kubernetes DNS name `database-service` and the frontend calls the backend via `backend-service:5000`.

---

## 🔌 API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Health check |
| `POST` | `/add_flight` | Add a new flight |
| `GET` | `/fetch_flights` | Get all flights |
| `DELETE` | `/delete_flight/<flight_id>` | Delete flight by identifier |

**Example POST `/add_flight` body:**
```json
{
  "flight_identifier": "GA101",
  "departure_location": "Mumbai",
  "destination_location": "Delhi",
  "scheduled_departure": "2026-04-01T08:00",
  "scheduled_arrival": "2026-04-01T10:00",
  "aircraft_model": "Boeing 737",
  "seating_capacity": 180,
  "crew_count": 6
}
```

---

## 📸 Screenshots

### 🌐 Web Application

**Home Page – Galactic Airlines Employee Portal**

![Home Page](screenshots/ss_home.png)

**Add Flight Form**

![Add Flight](screenshots/ss_add_flight.png)

**Fetch Flights – Live Data**

![Fetch Flights](screenshots/ss_fetch_flights.png)

**Delete Flight**

![Delete Flight](screenshots/ss_delete_flight.png)

---

### ☁️ AWS & DevOps Infrastructure

**EKS Cluster**

![EKS Cluster](screenshots/ss_eks_cluster.png)

**EC2 Instances (Jenkins + Worker Nodes)**

![EC2 Instances](screenshots/ss_ec2.png)

**Jenkins Dashboard**

![Jenkins](screenshots/ss_jenkins.png)

**Jenkins Pipeline – Build Stages**

![Jenkins Pipeline](screenshots/ss_jenkins_pipeline.png)

**Docker Hub – Images Pushed**

![Docker Hub](screenshots/ss_dockerhub.png)

**Kubernetes Pods Running**

![K8s Pods](screenshots/ss_k8s_pods.png)

**Kubernetes Services**

![K8s Services](screenshots/ss_k8s_services.png)

---

## 📜 License

This project is for educational/training purposes.
