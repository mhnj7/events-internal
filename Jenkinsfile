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
        jobDsl(useScriptText: true, scriptText: 'job(\'Test\') {   publishers {     findText {       textFinders {         textFinder {           regexp \'5 passing\'           changeCondition \'MATCH_FOUND\'           alsoCheckConsoleOutput true           buildResult \'ABORTED\'         }       }     }   } }', sandbox: true)
      }
    }

  }
}