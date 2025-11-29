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
