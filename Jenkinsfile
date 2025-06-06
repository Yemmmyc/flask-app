pipeline {
    agent any  // Run on any available Jenkins agent

    environment {
        // Jenkins credentials IDs for DockerHub and kubeconfig file
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  
        KUBECONFIG_FILE = credentials('kubeconfig')
    }

    stages {
        stage('Checkout Source') {
            steps {
                // Checkout source code from your Git repo (automatic in pipeline from SCM)
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image tagged with your DockerHub username and Jenkins build number
                    dockerImage = docker.build("yemisi76/flask-app:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Login and push the built Docker image to DockerHub
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Use kubeconfig file stored as Jenkins secret to connect to Kubernetes cluster
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    // Update the image of the existing deployment with the new image version
                    sh """
                    kubectl --kubeconfig=$KUBECONFIG set image deployment/flask-app-deployment \
                    flask-app=your-dockerhub-username/flask-app:${env.BUILD_NUMBER} --record
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace after build to save space
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
