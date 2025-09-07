pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO   = "759896056893.dkr.ecr.us-east-1.amazonaws.com/ft"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Kasimalbatros/add-paramters-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "build-${env.BUILD_NUMBER}"
                    sh "docker build -t ft:latest ."
                    sh "docker tag ft:latest ${ECR_REPO}:${imageTag}"
                    sh "docker tag ft:latest ${ECR_REPO}:latest"  // Also tag as latest
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'kas', region: "${AWS_REGION}") {
                    script {
                        def imageTag = "build-${env.BUILD_NUMBER}"
                        sh """
                            aws ecr get-login-password --region ${AWS_REGION} \
                                | docker login --username AWS --password-stdin ${ECR_REPO}
                            docker push ${ECR_REPO}:${imageTag}
                            docker push ${ECR_REPO}:latest
                        """
                    }
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                sh """
                    docker rmi ft:latest || true
                    docker rmi ${ECR_REPO}:build-${env.BUILD_NUMBER} || true
                    docker rmi ${ECR_REPO}:latest || true
                """
            }
        }
    }
    
    post {
        always {
            echo "Pipeline completed for build ${env.BUILD_NUMBER}"
        }
    }
}
