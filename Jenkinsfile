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
#    stage('Scan with Trivy') {
#      steps {
#        script {
#          sh "TRIVY_CACHE_DIR=${env.TRIVY_CACHE_DIR} XDG_CACHE_HOME=${env.XDG_CACHE_HOME} trivy image --exit-code 1 --severity HIGH,CRITICAL ${env.DOCKER_IMAGE_NAME}"
#        }
#      }
#    }
    stage('Push Docker Image') {
      steps {
        script {
          docker.withRegistry("https://${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com", 'ecr:aws-credentials') {
            dockerImage.push()
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

