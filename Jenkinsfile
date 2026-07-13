pipeline {
    agent any
    
    environment {
        // Configuración de registro[cite: 1]
        REGISTRY = 'ghcr.io'
        // Nombre de la imagen (formato: usuario/repo en minúsculas)[cite: 1]
        IMAGE_NAME = 'a-bpooks/jenkins-pipeline-practica'
        
        // Variables de versión[cite: 1]
        COMMIT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
        
        // Tags de la imagen[cite: 1]
        IMAGE_TAG_LATEST = "${REGISTRY}/${IMAGE_NAME}:latest"
        IMAGE_TAG_COMMIT = "${REGISTRY}/${IMAGE_NAME}:${COMMIT_SHA}"
        IMAGE_TAG_BUILD = "${REGISTRY}/${IMAGE_NAME}:build-${BUILD_TIMESTAMP}"
    }
    
    stages {
        // ============================================
        // STAGE 1: Preparación[cite: 1]
        // ============================================
        stage('Prepare') {
            steps {
                echo ' Preparando entorno...'
                sh 'docker --version'
                sh 'node --version'
                sh 'npm --version'
            }
        }
        
        // ============================================
        // STAGE 2: Instalación de Dependencias[cite: 1]
        // ============================================
        stage('Install Dependencies') {
            steps {
                echo '📥 Instalando dependencias...'
                sh 'npm ci'
            }
        }
        
        // ============================================
        // STAGE 3: Ejecutar Tests[cite: 1]
        // ============================================
        stage('Test') {
            steps {
                echo '🧪 Ejecutando tests...'
                sh 'npm test'
            }
        }
        
        // ============================================
        // STAGE 4: Construcción de Imagen Docker[cite: 1]
        // ============================================
        stage('Build Docker Image') {
            steps {
                echo '🐳 Construyendo imagen Docker...'
                script {
                    docker.build("${IMAGE_TAG_COMMIT}")
                }
            }
        }
        
        // ============================================
        // STAGE 5: Publicación en Registro[cite: 1]
        // ============================================
        stage('Push to Registry') {
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
        
        // ============================================
        // STAGE 6: Verificación[cite: 1]
        // ============================================
        stage('Verify Published Image') {
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
    
    // ============================================
    // POST: Acciones finales[cite: 1]
    // ============================================
    post {
        cleanup {
            echo '🧹 Limpiando recursos...'
            script {
                // Limpiar imágenes locales para ahorrar espacio[cite: 1]
                sh "docker image prune -f"
            }
        }
    }
}