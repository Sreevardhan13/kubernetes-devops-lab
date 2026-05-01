pipeline {
    agent any

    environment {
        // Replace with your Docker Hub username and image name
        DOCKER_IMAGE = "your-dockerhub-username/flask-app"
        AWS_REGION = "us-east-1"
        CLUSTER_NAME = "devops-lab-cluster"
    }

    stages {
        stage('Checkout') {
            steps {
                // This pulls your latest code from the GitHub repo you link in Jenkins
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Builds the image using the Dockerfile in your folder
                    sh "docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} ."
                    sh "docker tag ${DOCKER_IMAGE}:${env.BUILD_NUMBER} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                // Uses the 'docker-hub-creds' ID we created in the Jenkins UI
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    // Updates the deployment.yaml with the new image version
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${env.BUILD_NUMBER}|g' k8s/deployment.yaml"
                    
                    // Tells EKS to apply the changes
                    // Note: Ensure you have installed the 'Kubernetes CLI' plugin
                    sh "kubectl apply -f k8s/deployment.yaml"
                }
            }
        }
    }

    post {
        always {
            // Clean up the local images to save disk space on the EC2 instance
            sh "docker rmi ${DOCKER_IMAGE}:${env.BUILD_NUMBER} || true"
        }
    }
}