pipeline {
    agent any

    triggers {
        githubPush()
    }
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_NAME = 'nginx-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS = credentials('21285o6')
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                sh """
                    sudo docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }
        
        stage('Push Docker Image') {
            steps {
                sh """
                    echo ${DOCKER_CREDENTIALS_PSW} | docker login ${DOCKER_REGISTRY} -u ${DOCKER_CREDENTIALS_USR} --password-stdin
                    docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker logout
                """
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                sh """
                    sed -i 's|image: .*|image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
                    git config --global user.email "t39163463@gmail.com"
                    git config --global user.name "t39229"
                    git add k8s/deployment.yaml
                    git commit -m "Update image tag to ${IMAGE_TAG}"
                    git push origin main
                """
            }
        }
    }
}
