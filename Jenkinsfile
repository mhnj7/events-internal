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

  }
}
