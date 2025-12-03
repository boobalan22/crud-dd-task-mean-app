# CRUD-DD-TASK-MEAN-APP

This repository contains a **full-stack MEAN (MongoDB, Express, Angular, Node.js) application** deployed using Docker on AWS. This setup was implemented as part of the **DevOps Engineer Intern Assignment**.

# Jenkins CI/CD Pipeline for MEAN App Deployment

<img width="1919" height="1019" alt="image" src="https://github.com/user-attachments/assets/590a840a-11b6-4546-b524-20ebe4c16f82" />

# Running Images and Containers

<img width="1732" height="291" alt="image" src="https://github.com/user-attachments/assets/46d4b809-e5a7-4ca9-9a22-3588d6037029" />

# DockerHub

<img width="1568" height="428" alt="image" src="https://github.com/user-attachments/assets/460b9a74-d31b-4539-af36-4bff916927ae" />

## **Infrastructure Setup (AWS)**

**Instance Configuration:**

* **Instance Type:** `c7i-flex.large` (2 CPU, 4GB RAM)
* **AMI:** `Ubuntu 22.04 LTS`
* **Storage:** `15GB, gp3`

**Inbound Rules (Security Group):**

| Port  | Purpose              |
| ----- | -------------------- |
| 22    | SSH                  |
| 80    | HTTP                 |
| 443   | HTTPS                |
| 8080  | Backend API          |
| 9090  | Jenkins              |
| 27017 | MongoDB              |

<img width="1919" height="977" alt="image" src="https://github.com/user-attachments/assets/473d1464-604e-4e5f-ae16-46c93f81f7de" />

---

## **Repository**

**GitHub URL:** [https://github.com/boobalan22/crud-dd-task-mean-app.git](https://github.com/boobalan22/crud-dd-task-mean-app.git)

---

## **Docker Installation**

```bash
sudo apt update && sudo apt install net-tools -y
curl https://get.docker.com/ | bash
docker --version
# Example Output: Docker version 29.1.0, build 360952c

sudo apt install docker-compose -y
docker-compose --version
# Example Output: docker-compose version 1.29.2
```

---

## **Docker Setup**

**Dockerfile (backend/Dockerfile):**

```dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8080

CMD ["node", "server.js"]
```

---

### **Frontend Container**

**Dockerfile (frontend/Dockerfile):**

```dockerfile
FROM node:18 as builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build --prod

FROM nginx:alpine

COPY --from=builder /app/dist/angular-15-crud /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
```

---

## **Docker Compose Setup**

**docker-compose.yml:**

```yaml
version: '3.8'

services:
  mongo:
    image: mongo
    container_name: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  backend:
    image: boobu/backend-mean
    container_name: backend
    ports:
      - "8080:8080"
    depends_on:
      - mongo
    environment:
      - MONGO_URL=mongodb://mongo:27017/dd_db

  frontend:
    image: boobu/frontend-mean
    container_name: frontend
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  mongo-data:
```

---
# Nginx setup
## nginx.config

```config
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://backend:8080/api/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

# Jenkins CI/CD Pipeline for MEAN App Deployment

This README provides step-by-step instructions to set up **Jenkins CI/CD pipeline** for deploying the **CRUD-DD-TASK-MEAN-APP** using Docker on an AWS VM.

---

## **Step 1: Install Jenkins & Plugins**

**Install Jenkins on Ubuntu VM:**

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install jenkins
# Start and enable Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
# Change Jenkins Port to 9090 on Ubuntu

This guide explains how to change the default Jenkins port (8080) to **9090** on an Ubuntu VM.

---

## **Step 1: Edit Jenkins Default Configuration**

Open the Jenkins default configuration file:

```bash
sudo vi /etc/default/jenkins
```

Find the line:

```bash
HTTP_PORT=8080
```

Change it to:

```bash
HTTP_PORT=9090
```

Save and exit the file.

---

## **Step 2: Edit Systemd Service File (Optional)**

If needed, open the systemd service file:

```bash
sudo vi /lib/systemd/system/jenkins.service
```

Check for any hard-coded port values and ensure it points to the `$HTTP_PORT` variable from `/etc/default/jenkins`.

Save and exit.

---

## **Step 3: Reload Systemd Daemon**

```bash
sudo systemctl daemon-reload
```

---

## **Step 4: Restart Jenkins**

```bash
sudo systemctl restart jenkins
```

---

## **Step 5: Enable Jenkins on Boot**

```bash
sudo systemctl enable jenkins
```

---

## **Step 6: Verify Jenkins Port**

1. Check Jenkins status:

```bash
sudo systemctl status jenkins
```

2. Open your browser and navigate to:

```
http://<VM_IP>:9090
```

Jenkins should now be accessible on port **9090**.

---

**Access Jenkins:**

Open in browser: `http://<VM_IP>:9090` I change the port number, Because backend is running on 8080

**Install Plugins:**

Navigate to: `Manage Jenkins → Manage Plugins → Available`

Install:

* Docker Pipeline
* Git
* Pipeline

**Allow Jenkins to access Docker:**

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo systemctl restart docker
```

---

## **Step 2: Add Docker Hub Credentials**

Navigate to: `Manage Jenkins → Credentials → System → Global credentials → Add Credentials`

* **Kind:** Username with password
* **Username:** Docker Hub username
* **Password:** Docker Hub password
* **ID:** `dockerhub-creds`  *(used in Jenkinsfile)*

---

## **Step 3: Create a Jenkins Pipeline Job**

1. Click **New Item → Pipeline → OK**
2. Pipeline → Definition → Pipeline script from SCM

<img width="1919" height="1019" alt="image" src="https://github.com/user-attachments/assets/cd726b81-6d2f-4929-9d50-40e132a2b50f" />

---

## **Step 4: Jenkinsfile**

This pipeline automates the following steps:

* Clone GitHub repo
* Build Docker images
* Push Docker images to Docker Hub
* Deploy containers on AWS VM

```groovy
pipeline {
    agent any
    stages {
        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh """
                        docker rmi -f boobu/backend-mean || true
                        docker build -t boobu/backend-mean .
                    """
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh """
                        docker rmi -f boobu/frontend-mean || true
                        docker build -t boobu/frontend-mean .
                    """
                }
            }
        }

        stage("Push the Images to Docker Hub") { 
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push $DOCKER_USER/frontend-mean
                        docker push $DOCKER_USER/backend-mean

                        docker logout
                    """
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                sh '''
                    docker-compose down || true
                    docker-compose up -d --build
                '''
            }
        }

        stage('Clean Dangling Images') {
            steps {
                sh 'docker image prune -f'
            }
        }
    }
}
```
---

## **Step 5: How It Works **

1. **Checkout Code** → Pulls the latest code from GitHub.
2. **Build Docker Images** → Builds backend & frontend images.
3. **Push Docker Images** → Pushes images to Docker Hub.
4. **Run Docker Compose** → Runs the Docker Container.

---

This pipeline provides a **fully automated CI/CD process** for the MEAN stack application with **Docker and Jenkins**

<img width="1919" height="1019" alt="image" src="https://github.com/user-attachments/assets/590a840a-11b6-4546-b524-20ebe4c16f82" />



### **Optional Verification**

Check running containers:

```bash
docker ps
```


Open your browser and go to `http://<EC2-IP>` to verify that:

* The Angular frontend loads.
* API calls to `/api/` route correctly to the backend.

##Output:

<img width="1919" height="1018" alt="Screenshot 2025-12-01 211335" src="https://github.com/user-attachments/assets/79c3278b-e04a-4bfe-9612-db5eb91129b0" />


<img width="1919" height="1019" alt="Screenshot 2025-12-01 211343" src="https://github.com/user-attachments/assets/92e7665d-69d9-407e-affe-60d8978409f1" />


<img width="1919" height="1019" alt="Screenshot 2025-12-01 211400" src="https://github.com/user-attachments/assets/47dedb17-24a7-466c-8a76-5e6c860a8922" />


<img width="1918" height="1018" alt="Screenshot 2025-12-01 211414" src="https://github.com/user-attachments/assets/38d9759e-e9cd-4182-a38c-1eaae68d59d5" />


<img width="1919" height="1020" alt="Screenshot 2025-12-01 211456" src="https://github.com/user-attachments/assets/8cd7ebdc-29d2-49e4-9d83-847b290c242d" />


<img width="1919" height="968" alt="image" src="https://github.com/user-attachments/assets/6ad0c13b-becf-4f8f-85cb-d46246d817ad" />




