pipeline {
    agent any

    environment {
        // App & build config
        APP_NAME        = "xv3i01a5"
        MX_VERSION      = "10.24"
        K8S_NAMESPACE   = "mxdev"

        // AWS / ECR
        AWS_REGION      = "us-east-1"
        AWS_ACCOUNT_ID = "320368024572"
        ECR_REPO        = "aws-ecr"
        ECR_REGISTRY    = "320368024572.dkr.ecr.us-east-1.amazonaws.com"
        IMAGE_TAG       = "mxdev-xv3i01a5-${BUILD_NUMBER}"

        // Mendix build image
        MXBUILD_IMAGE   = "320368024572.dkr.ecr.us-east-1.amazonaws.com/aws-ecr:mxdev-build-10.24"
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
                    bat """
                    aws ecr get-login-password --region us-east-1 ^
                    | docker login --username AWS --password-stdin 320368024572.dkr.ecr.us-east-1.amazonaws.com
                    """
                }
            }
        }

        stage('Build Mendix App (.mda)') {
            steps {
                echo "🏗️ Building Mendix MDA"
                bat """
                docker run --rm ^
                  -v "%cd%:/project" ^
                  "320368024572.dkr.ecr.us-east-1.amazonaws.com/aws-ecr:mxdev-build-10.24" ^
                  mxbuild ^
                  --project-directory=/project ^
                  --target=dist/xv3i01a5.mda
                """
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                echo "🐳 Building and pushing Docker image"
                bat """
                docker build ^
                  -t 320368024572.dkr.ecr.us-east-1.amazonaws.com/aws-ecr:mxdev-xv3i01a5-${BUILD_NUMBER} .

                docker push 320368024572.dkr.ecr.us-east-1.amazonaws.com/aws-ecr:mxdev-xv3i01a5-${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying to Kubernetes"
                bat """
                kubectl set image deployment/xv3i01a5 ^
                  xv3i01a5=320368024572.dkr.ecr.us-east-1.amazonaws.com/aws-ecr:mxdev-xv3i01a5-${BUILD_NUMBER} ^
                  -n mxdev

                kubectl rollout status deployment/xv3i01a5 -n mxdev
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
