pipeline {
  agent any

  environment {
    REPO_URL = 'https://github.com/triafus/pipeline.git'
    IMAGE_NAME = 'ghcr.io/triafus/todo-app'
  }

  stages {
    stage('Checkout') {
      steps {
        git url: "${REPO_URL}", branch: 'main', credentialsId: 'github-https-creds'
      }
    }

    stage('Install & Build') {
      steps {
        sh 'npm install'
      }
    }

    stage('Tests') {
      steps {
        sh 'npm test'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t $IMAGE_NAME:${BUILD_NUMBER} .'
      }
    }

    stage('Tag Repo') {
      steps {
        sh 'git config user.email "jenkins@example.com"'
        sh 'git config user.name "Jenkins CI"'
        sh 'git tag v${BUILD_NUMBER}'
        sh 'git push origin v${BUILD_NUMBER}'
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'github-container-registry',
                                          usernameVariable: 'USERNAME',
                                          passwordVariable: 'TOKEN')]) {
          sh 'echo $TOKEN | docker login ghcr.io -u $USERNAME --password-stdin'
          sh 'docker push $IMAGE_NAME:${BUILD_NUMBER}'
        }
      }
    }
  }

  triggers {
    githubPush()
  }
}
