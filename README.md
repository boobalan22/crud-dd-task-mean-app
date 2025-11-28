# CRUD-DD-TASK-MEAN-APP

This repository contains a **full-stack MEAN (MongoDB, Express, Angular, Node.js) application** deployed using Docker on AWS. This setup was implemented as part of the **DevOps Engineer Intern Assignment**.

---
Website URL - http://34.229.120.78/
---

## **Infrastructure Setup (AWS)**

**Instance Configuration:**

* **Instance Type:** `m7i-flex.large` (2 CPU, 8GB RAM)
* **AMI:** `Ubuntu 22.04 LTS`
* **Storage:** `20GB, gp3`

**Inbound Rules (Security Group):**

| Port  | Purpose              |
| ----- | -------------------- |
| 22    | SSH                  |
| 80    | HTTP                 |
| 443   | HTTPS                |
| 8080  | Backend API          |
| 8081  | Frontend Angular App |
| 9090  | Optional Monitoring  |
| 27017 | MongoDB              |

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

## **Application Configuration**

### **Backend (Node.js + Express)**

1. **Update MongoDB Connection**

Edit `backend/app/config/db.config.js`:

```javascript
// Original
module.exports = {
  url: "mongodb://localhost:27017/dd_db"
};

// Updated for Docker networking
module.exports = {
  url: "mongodb://mongodb:27017/dd_db"
};
```

2. **Enable CORS in `server.js`**

```javascript
const express = require("express");
const cors = require("cors");  // Add this

const app = express();

app.use(cors()); // Add this
```

---

### **Frontend (Angular)**

Edit `frontend/src/app/services/tutorial.service.ts`:

```typescript
// Replace <ENTER-IP> with your AWS public IP
const baseUrl = 'http://<ENTER-IP>:8080/api/tutorials';
```

---

## **Docker Setup**

### **Create Docker Network**

```bash
docker network create mern
```

---

### **MongoDB Container**

```bash
docker run -d --name mongodb --network mern -p 27017:27017 mongo:latest
```

---

### **Backend Container**

**Dockerfile (backend/Dockerfile):**

```dockerfile
FROM node:18

WORKDIR /usr/src/app

COPY package.json ./
RUN npm install

COPY . .

EXPOSE 8080

CMD ["npm", "start"]
```

**Build and Run Backend:**

```bash
docker build -t mern-backend .
docker run -itd --name backend --network mern -p 8080:8080 mern-backend:latest
```

---

### **Frontend Container**

**Dockerfile (frontend/Dockerfile):**

```dockerfile
FROM node:18

WORKDIR /app

RUN npm install -g @angular/cli

COPY package*.json ./
RUN npm install

COPY . .

RUN ng build --configuration production
RUN npm install -g http-server

EXPOSE 8081

CMD ["http-server", "dist/angular-15-crud", "-p", "8081"]
```

**Build and Run Frontend:**

```bash
docker build -t mern-frontend .
docker run -itd --name frontend -p 8081:8081 mern-frontend:latest
```

> Open your browser and navigate to `http://<AWS-IP>:8081` to verify the frontend is working.

---

## **Docker Compose Setup**

**docker-compose.yml:**

```yaml
version: '3'

services:
  mongodb:
    image: mongo:latest
    container_name: mongodb
    ports:
      - "27017:27017"
    networks:
      - mern

  backend:
    build: ./backend
    container_name: backend
    ports:
      - "8080:8080"
    networks:
      - mern
    depends_on:
      - mongodb

  frontend:
    build: ./frontend
    container_name: frontend
    ports:
      - "8081:8081"
    networks:
      - mern
    depends_on:
      - backend

networks:
  mern:
    driver: bridge
```

**Commands:**

```bash
docker compose up -d --build
docker compose down
```

---

## **Check Application**

1. MongoDB running: `docker ps | grep mongodb`
2. Backend running: `docker ps | grep backend`
3. Frontend running: `docker ps | grep frontend`
4. Access Angular fronten

# Jenkins CI/CD Pipeline for MEAN App Deployment

This README provides step-by-step instructions to set up **Jenkins CI/CD pipeline** for deploying the **CRUD-DD-TASK-MEAN-APP** using Docker on an AWS VM.

---

## **Step 1: Install Jenkins & Plugins**

**Install Jenkins on Ubuntu VM:**

```bash
sudo apt update
sudo apt install openjdk-21-jdk -y

wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

sudo apt update
sudo apt install jenkins -y

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
2. Pipeline → Definition → Pipeline script

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

    environment {
        DOCKERHUB = credentials('dockerhub-creds')
        DOCKERHUB_USER = "${DOCKERHUB_USR}"
        DOCKERHUB_PASS = "${DOCKERHUB_PSW}"

        FRONTEND_IMG = "${DOCKERHUB_USR}/frontend"
        BACKEND_IMG  = "${DOCKERHUB_USR}/backend"
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/boobalan22/crud-dd-task-mean-app.git'
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                docker build -t $FRONTEND_IMG:latest ./frontend
                docker build -t $BACKEND_IMG:latest ./backend
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh '''
                echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                docker push $FRONTEND_IMG:latest
                docker push $BACKEND_IMG:latest
                '''
            }
        }

        stage('Deploy on AWS VM') {
            steps {
                sh '''
                docker network create app-network || true
                docker rm -f frontend backend mongo || true

                docker run -d --name mongo --network app-network -p 27017:27017 mongo:latest

                docker run -d --name backend --network app-network -p 8080:8080 \
                    -e MONGO_URL=mongodb://mongo:27017/testdb \
                    $BACKEND_IMG:latest

                docker run -d --name frontend --network app-network -p 8081:8081 \
                    $FRONTEND_IMG:latest
                '''
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
4. **Deploy Containers** → Stops old containers, runs new ones with Docker.
5. **Logout** → Logs out from Docker Hub.

---

This pipeline provides a **fully automated CI/CD process** for the MEAN stack application with **Docker and Jenkins**

<img width="1919" height="970" alt="Screenshot 2025-11-28 163143" src="https://github.com/user-attachments/assets/67054f46-42dd-475f-954c-cd4b8e23ecd2" />

# Nginx Reverse Proxy for MEAN App

This guide sets up **Nginx as a reverse proxy** for the **CRUD-DD-TASK-MEAN-APP**. The entire application will be accessible via **port 80**. HTTPS and domain mapping are not required.

---

## **Step 1: Create Nginx Configuration**

Create a folder and the configuration file:

```bash
mkdir -p /opt/nginx
nano /opt/nginx/default.conf
```

Paste the following content into `default.conf`:

```nginx
server {
    listen 80;

    # Frontend proxy
    location / {
        proxy_pass http://frontend:8081/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API proxy
    location /api/ {
        proxy_pass http://backend:8080/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

This configuration ensures:

* `/` routes to the Angular frontend.
* `/api/` routes to the Node.js backend.

---

## **Step 2: Run the Nginx Container**

Run Nginx attached to your existing Docker network:

```bash
docker run -d \
  --name nginx \
  --network app-network \
  -p 80:80 \
  -v /opt/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro \
  nginx:latest
```

**Explanation of flags:**

| Flag                    | Purpose                                                                     |
| ----------------------- | --------------------------------------------------------------------------- |
| `--name nginx`          | Assigns a name to the container for easy reference                          |
| `--network app-network` | Connects Nginx to the Docker network to communicate with frontend & backend |
| `-p 80:80`              | Exposes Nginx on port 80 of the EC2 instance                                |
| `-v ...`                | Mounts the local configuration file into the container (read-only)          |

After running this, your MEAN app will be accessible via `http://<EC2-IP>`.

---

### **Optional Verification**

Check running containers:

```bash
docker ps
```

Logs for Nginx:

```bash
docker logs -f nginx
```

Open your browser and go to `http://<EC2-IP>` to verify that:

* The Angular frontend loads.
* API calls to `/api/` route correctly to the backend.

##Output-1:

<img width="1919" height="968" alt="Screenshot 2025-11-28 170623" src="https://github.com/user-attachments/assets/72b68920-bd9c-478b-8a4a-7d5d53c0a30a" />

##Output-2:

<img width="1919" height="968" alt="Screenshot 2025-11-28 170723" src="https://github.com/user-attachments/assets/daf9f98b-e388-486f-9687-c099e90160b8" />

##Output-3:

<img width="1916" height="968" alt="Screenshot 2025-11-28 170744" src="https://github.com/user-attachments/assets/eb53de5d-ce46-452d-bc1f-7ec713848ee6" />

##Output-4:

<img width="1919" height="974" alt="Output-4" src="https://github.com/user-attachments/assets/6cd66d2a-906b-4271-b838-c565a446be4b" />



