pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-creds-id' 
    DOCKERHUB_REPO = 'parthiban46/trend-react-app' 
    IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
    AWS_REGION = 'us-east-1'
    EKS_CLUSTER = 'trend-eks-cluster'
  }
  stages {
    stage('Checkout Source') {
      steps {
        git url: 'https://github.com/parthiban4626/trend-app.git', branch: 'main'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh '''
          export NVM_DIR="$HOME/.nvm"
          [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
          nvm use 12
          npm install
          npm run build
          docker build -t $DOCKERHUB_REPO:$IMAGE_TAG .
        '''
      }
    }
    stage('Login to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
        }
      }
    }
    stage('Push Docker Image') {
      steps {
        sh '''
          docker push $DOCKERHUB_REPO:$IMAGE_TAG
        '''
      }
    }
    stage('Configure Kubeconfig') {
      steps {
        sh '''
          aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER
        '''
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          # Update image in deployment YAML if needed or patch deployment directly
          kubectl set image deployment/trend-app-deployment trend-app-container=$DOCKERHUB_REPO:$IMAGE_TAG
          kubectl rollout status deployment/trend-deployment
          # Apply service manifest if not already applied or changed
          kubectl apply -f k8s/trend-service.yaml
        '''
      }
    }
  }
  post {
    success {
      echo 'Build, push, and deployment completed successfully!'
    }
    failure {
      echo 'Pipeline failed. Check logs for details.'
    }
  }
}
