pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO   = "759896056893.dkr.ecr.us-east-1.amazonaws.com/ft"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Kasimalbatros/push-aws-ECR-credential-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "build-${env.BUILD_NUMBER}"
                    sh "docker build -t ft:latest ."
                    sh "docker tag ft:latest ${ECR_REPO}:${imageTag}"
                    sh "docker tag ft:latest ${ECR_REPO}:latest"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    def imageTag = "build-${env.BUILD_NUMBER}"
                    
                    // Use AWS CLI for ECR login (no plugin required)
                    sh """
                        # Login to ECR using AWS CLI
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                        
                        # Push the images to ECR
                        docker push ${ECR_REPO}:${imageTag}
                        docker push ${ECR_REPO}:latest
                        
                        # Verify the push was successful
                        echo "Successfully pushed to ECR:"
                        echo "- ${ECR_REPO}:${imageTag}"
                        echo "- ${ECR_REPO}:latest"
                    """
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                script {
                    def imageTag = "build-${env.BUILD_NUMBER}"
                    sh """
                        # Remove local Docker images to free up space
                        docker rmi ft:latest || echo "ft:latest not found"
                        docker rmi ${ECR_REPO}:${imageTag} || echo "${ECR_REPO}:${imageTag} not found"
                        docker rmi ${ECR_REPO}:latest || echo "${ECR_REPO}:latest not found"
                        
                        echo "Cleanup completed successfully"
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline execution completed for build ${env.BUILD_NUMBER}"
        }
        success {
            echo "✅ SUCCESS: Docker images built and pushed to ECR successfully!"
            echo "ECR Repository: ${ECR_REPO}"
        }
        failure {
            echo "❌ FAILURE: Pipeline execution failed"
            echo "Check AWS credentials, ECR permissions, or network connectivity"
        }
    }
}
