pipeline {
    agent any

    environment {
        KUBECONFIG_PATH = 'C:/Users/IT-WORKSTATION/Desktop/flask-app/minikube-kubeconfig.yaml'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DEPLOYMENT_NAME = 'flask-app'
        CONTAINER_NAME = 'flask-app'
        DOCKER_IMAGE = 'yemisi76/flask-app'
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Yemmmyc/flask-app.git'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat """
                    echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                    """
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                bat """
                docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .
                docker push %DOCKER_IMAGE%:%IMAGE_TAG%
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat """
                kubectl --kubeconfig=%KUBECONFIG_PATH% apply -f k8s/
                kubectl --kubeconfig=%KUBECONFIG_PATH% set image deployment/%DEPLOYMENT_NAME% %CONTAINER_NAME%=%DOCKER_IMAGE%:%IMAGE_TAG%
                """
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Pipeline failed! Check logs for errors.'
        }
    }
}
