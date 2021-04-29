node {
   def commit_id
   def image = 'mhnj7/events-internal'
   def image1
   def image2
   env.JENKINS_REPORTER_ENABLE_SONAR=true

   stage('clone') {
       deleteDir()
       sh "git clone 'https://github.com/mhnj7/events-internal' ."
       sh "git rev-parse --short HEAD > .git/commit-id"                        
       commit_id = readFile('.git/commit-id').trim()
   }
   
   try {
      stage('test') {
         sh 'npm install'
         sh 'npm test'
      }
   } catch (e) {
      throw e
   } finally {
      junit 'report.xml'
   }
   
   stage('sonar-scanner') {
      def sonarqubeScannerHome = tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
      withSonarQubeEnv() {
         sh "${sonarqubeScannerHome}/bin/sonar-scanner -Dsonar.projectKey=events-internal"
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
        image1 = image + ":${BUILD_NUMBER}.${commit_id}"
        image2 = image + ":latest"
        def app1 = docker.build(image1, '.').push()
        def app2 = docker.build(image2, '.').push()
     }
   }
   
   stage('deploy') {
      sh "gcloud container clusters get-credentials devops-demo-cluster --zone us-east4-c --project events-demo-308800"
      try {
         sh "kubectl delete -f kubernetes"
      } catch (e) {
         // do nothing
      } finally {
         sh "kubectl apply -f kubernetes"
      }
   }
   
   stage('cleanup') {
      sh "docker rmi ${image1} ${image2}"
   }
   
}
