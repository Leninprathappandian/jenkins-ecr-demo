
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-ecr-demo'
        CONTAINER_NAME = 'jenkins-ecr-demo-web'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code checked out from GitHub'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker rm -f jenkins-ecr-test || true

                    docker run -d \
                        --name jenkins-ecr-test \
                        -p 8081:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    sleep 3

                    curl --fail http://localhost:8081/

                    docker rm -f jenkins-ecr-test
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p 80:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'curl --fail http://localhost/'
                echo 'Website deployment successful!'
            }
        }
    }
}
