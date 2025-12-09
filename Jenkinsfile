pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "arshiya12334"
        IMAGE_NAME = "myapp"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Arshiya1305/myapp.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Login & Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([string(credentialsId: 'kubeconfig', variable: 'KCFG')]) {
                    sh '''
                    echo "$KCFG" > kubeconfig
                    export KUBECONFIG=$(pwd)/kubeconfig
                    kubectl set image deployment/myapp-deployment myapp-container=${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                    kubectl rollout status deployment/myapp-deployment
                    '''
                }
            }
        }
    }
}

