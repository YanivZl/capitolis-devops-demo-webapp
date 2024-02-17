pipeline {
    agent any

    environment {
        // Customize these environment variables as per your setup
        DOCKER_HUB_CREDENTIALS = credentials('dokcer-hub')
        DOCKER_IMAGE_NAME = 'capitolis-devops-demo-webapp'
        DOCKERFILE_PATH = './Dockerfile'
    }

    stages {
        stage('Build') {
            steps {
                // Install Node.js and npm
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build Docker image
                script {
                    docker.build(DOCKER_IMAGE_NAME, "-f ${DOCKERFILE_PATH} .")
                }
            }
        }

        stage('Push to Registry') {
            steps {
                // Push Docker image to registry
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS)]) {
                        docker.withRegistry('https://index.docker.io/v1/') {
                            docker.image("${DOCKER_IMAGE_NAME}:${env.BUILD_ID}").push('latest')
                        }
                    }
                }
            }
        }
    }
}
