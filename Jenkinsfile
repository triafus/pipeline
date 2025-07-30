pipeline {
  agent any

  environment {
    REPO_URL = 'https://github.com/triafus/pipeline.git'
    IMAGE_NAME = 'ghcr.io/triafus/todo-app'
  }

  stages {
    stage('Checkout') {
      steps {
        bat 'git clone %REPO_URL% --branch main --single-branch .'
      }
    }

    stage('Install & Build') {
      steps {
        bat 'npm install'
      }
    }

    stage('Tests') {
      steps {
        bat 'npm test'
      }
    }

    stage('Docker Build') {
      steps {
        bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
      }
    }

    stage('Tag Repo') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'hub-https-creds',
                                          usernameVariable: 'USERNAME',
                                          passwordVariable: 'TOKEN')]) {
          bat '''
            git config user.email "jenkins@example.com"
            git config user.name "Jenkins CI"
            git remote set-url origin https://%USERNAME%:%TOKEN%@github.com/triafus/pipeline.git
            git tag v%BUILD_NUMBER%
            git push origin v%BUILD_NUMBER%
          '''
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'hub-https-creds',
                                          usernameVariable: 'USERNAME',
                                          passwordVariable: 'TOKEN')]) {
          bat '''
            echo %TOKEN% | docker login ghcr.io -u %USERNAME% --password-stdin
            docker push %IMAGE_NAME%:%BUILD_NUMBER%
          '''
        }
      }
    }
  }

  triggers {
    githubPush()
  }
}
