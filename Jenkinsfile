pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'maheshgowdamg25'
        DOCKER_HUB_REPO = 'docker'
        IMAGE_NAME = 'nginx'
        CONTAINER_NAME = 'app'
        DOCKER_HUB_PASS = 'Mahi@2001'  
    }

    stages {
        stage('Install Docker') {
            steps {
                script {
                    def dockerInstalled = sh(script: 'which docker', returnStatus: true) == 0
                    if (!dockerInstalled) {
                        sh '''
                        sudo yum update -y
                        sudo yum install -y docker
                        sudo systemctl start docker
                        sudo systemctl enable docker
                        sudo usermod -aG docker jenkins
                        '''
                    }
                }
            }
        }

        stage('Pull Nginx Image') {
            steps {
                sh 'docker pull nginx'
            }
        }

        stage('Remove Existing Container (if any)') {
            steps { 
                script {
                    def containerExists = sh(script: "docker ps -a --filter 'name=${CONTAINER_NAME}' --format '{{.ID}}'", returnStdout: true).trim()
                    if (containerExists) {
                        sh "docker rm -f ${CONTAINER_NAME}"
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d --name ${CONTAINER_NAME} -p 8000:80 nginx'
            }
        }

        stage('Create Snapshot of Running Container') {
            steps {
                sh 'docker commit ${CONTAINER_NAME} ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO}:${IMAGE_NAME}'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh 'echo "$DOCKER_HUB_PASS" | docker login -u "$DOCKER_HUB_USER" --password-stdin'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh 'docker push ${DOCKER_HUB_USER}/${DOCKER_HUB_REPO}:${IMAGE_NAME}'
            }
        }
    }

    post {
        always {
            script {
                // Delete the image after the pipeline ends
                def imageExists = sh(script: "docker images -q nginx", returnStdout: true).trim()
                if (imageExists) {
                    sh "docker rmi -f nginx"
                }
            }
        }

        success {
            script {
                // Remove the container after a successful pipeline
                def containerExists = sh(script: "docker ps -q -f name=${CONTAINER_NAME}", returnStdout: true).trim()
                if (containerExists) {
                    sh "docker stop ${CONTAINER_NAME}"
                    sh "docker rm ${CONTAINER_NAME}"
                }
            }
        }
    }
}