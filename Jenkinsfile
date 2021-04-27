node {
   def commit_id
   def image = 'mhnj7/events-internal'

   stage('clone') {
       deleteDir()
       sh "git clone 'https://github.com/mhnj7/events-internal' ."
       sh "git rev-parse --short HEAD > .git/commit-id"                        
       commit_id = readFile('.git/commit-id').trim()
   }
   
   try {
      stage('test') {
         nodejs(nodeJSInstallationName: 'nodejs') {
            sh 'npm install'
            sh 'npm test'
         }
      }
   } catch (e) {
      throw e
   } finally {
      junit 'report.xml'
   }
   
   stage('sonar-scanner') {
       def sonarqubeScannerHome = tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
       withSonarQubeEnv() {
           sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=events-internal"
       }
    }
   
   stage('quality gate'){
      timeout(5) { // Just in case something goes wrong, pipeline will be killed after a timeout
         def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
         if (qg.status != 'OK') {
            error "Pipeline aborted due to quality gate failure: ${qg.status}"
         }
      }
   }
   
   stage('docker build/push') {
     docker.withRegistry('https://index.docker.io/v1/', 'dockerhubid') {
       def app1 = docker.build(image + ":${BUILD_NUMBER}.${commit_id}", '.').push()
       def app2 = docker.build(image + ":latest", '.').push()
     }
   }
   
}
