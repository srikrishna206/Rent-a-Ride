pipeline {
  agent any

  environment {
    // edit these for your registry / image names
    DOCKER_REGISTRY = "yourdockerhubusername"           // e.g. mydockeruser
    BACKEND_IMAGE = "${DOCKER_REGISTRY}/myapp-backend"
    FRONTEND_IMAGE = "${DOCKER_REGISTRY}/myapp-frontend"
    TAG = "build-${env.BUILD_NUMBER}"
    BACKEND_DIR = "backend"    // change if your backend folder differs
    FRONTEND_DIR = "."         // "." assumes frontend files are repo root
    DEPLOY_DIR = "/home/ubuntu/app" // deploy host folder where docker-compose.yml lives
    DEPLOY_HOST = "your.server.ip.or.domain"  // set in job or leave placeholder
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Backend Image') {
      steps {
        dir("${BACKEND_DIR}") {
          sh """
            docker build -t ${BACKEND_IMAGE}:${TAG} .
          """
        }
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
        // docker registry credentials: create username/password credential with id 'dockerhub-creds'
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
        // create SSH credential in Jenkins of type "SSH Username with private key" (id: 'deploy-ssh-key')
        withCredentials([sshUserPrivateKey(credentialsId: 'deploy-ssh-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'DEPLOY_SSH_USER')]) {
          sh """
            chmod 600 "$SSH_KEY" || true
            # copy docker-compose.yml to remote host (assumes docker-compose.yml at repo root)
            scp -o StrictHostKeyChecking=no -i "$SSH_KEY" docker-compose.yml ${DEPLOY_SSH_USER}@${DEPLOY_HOST}:${DEPLOY_DIR}/docker-compose.yml

            # copy backend.env if you keep it in deploy/backend.env (optional)
            # scp -o StrictHostKeyChecking=no -i "$SSH_KEY" deploy/backend.env ${DEPLOY_SSH_USER}@${DEPLOY_HOST}:${DEPLOY_DIR}/backend.env || true

            # run docker compose pull && up on the host
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
