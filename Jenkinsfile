pipeline {
  agent any

  environment {
    AWS_REGION = "us-east-1"
    REPO_NAME = "cloud-native-app"
    ACCOUNT_ID = "989961910775"
    IMAGE_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
    IMAGE_TAG = "build-${BUILD_NUMBER}"
    CLUSTER_NAME = "cloud-native-cluster"
  }

  stages {
    stage('Checkout Code') {
      steps {
        git branch: 'main', url: 'https://github.com/gitthejesh/cloud-native-app.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE_URI:$IMAGE_TAG .'
      }
    }

    stage('Login to AWS ECR') {
      steps {
        withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
          sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $IMAGE_URI'
        }
      }
    }

    stage('Push Docker Image to ECR') {
      steps {
        sh 'docker push $IMAGE_URI:$IMAGE_TAG'
      }
    }

    stage('Deploy to EKS') {
      steps {
        withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
          sh '''
            aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
            kubectl set image deployment/cloud-native-app app=$IMAGE_URI:$IMAGE_TAG --record
          '''
        }
      }
    }
  }
}
