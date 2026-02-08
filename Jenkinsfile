pipeline {
    agent any

    environment {
        APP_NAME      = "xv3i01a5"
        MX_VERSION   = "10.24"
        K8S_NAMESPACE = "mxdev"

        AWS_REGION    = "us-east-1"
        AWS_ACCOUNT_ID = "320368024572"
        ECR_REPO      = "aws-ecr"
        ECR_REGISTRY  = "320368024572.dkr.ecr.us-east-1.amazonaws.com"
        IMAGE_TAG     = "${BUILD_NUMBER}"

        MXBUILD_IMAGE = "320368024572.dkr.ecr.us-east-1.amazonaws.com/aws-ecr:mxdev-build-10.24"
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
                echo "🔐 Logging in to AWS ECR repo"
                withCredentials([[ 
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr-creds'
                ]]) {
                    bat """
                    aws ecr get-login-password --region %AWS_REGION% ^
                    | docker login --username AWS --password-stdin %ECR_REGISTRY%
                    """
                }
            }
        }



        stage('Build Mendix App Image') {
      steps {
        sh '''
          docker build -t $ECR_REPO:$IMAGE_TAG .
          docker tag $ECR_REPO:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
        '''
      }
    }

        stage('Build & Push Docker Image') {
            steps {
                echo "🐳 Building and pushing Docker image"
                bat """
                docker build ^
                  -t %ECR_REGISTRY%/%ECR_REPO%:%IMAGE_TAG% .

                docker push %ECR_REGISTRY%/%ECR_REPO%:%IMAGE_TAG%
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying to Kubernetes"
                bat """
                kubectl set image deployment/%APP_NAME% ^
                  %APP_NAME%=%ECR_REGISTRY%/%ECR_REPO%:%IMAGE_TAG% ^
                  -n %K8S_NAMESPACE%

                kubectl rollout status deployment/%APP_NAME% -n %K8S_NAMESPACE%
                """
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
