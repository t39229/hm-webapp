properties([
    pipelineTriggers([
        githubPush()
    ])
])

pipeline {
    agent any
    
    options {
        skipDefaultCheckout(false)
    }
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_NAME = 'nginx-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS = credentials('21285o6')
        GITHUB_TOKEN = credentials('github-token')
        REPOSITORY_URL = 'https://github.com/t39229/k8s-demo-app.git'
    }
    
    stages {
        stage('Test GitHub Connection') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh 'git ls-remote -h ${REPOSITORY_URL}'
                }
            }
        }

        stage('Checkout') {
            steps {
                cleanWs()
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: env.REPOSITORY_URL
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: '21285o6', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo \$DOCKER_PASS | docker login ${DOCKER_REGISTRY} -u \$DOCKER_USER --password-stdin
                        docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        git config --global user.email "t39163463@gmail.com"
                        git config --global user.name "t39229"
                        sed -i 's|image: .*|image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
                        git add k8s/deployment.yaml
                        git commit -m "Update image tag to ${IMAGE_TAG}"
                        git push https://\${GITHUB_TOKEN}@github.com/t39229/k8s-demo-app.git main
                    """
                }
            }
        }
    }
    
    post {
        always {
            script {
                cleanWs()
            }
        }
        failure {
            echo 'Pipeline failed'
        }
        success {
            echo 'Pipeline succeeded'
        }
    }
}
