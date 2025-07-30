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

    stage('Install & Build & Test') {
      steps {
        script {
          docker.image('node:20').inside('-v /var/run/docker.sock:/var/run/docker.sock') {
            sh 'npm install'
            sh 'npm test'
            sh 'docker build -t $IMAGE_NAME:${BUILD_NUMBER} .'
          }
        }
      }
    }

    stage('Tag Repo') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'hub-https-creds',
                                          usernameVariable: 'USERNAME',
                                          passwordVariable: 'TOKEN')]) {
          bat 'git config user.email "jenkins@example.com"'
          bat 'git config user.name "Jenkins CI"'
          bat 'git remote set-url origin https://%USERNAME%:%TOKEN%@github.com/triafus/pipeline.git'
          bat 'git tag v%BUILD_NUMBER%'
          bat 'git push origin v%BUILD_NUMBER%'
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'hub-https-creds',
                                          usernameVariable: 'USERNAME',
                                          passwordVariable: 'TOKEN')]) {
          bat 'echo %TOKEN% | docker login ghcr.io -u %USERNAME% --password-stdin'
          bat 'docker push %IMAGE_NAME%:%BUILD_NUMBER%'
        }
      }
    }
  }

  triggers {
    githubPush()
  }
}
