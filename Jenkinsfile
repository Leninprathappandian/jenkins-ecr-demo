
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        AWS_REGION = 'ap-southeast-2'
        AWS_ACCOUNT_ID = '428819150292'
        ECR_REPOSITORY = 'jenkins-ecr-demo'
        IMAGE_NAME = 'jenkins-ecr-demo'
        ECR_URL = '428819150292.dkr.ecr.ap-southeast-2.amazonaws.com'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Leninprathappandian/jenkins-ecr-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \
                    ${ECR_URL}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \
                    ${ECR_URL}/${ECR_REPOSITORY}:latest
                '''
            }
        }

        stage('Authenticate with ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ecr-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        #!/bin/bash
                        set -euo pipefail
                        set +x

                        export AWS_DEFAULT_REGION="${AWS_REGION}"

                        echo "Checking AWS credentials..."
                        aws sts get-caller-identity

                        echo "Authenticating with Amazon ECR..."

                        aws ecr get-login-password \
                            --region "${AWS_REGION}" |
                        docker login \
                            --username AWS \
                            --password-stdin "${ECR_URL}"
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push ${ECR_URL}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                    docker push ${ECR_URL}/${ECR_REPOSITORY}:latest
                '''
            }
        }

        stage('Pull Image from ECR') {
            steps {
                sh '''
                    docker pull \
                    ${ECR_URL}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                    docker images
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f jenkins-ecr-website || true

                    docker run -d \
                        --name jenkins-ecr-website \
                        --restart unless-stopped \
                        -p 8081:80 \
                        ${ECR_URL}/${ECR_REPOSITORY}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    for i in 1 2 3 4 5; do

                        if curl -fsS http://localhost:8081/; then
                            echo "Website deployment successful!"
                            exit 0
                        fi

                        sleep 3
                    done

                    echo "Website deployment failed!"
                    docker logs jenkins-ecr-website
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Image built, pushed, pulled and deployed!'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage console output.'
        }
    }
}
