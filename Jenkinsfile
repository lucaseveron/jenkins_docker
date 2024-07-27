pipeline {
    agent any
    
    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' // ID de las credenciales de Docker en Jenkins
        IMAGE_NAME = 'my-app' // Nombre de la imagen
        IMAGE_TAG = 'latest' // Etiqueta de la imagen
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gastonbarbaccia/jenkins_terraform_s3_demo.git'
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
                    // Iniciar sesión en el registro Docker
                    withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS_ID}", url: "https://${DOCKER_REGISTRY}"]) {
                        // Etiquetar la imagen con el registro y subirla
                        sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }
        
    }
}
