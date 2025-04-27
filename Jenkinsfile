pipeline {
  agent any

  environment {
    AWS_REGION = "us-east-1"
    REPO_NAME = "cloud-native-app"
    ACCOUNT_ID = "989961910775"
    IMAGE_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
    CLUSTER_NAME = "cloud-native-cluster"
  }

  stages {
    stage('Checkout Code') {
      steps {
        echo 'Code checkout step - done locally (GitHub later)'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE_URI:v1 .'
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
        sh 'docker push $IMAGE_URI:v1'
      }
    }

    stage('Deploy to EKS') {
      steps {
        withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
          sh '''
            aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
            kubectl set image deployment/cloud-native-app app=$IMAGE_URI:v1 --record
          '''
        }
      }
    }
  }
}
