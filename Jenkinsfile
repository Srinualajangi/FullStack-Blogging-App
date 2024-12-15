pipeline {
  agent any
  tools {
    maven 'Maven 3.8.8' // Name of the Maven installation configured in Jenkins
  }
  environment {
    ECR_REPO_NAME = 'my-ecr-repo'
    IMAGE_TAG = "my-ecr-repo:${env.BUILD_ID}"
    TRIVY_CACHE_DIR = '/mnt/trivy-cache'
    XDG_CACHE_HOME = '/mnt/trivy-cache'
    AWS_ACCOUNT_ID = '971422672236'
    AWS_DEFAULT_REGION = 'us-east-2'
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Srinualajangi/FullStack-Blogging-App.git', credentialsId: 'github-credentials'
      }
    }
    stage('Build') {
      steps {
        script {
          sh 'mvn clean package'
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("${env.ECR_REPO_NAME}:${env.BUILD_ID}")
          env.DOCKER_IMAGE_NAME = "${env.ECR_REPO_NAME}:${env.BUILD_ID}"
        }
      }
    }
    stage('Push Docker Image') {
      steps {
        script {
          withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'ecr:aws-credentials']]) {
            sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 971422672236.dkr.ecr.us-east-2.amazonaws.com'
            sh "docker push 971422672236.dkr.ecr.us-east-2.amazonaws.com/my-ecr-repo:${env.BUILD_ID}"
          }
        }
      }
    }
    stage('Deploy to EKS') {
      steps {
        script {
          sh '''
          aws eks --region $AWS_DEFAULT_REGION update-kubeconfig --name $CLUSTER_NAME
          kubectl apply -f k8s/deployment.yaml
          '''
        }
      }
    }
  }
}

