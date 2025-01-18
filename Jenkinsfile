pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_NAME = 'nginx-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://docker.io', 't39229') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    // Update image tag in deployment.yaml
                    sh """
                        sed -i 's|image: .*|image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
                        git config --global user.email "jenkins@example.com"
                        git config --global user.name "Jenkins"
                        git add k8s/deployment.yaml
                        git commit -m "Update image tag to ${IMAGE_TAG}"
                        git push origin main
                    """
                }
            }
        }
    }
}
