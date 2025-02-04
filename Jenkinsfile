pipeline {
    agent any
    
    environment {
        AWS_ACCOUNT_ID = "588738609043"  // Your AWS Account ID
        AWS_REGION = "us-east-1"         // Your AWS region
        IMAGE_NAME = "myapp"             // Docker image name
        ECR_URI = "588738609043.dkr.ecr.us-east-1.amazonaws.com/myapp"  // ECR URI
        EC2_PUBLIC_IP = "18.234.134.27"  // Replace with your EC2-2 public IP
        GITHUB_REPO = 'https://github.com/Nishita084/nodejs-devops-project.git'  // Your GitHub repository URL
        GITHUB_CREDENTIALS_ID = 'github-token'  // Your GitHub credentials ID in Jenkins
        EC2_KEY = 'ec2-key'  // SSH key for EC2 instance
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                // Clone the repository from GitHub
                git branch: 'main', url: "${GITHUB_REPO}", credentialsId: "${GITHUB_CREDENTIALS_ID}"
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image
                sh 'docker build -t myapp:latest -f Dockerfile .'

            }
        }

        stage('Login to AWS ECR') {
            steps {
                // Login to AWS ECR using the IAM role credentials
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URI
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                // Tag the Docker image and push it to ECR
                sh '''
                docker tag $IMAGE_NAME:latest $ECR_URI:latest
                docker push $ECR_URI:latest
                '''
            }
        }

        stage('Deploy to EC2 Instance') {
            steps {
                sshagent(['${EC2_KEY}']) {
                    // SSH into EC2 instance and deploy the app
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$EC2_PUBLIC_IP << EOF
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URI
                    docker pull $ECR_URI:latest
                    docker stop myapp || true
                    docker rm myapp || true
                    docker run -d -p 3000:3000 --name myapp $ECR_URI:latest
                    EOF
                    '''
                }
            }
        }
    }
    
    post {
        success {
            // Notify on successful deployment
            echo "Deployment to EC2 was successful!"
        }
        failure {
            // Notify on failure
            echo "Deployment failed!"
        }
    }
}
