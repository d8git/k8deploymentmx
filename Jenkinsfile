pipeline {
    agent any

    environment {
        APP_NAME      = "xv3i01a5"
        K8S_NAMESPACE = "mxdev"

        AWS_REGION     = "us-east-1"
        AWS_ACCOUNT_ID = "320368024572"

        ECR_REPO      = "aws-ecr"
        ECR_REGISTRY  = "320368024572.dkr.ecr.us-east-1.amazonaws.com"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📥 Checking out source code"
                checkout scm
            }
        }

        stage('Login to AWS ECR') {
            steps {
                echo "🔐 Logging in to AWS ECR"
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr-creds'
                ]]) {
                    bat '''
                    aws ecr get-login-password --region %AWS_REGION% ^
                    | docker login --username AWS --password-stdin %ECR_REGISTRY%
                    '''
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                echo "🐳 Building and pushing Docker image"
                bat '''
                docker build -t %ECR_REGISTRY%/%ECR_REPO%:%BUILD_NUMBER% .
                docker push %ECR_REGISTRY%/%ECR_REPO%:%BUILD_NUMBER%
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying to Kubernetes"
                bat '''
                kubectl set image deployment/%APP_NAME% ^
                  %APP_NAME%=%ECR_REGISTRY%/%ECR_REPO%:%BUILD_NUMBER% ^
                  -n %K8S_NAMESPACE%

                kubectl rollout status deployment/%APP_NAME% -n %K8S_NAMESPACE%
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Mendix app deployed successfully"
        }
        failure {
            echo "❌ Build or deployment failed"
        }
    }
}
