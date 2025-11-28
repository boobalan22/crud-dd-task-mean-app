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
