pipeline {
  agent any

  environment {
    REGISTRY = 'ghcr.io/ton-compte/todo-app'
    GITHUB_TOKEN = credentials('github-pat')
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/ton-compte/ton-projet-todo.git'
      }
    }
    stage('Install & Test') {
      steps {
        sh 'npm install'
        sh 'npm test'
      }
    }
    stage('Docker Build') {
      steps {
        sh 'docker build -t $REGISTRY:${env.BUILD_NUMBER} .'
      }
    }
    stage('Docker Push') {
      steps {
        withCredentials([string(credentialsId: 'github-pat', variable: 'TOKEN')]) {
          sh "echo $TOKEN | docker login ghcr.io -u ton-compte --password-stdin"
          sh "docker push $REGISTRY:${env.BUILD_NUMBER}"
        }
      }
    }
    stage('Tag Repo') {
      steps {
        sh "git tag -a v${env.BUILD_NUMBER} -m 'Build ${env.BUILD_NUMBER}'"
        sh "git push origin v${env.BUILD_NUMBER}"
      }
    }
  }
}
