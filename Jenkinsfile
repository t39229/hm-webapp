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
        DOCKERHUB_CREDENTIALS = credentials('docker-hub')  // Changed to match credentials ID
        GITHUB_TOKEN = credentials('github-token')
        REPOSITORY_URL = 'https://github.com/t39229/k8s-demo-app.git'
    }
    
    stages {
        stage('Test GitHub Connection') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                       git ls-remote -h https://${GITHUB_TOKEN}@github.com/t39229/k8s-demo-app.git
                    '''
                }
            }
        }

        stage('Checkout') {
            steps {
               cleanWs()
               git branch: 'main',
                  url: "https://${GITHUB_TOKEN}@github.com/t39229/k8s-demo-app.git"
            }
        }
        
        stage('Docker Login') {    // Fixed syntax error in stage name
            steps {
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_CREDENTIALS_USR}/${IMAGE_NAME}:${IMAGE_TAG} ."  // Added username prefix
            }
        }
        
        stage('Push Docker Image') {
            steps {
                sh """
                    docker push ${DOCKERHUB_CREDENTIALS_USR}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        git config --global user.email "t39163463@gmail.com"
                        git config --global user.name "t39229"
                        sed -i 's|image: .*|image: ${DOCKERHUB_CREDENTIALS_USR}/${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
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
            sh 'docker logout'  // Added explicit logout
            node('built-in') {
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