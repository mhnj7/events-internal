pipeline {
  agent any
  stages {
    stage('Clone') {
      steps {
        deleteDir()
        git(url: 'https://github.com/mhnj7/events-internal', branch: 'main')
      }
    }

    stage('Test') {
      steps {
        sh 'npm install'
        sh 'npm test'
      }
    }

    stage('Build') {
      steps {
        sh 'docker build -t mhnj7/events-internal:v1.0 .'
      }
    }

    stage('') {
      steps {
        sh 'docker push mhnj7/events-internal:v1.0 '
      }
    }

  }
  environment {
    dockerid = 'dockerhubcreds'
  }
}