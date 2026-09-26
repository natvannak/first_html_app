pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'echo "Building the project..."'
                sh 'docker build -t krolnoeurnrpisb/usea-app-html:${BUILD_NUMBER} .'
               
            }
        }
        stage('Push Image') {
            steps {
                sh 'echo "Push image to registry..."'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
                sh 'docker push krolnoeurnrpisb/usea-app-html:${BUILD_NUMBER}'
                // Add your test commands here
            }
        }
        stage('Deploy') {
            steps {
                script{
                    sh 'echo "Deploying the project..."'
                // ssh agent(['your-ssh-credentials-id']) {
                //     sh 'ssh user@your-server "docker pull krolnoeurnrpisb/usea-app-html:1.0.${BUILD_NUMBER} && docker stop your-container-name || true && docker rm your-container-name || true && docker run -d --name your-container-name -p 80:80 krolnoeurnrpisb/usea-app-html:1.0.${BUILD_NUMBER}"'
                // }
                    ssh '''
                        // remove container if it exists
                        ssh root@54.204.234.40 docker stop usea-app-html || true
                    '''
                    sh 'ssh root@54.204.234.40 docker run -d --name usea-app-html -p 9099:80 krolnoeurnrpisb/usea-app-html:${BUILD_NUMBER}'
                // Add your deploy commands here
                }
                
            }
        }
    }
}