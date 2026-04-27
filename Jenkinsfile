pipeline {
    agent any

    options {
        timeout(time: 2, unit: 'MINUTES') // Tiempo máximo para la ejecución del pipeline
    }

    /**environment {
        NEXUS_URL = "http://localhost:8083"
        CREDENTIALS_ID = "f0142294-69d8-4e13-9215-33104e705eb6"
        IMAGE_NAME = "sumador" // Nombre de la imagen Docker
        IMAGE_TAG = "${env.BUILD_NUMBER}" // Etiqueta de la imagen basada en el número de build
        NEXUS_HOST = "localhost:8083" // Host y puerto de Nexus
        NEXUS_REPO = "repository/myrepo" // Ruta del repositorio en Nexus
        ARTIFACT_ID = "elbuo8/webapp:${env.BUILD_NUMBER}"
    }**/
    environment {
        CREDENTIALS_ID = "nexus-credentials" // ID de las credenciales almacenadas en Jenkins
        IMAGE_NAME = "sumador" // Nombre de la imagen Docker
        IMAGE_TAG = "${env.BUILD_NUMBER}" // Etiqueta de la imagen basada en el
        NEXUS_HOST = "localhost:8083" // Host y puerto de Nexus
        NEXUS_URL = "http://localhost:8083" // URL completa de Nexus
        NEXUS_REPO = "repository/myrepo" // Ruta del repositorio en Nexus
    }

    stages {
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Run tests') {
          steps {
            sh "docker run ${IMAGE_NAME}:${IMAGE_TAG} npm test"
          }
        }
        
        stage('Tag Docker Image') {
            steps {
                echo "Tagging Docker image for Nexus repository..."
                sh """
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${NEXUS_HOST}/${NEXUS_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: CREDENTIALS_ID, usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                    sh """
                    echo $NEXUS_PASSWORD | docker login ${NEXUS_HOST} -u $NEXUS_USERNAME --password-stdin
                    docker push ${NEXUS_HOST}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up local Docker images..."
            sh """
            docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
            docker rmi ${NEXUS_HOST}/${NEXUS_REPO}/${IMAGE_NAME}:${IMAGE_TAG} || true
            """
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
    }
}