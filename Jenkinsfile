pipeline {
    agent any
    
    environment {
        IMAGE_NAME = 'my-app' // Nombre de la imagen
        IMAGE_TAG = 'latest' // Etiqueta de la imagen
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gastonbarbaccia/jenkins_docker.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Construir la imagen Docker
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Iniciar sesión en Docker Hub de manera segura
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USR', passwordVariable: 'DOCKER_PSW')]) {
                        sh '''
                            echo $DOCKER_PSW | docker login -u $DOCKER_USR --password-stdin
                        '''
                        
                        // Etiquetar la imagen y empujarla a Docker Hub
                        sh '''
                            docker tag ${IMAGE_NAME}:${IMAGE_TAG} $DOCKER_USR/${IMAGE_NAME}:${IMAGE_TAG}
                            docker push $DOCKER_USR/${IMAGE_NAME}:${IMAGE_TAG}
                        '''
                    }
                }
            }
        }
    }
}