pipeline {
  agent any
  environment {
    FRONT_IMAGE = "tipu1/frontend:latest"
    BACK_IMAGE  = "tipu1/backend:latest"
    AWS_REGION  = "us-east-1"
  }
  stages {
    stage('Clone Code') {
      steps {
        git branch: 'main', url: 'https://github.com/tipusultan-cloud/sultan-devops-portfolio.git'
      }
    }
    stage('Build Docker Images') {
      steps {
        sh 'docker build -t $FRONT_IMAGE ./frontend'
        sh 'docker build -t $BACK_IMAGE ./backend'
      }
    }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
          echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
          docker push $FRONT_IMAGE
          docker push $BACK_IMAGE
          '''
        }
      }
    }
    stage('Deploy to EKS') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
          sh '''
          aws eks update-kubeconfig --region $AWS_REGION --name sultan-cluster
          kubectl apply -f k8s/
          '''
        }
      }
    }
  }
}
// Jenkins pipeline goes here
