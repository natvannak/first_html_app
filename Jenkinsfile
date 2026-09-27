pipeline {
    agent any

    environment {
        IMAGE_NAME = 'natvannak/usea-app-html'
        SERVER_IP  = '44.223.99.204'
        SERVER_PORT = '9099'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Building the Docker image..."'
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Image') {
            steps {
                sh 'echo "Logging in to Docker Hub..."'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-id',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }

                sh 'echo "Pushing image to Docker Hub..."'
                sh 'docker push ${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }

        stage('Deploy') {
            steps {
                sh 'echo "Deploying application to AWS EC2..."'

                sh '''
                    ssh root@${SERVER_IP} "
                        docker pull ${IMAGE_NAME}:${BUILD_NUMBER} &&
                        docker stop usea-app-html || true &&
                        docker rm usea-app-html || true &&
                        docker run -d \
                            --name usea-app-html \
                            -p ${SERVER_PORT}:80 \
                            ${IMAGE_NAME}:${BUILD_NUMBER}
                    "
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
            echo "Application: http://${SERVER_IP}:${SERVER_PORT}"
        }

        failure {
            echo "Pipeline failed."
        }
    }
}