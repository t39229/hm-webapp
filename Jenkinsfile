properties([
    pipelineTriggers([
        githubPush()
    ])
])

pipeline {
    agent any
    
    options {
        skipDefaultCheckout(false)
        disableConcurrentBuilds() // Prevents parallel builds
        quietPeriod(60) // Adds a 60-second delay before starting a new build
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub')
        GITHUB_TOKEN = credentials('github-token')
        IMAGE_NAME = 'nginx-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
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
        
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    '''
                }
            }
        }

        stage('Build and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh """
                        docker build -t \${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .
                        docker push \${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                withCredentials([
                    string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN'),
                    usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')
                ]) {
                    sh """
                        git config --global user.email "t39163463@gmail.com"
                        git config --global user.name "t39229"
                        sed -i 's|image: .*|image: '\${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}'|' k8s/deployment.yaml
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
            sh 'docker logout'
            cleanWs()
        }
        failure {
            echo 'Pipeline failed'
        }
        success {
            echo 'Pipeline succeeded'
        }
    }
}
