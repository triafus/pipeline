pipeline {
  agent any

  environment {
    REPO_URL = 'https://github.com/triafus/pipeline.git'
    IMAGE_NAME = 'ghcr.io/triafus/todo-app'
  }

  stages {
    stage('Checkout') {
      steps {
        git url: "${REPO_URL}", branch: 'main', credentialsId: 'hub-https-creds'
      }
    }

    stage('Install & Build') {
      steps {
        sh 'docker run --rm -v $PWD:/app -w /app node:20-alpine npm install'
      }
    }

    stage('Tests') {
      steps {
        sh 'docker run --rm -v $PWD:/app -w /app node:20-alpine npm test'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t $IMAGE_NAME:${BUILD_NUMBER} .'
      }
    }

    stage('Tag Repo') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'hub-https-creds',
                                          usernameVariable: 'USERNAME',
                                          passwordVariable: 'TOKEN')]) {
          sh 'git config user.email "jenkins@example.com"'
          sh 'git config user.name "Jenkins CI"'
          sh 'git remote set-url origin https://git:$TOKEN@github.com/triafus/pipeline.git'
          sh 'git tag v${BUILD_NUMBER}'
          sh 'git push origin v${BUILD_NUMBER}'
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'hub-https-creds',
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
