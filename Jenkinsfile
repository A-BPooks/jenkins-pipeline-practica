pipeline {
    agent any
    
    environment {
        // Configuración de registro
        REGISTRY = 'ghcr.io'
        // Nombre de la imagen (formato: usuario/repo en minúsculas)
        IMAGE_NAME = 'a-bpooks/jenkins-pipeline-practica'
        
        // Variables de versión
        COMMIT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
        
        // Tags de la imagen
        IMAGE_TAG_LATEST = "${REGISTRY}/${IMAGE_NAME}:latest"
        IMAGE_TAG_COMMIT = "${REGISTRY}/${IMAGE_NAME}:${COMMIT_SHA}"
        IMAGE_TAG_BUILD = "${REGISTRY}/${IMAGE_NAME}:build-${BUILD_TIMESTAMP}"
    }
    
    stages {
        stage('Prepare') {
            steps {
                echo ' Preparando entorno...'
                sh 'docker --version'
                sh 'node --version'
                sh 'npm --version'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo '📥 Instalando dependencias...'
                sh 'npm ci'
            }
        }
        
        stage('Test') {
            steps {
                echo '🧪 Ejecutando tests...'
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '🐳 Construyendo imagen Docker...'
                script {
                    docker.build("${IMAGE_TAG_COMMIT}")
                }
            }
        }
        
        stage('Push to Registry') {
            when {
                branch 'main'
            }
            steps {
                echo '📤 Publicando imagen en GitHub Container Registry...'
                withCredentials([
                    string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')
                ]) {
                    sh '''
                        echo $GITHUB_TOKEN | docker login ghcr.io -u a-bpooks --password-stdin
                        docker tag ${IMAGE_TAG_COMMIT} ${IMAGE_TAG_LATEST}
                        docker tag ${IMAGE_TAG_COMMIT} ${IMAGE_TAG_BUILD}
                        
                        docker push ${IMAGE_TAG_COMMIT}
                        docker push ${IMAGE_TAG_LATEST}
                        docker push ${IMAGE_TAG_BUILD}
                    '''
                }
            }
        }
        
        stage('Verify Published Image') {
            when {
                branch 'main'
            }
            steps {
                echo '✅ Verificando imagen publicada...'
                script {
                    sh """
                        echo "📦 Imagen publicada: ${IMAGE_TAG_COMMIT}"
                        echo "🏷️ Tags disponibles:"
                        echo "  - ${IMAGE_TAG_COMMIT}"
                        echo "  - ${IMAGE_TAG_LATEST}"
                        echo "  - ${IMAGE_TAG_BUILD}"
                    """
                }
            }
        }
    }
    
    post {
        cleanup {
            echo '🧹 Limpiando recursos...'
            script {
                sh "docker image prune -f"
            }
        }
    }
}