pipeline {
  agent any
  environment {
    AWS_REGION = 'us-east-1'
    EKS_CLUSTER = 'trend-eks-cluster'
  }
  stages {
    stage('Checkout Source') {
      steps {
        git url: 'https://github.com/parthiban4626/trend-app.git', branch: 'main'
      }
    }
    stage('Build React App') {
  steps {
    sh '''
      export NVM_DIR="$HOME/.nvm"
      [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
      nvm use 12
      npm install
      npm run build
    '''
  }
}
    }
    stage('Configure Kubeconfig') {
      steps {
        sh '''
          aws eks update-kubeconfig --region us-east-1 --name trend-eks-cluster
        '''
      }
    }
    stage('Deploy to EKS') {
      steps {
        sh '''
          kubectl apply -f k8s/trend-deployment.yaml
          kubectl rollout status deployment/trend-deployment
        '''
      }
    }
  }
  post {
    success {
      echo 'Build and deployment completed successfully!'
    }
    failure {
      echo 'Build or deployment failed.'
    }
  }
}
