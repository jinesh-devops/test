pipeline {
  agent any

  environment {
    AWS_REGION = "us-east-1"
    ECR_URL = "245003610077.dkr.ecr.us-east-1.amazonaws.com/hello-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t hello-app:$IMAGE_TAG .'
      }
    }

    stage('Security Scan - Trivy') {
      steps {
        sh '''
          trivy image \
          --severity HIGH,CRITICAL \
          --exit-code 1 \
          hello-app:$IMAGE_TAG
        '''
      }
    }

    stage('Push to ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region $AWS_REGION | \
          docker login --username AWS --password-stdin $ECR_URL

          docker tag hello-app:$IMAGE_TAG $ECR_URL:$IMAGE_TAG
          docker push $ECR_URL:$IMAGE_TAG
        '''
      }
    }

    stage('Deploy to EKS') {
      steps {
        sh '''
          aws eks update-kubeconfig --region us-east-1 --name demo-eks
          sed -i "s|IMAGE_PLACEHOLDER|$ECR_URL:$IMAGE_TAG|" k8s/deployment.yaml
          kubectl apply -f k8s/
        '''
      }
    }
  }
}

