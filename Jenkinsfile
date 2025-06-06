pipeline {
    agent any  // Run on any available Jenkins agent

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  
        KUBECONFIG_FILE = credentials('kubeconfig')
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("yemisi76/flask-app:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    script {
                        if (isUnix()) {
                            sh """
                            kubectl --kubeconfig=$KUBECONFIG set image deployment/flask-app-deployment flask-app=yemisi76/flask-app:${env.BUILD_NUMBER} --record
                            """
                        } else {
                            bat """
                            kubectl --kubeconfig=%KUBECONFIG% set image deployment/flask-app-deployment flask-app=yemisi76/flask-app:${env.BUILD_NUMBER} --record
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check logs.'
        }
    }
}
