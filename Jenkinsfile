pipeline {
    agent any

    environment {
        KUBECONFIG_PATH = 'C:/Users/IT-WORKSTATION/Desktop/flask-app/minikube-kubeconfig.yaml'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DEPLOYMENT_NAME = 'flask-app-deployment'
        CONTAINER_NAME = 'flask-app'
        DOCKER_IMAGE = 'yemisi76/flask-app'
        K8S_MANIFEST_DIR = 'k8s'
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Yemmmyc/flask-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat """
                    docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %DOCKER_IMAGE%:%IMAGE_TAG%
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat """
                kubectl --kubeconfig=%KUBECONFIG_PATH% apply -f %K8S_MANIFEST_DIR%/
                kubectl --kubeconfig=%KUBECONFIG_PATH% set image deployment/%DEPLOYMENT_NAME% %CONTAINER_NAME%=%DOCKER_IMAGE%:%IMAGE_TAG%
                """
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed! Check logs for errors.'
        }
        success {
            echo 'Deployment successful!'
        }
    }
}
