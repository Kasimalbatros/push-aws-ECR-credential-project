pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO   = "759896056893.dkr.ecr.us-east-1.amazonaws.com/ft"
    }

    stages {
        stage('Verify AWS Setup') {
            steps {
                script {
                    sh """
                        echo "Checking AWS CLI configuration..."
                        aws --version
                        
                        echo "Testing AWS credentials..."
                        aws sts get-caller-identity
                        
                        echo "Checking ECR repository access..."
                        aws ecr describe-repositories --repository-names ft --region ${AWS_REGION}
                        
                        echo "✅ AWS setup verified successfully"
                    """
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Kasimalbatros/push-aws-ECR-credential-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "build-${env.BUILD_NUMBER}"
                    sh """
                        echo "Building Docker image..."
                        docker build -t ft:latest .
                        
                        echo "Tagging images for ECR..."
                        docker tag ft:latest ${ECR_REPO}:${imageTag}
                        docker tag ft:latest ${ECR_REPO}:latest
                        
                        echo "✅ Docker images built and tagged successfully"
                    """
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    def imageTag = "build-${env.BUILD_NUMBER}"
                    
                    sh """
                        echo "Logging into Amazon ECR..."
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                        
                        echo "Pushing ${ECR_REPO}:${imageTag}..."
                        docker push ${ECR_REPO}:${imageTag}
                        
                        echo "Pushing ${ECR_REPO}:latest..."
                        docker push ${ECR_REPO}:latest
                        
                        echo "✅ All images successfully pushed to ECR"
                        
                        # List the pushed images
                        echo "Pushed images:"
                        aws ecr list-images --repository-name ft --region ${AWS_REGION}
                    """
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                script {
                    def imageTag = "build-${env.BUILD_NUMBER}"
                    sh """
                        echo "Cleaning up local Docker images..."
                        docker rmi ft:latest || true
                        docker rmi ${ECR_REPO}:${imageTag} || true
                        docker rmi ${ECR_REPO}:latest || true
                        echo "✅ Cleanup completed"
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo "🚀 Pipeline execution completed for build ${env.BUILD_NUMBER}"
        }
        success {
            echo "🎉 SUCCESS: All stages completed successfully!"
            echo "📦 Images pushed to: ${ECR_REPO}"
        }
        failure {
            echo "❌ FAILURE: Pipeline execution failed"
            echo "Check the logs above for detailed error information"
        }
    }
}
