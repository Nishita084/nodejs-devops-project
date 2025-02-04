pipeline {
    agent any

    environment {
        // Define environment variables (e.g., Docker registry, repository URL, etc.)
        ECR_REGISTRY = '588738609043.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = 'latest-18'
        AWS_REGION = 'us-east-1'
        GITHUB_REPO = 'https://github.com/Nishita084/nodejs-devops-project.git'
        DOCKERFILE_PATH = 'src/Dockerfile'
        EC2_INSTANCE = 'ec2-user@your-ec2-instance-public-ip'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // Checkout the source code from the repository
                checkout scm
            }
        }

        stage('Clone Repository') {
            steps {
                // Clone the repository again for safety or to reset workspace
                script {
                    git url: "${GITHUB_REPO}"
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build and tag Docker image using the Dockerfile located in 'src'
                    sh """
                    docker build -f ${DOCKERFILE_PATH} -t ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                    
                    // Log in to AWS ECR
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """

                    // Push the Docker image to the ECR repository
                    sh """
                    docker push ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to EC2 Instance') {
            when {
                // Only run this stage if previous stages were successful
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    // Deploy the Docker image to the EC2 instance
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_INSTANCE} "
                        docker pull ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} && 
                        docker run -d --name ${IMAGE_NAME} ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    "
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up after each run
            cleanWs()
        }

        success {
            // Send success message to Slack or other notification
            slackSend (color: 'good', message: "Build and deployment successful!")
        }

        failure {
            // Send failure message to Slack or other notification
            slackSend (color: 'danger', message: "Build or deployment failed!")
        }
    }
}
