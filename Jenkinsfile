node{
   try{
	stage('SCM Checkout'){
     git 'https://github.com/octopent/crudmvc_osr.git'
   }
   
   
   stage('SonarQube Server') {
        def mvnHome =  tool name: 'jenkins_maven', type: 'maven'
        withSonarQubeEnv('jenkins_sonar') { 
          sh "${mvnHome}/bin/mvn sonar:sonar"
        }
    }
    
    stage("Quality Gate Check"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
      }
      
      stage('Package Compilation'){
      def mvnHome =  tool name: 'jenkins_maven', type: 'maven'   
      sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean install"
      
   }
      
      stage('Artifact upload') {
    def server = Artifactory.server 'jenkins_artifactory'
   	def uploadSpec = """{
     "files": [
           {
              "pattern": "/var/lib/jenkins/workspace/TC-SQL__PIPELINE/target/*.war",
              "target": "mayank-snapshot"
           }
        ]
        }"""
    server.upload(uploadSpec)
	}
	
	stage('downloading artifact')
    {
        def server = Artifactory.server 'jenkins_artifactory'
        def downloadSpec="""{
        "files":[
        {
           "pattern":"mayank-snapshot/crudMVC.war",
            "target":"/var/lib/jenkins/warFiles/"
        }
        ]
        }"""
    server.download(downloadSpec)
    }
    
    stage ('Final deploy'){
        sh 'scp /var/lib/jenkins/warFiles/crudMVC.war minduser@deployment-vm.eastus.cloudapp.azure.com:/opt/apache-tomcat-8.5.37/webapps'
    }
    
	stage('Email Notification'){
		mail bcc: '', body: '''Hi!
		Your Build Passed!
		All Good!
		Mayank''', cc: '', from: '', replyTo: '', subject: 'PROJECT BUILD SUCCESS', to: 'rathore.mayanksgh@gmail.com'
	}
   }catch (err){
		mail bcc: '', body: '''Hi! 
		Oops! Build Failed!
		Mayank''', cc: '', from: '', replyTo: '', subject: 'PROJECT BUILD FAILED', to: 'rathore.mayanksgh@gmail.com'
	
		currentBuild.result = 'FAILURE'
   }
}
