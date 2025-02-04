pipeline {
    agent any
    
    environment {
        AWS_ACCOUNT_ID = "588738609043"
        AWS_REGION = "us-east-1"
        IMAGE_NAME = "myapp"
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}"
        EC2_PUBLIC_IP = "18.234.134.27"
        GITHUB_REPO = 'https://github.com/Nishita084/nodejs-devops-project.git'
        GITHUB_CREDENTIALS_ID = 'github-token'
        EC2_KEY = 'ec2-key'
        DOCKER_TAG = "latest-${BUILD_NUMBER}"  // Unique tag per build
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: "${GITHUB_REPO}", credentialsId: "${GITHUB_CREDENTIALS_ID}"
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build with a unique tag
                    sh "docker build -t ${ECR_URI}:${DOCKER_TAG} -t ${ECR_URI}:latest ."
                    
                    // Login to ECR
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}"
                    
                    // Push both tags
                    sh "docker push ${ECR_URI}:${DOCKER_TAG}"
                    sh "docker push ${ECR_URI}:latest"
                }
            }
        }

        stage('Deploy to EC2 Instance') {
            steps {
                sshagent([EC2_KEY]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_PUBLIC_IP} '
                            # Pull the latest image
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}
                            docker pull ${ECR_URI}:${DOCKER_TAG}
                            
                            # Stop and remove old container
                            docker stop myapp || true
                            docker rm myapp || true
                            
                            # Run new container
                            docker run -d --restart unless-stopped -p 3000:3000 --name myapp ${ECR_URI}:${DOCKER_TAG}
                        '
                    """
                }
            }
        }
    }
    
    post {
        success {
            slackSend(color: 'good', message: "Deployment succeeded: ${BUILD_URL}")
        }
        failure {
            slackSend(color: 'danger', message: "Deployment failed: ${BUILD_URL}")
        }
    }
}