pipeline {
  agent any

  environment {
    DOCKER_REGISTRY = "yourdockerhubusername"
    BACKEND_IMAGE = "${DOCKER_REGISTRY}/myapp-backend"
    FRONTEND_IMAGE = "${DOCKER_REGISTRY}/myapp-frontend"
    TAG = "build-${env.BUILD_NUMBER}"
    BACKEND_DIR = "backend"
    FRONTEND_DIR = "."
    DEPLOY_DIR = "/home/ubuntu/app"
    DEPLOY_HOST = "your.server.ip.or.domain"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Backend Image') {
      steps {
        sh """
          docker build -t ${BACKEND_IMAGE}:${TAG} ${BACKEND_DIR}
        """
      }
    }

    stage('Build Frontend Image') {
      steps {
        dir("${FRONTEND_DIR}") {
          sh """
            docker build -f Dockerfile.frontend -t ${FRONTEND_IMAGE}:${TAG} .
          """
        }
      }
    }

    stage('Push to Registry') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
          sh '''
            echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin

            docker tag ${BACKEND_IMAGE}:${TAG} ${BACKEND_IMAGE}:latest
            docker tag ${FRONTEND_IMAGE}:${TAG} ${FRONTEND_IMAGE}:latest

            docker push ${BACKEND_IMAGE}:${TAG}
            docker push ${BACKEND_IMAGE}:latest
            docker push ${FRONTEND_IMAGE}:${TAG}
            docker push ${FRONTEND_IMAGE}:latest
          '''
        }
      }
    }

    stage('Deploy to Server') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'deploy-ssh-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'DEPLOY_SSH_USER')]) {
          sh """
            chmod 600 "$SSH_KEY" || true
            scp -o StrictHostKeyChecking=no -i "$SSH_KEY" docker-compose.yml ${DEPLOY_SSH_USER}@${DEPLOY_HOST}:${DEPLOY_DIR}/docker-compose.yml
            ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ${DEPLOY_SSH_USER}@${DEPLOY_HOST} 'cd ${DEPLOY_DIR} && docker compose pull && docker compose up -d --remove-orphans'
          """
        }
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
  }
}
