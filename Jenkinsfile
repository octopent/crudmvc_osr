node{
   try{
	stage('SCM Checkout'){
     git 'https://github.com/octopent/crudmvc_osr.git'
   }
   
   //abc
   
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
    
    stage('JIRA ENVIRONMENT'){
    
          if(currentBuild.previousBuild.result.toString() == 'SUCCESS'){
              
              //SENDING EMAIL
              stage('EMAIL NOTIFICATION'){
                    
                    mail bcc: '', body: '''Hi!
        		    Your Build Passed!
        		    All Good!
        		    Mayank''', cc: '', from: '', replyTo: '', subject: 'BUILD SUCCESS', to: 'rathore.mayanksgh@gmail.com'
              }
          }
          
          else{
                //MAKING A TRANSITION 
                stage('JIRA ISSUE TRANSITION'){
                          withEnv(['JIRA_SITE=jenkins_jirasteps']) {
                      def transitionInput = 
                      [
                          transition: [
                              id: '21'
                          ]
                      ]
                      jiraTransitionIssue idOrKey: 'MAYAN-1', input: transitionInput
                    }
                }
          }
       }
   }catch (err){
	    if(currentBuild.previousBuild.result.toString() == 'SUCCESS'){
            
	    //CREATING A NEW JIRA ISSUE
            stage('CREATTING NEW JIRA ISSUE'){

                withEnv(['JIRA_SITE=jenkins_jirasteps']) {
                def testIssue = [fields: [ project: [id: '10001'],
                                      summary: 'BUILD FAILED' ,
                                      description: 'BUILD FAILED FOR BUILD ' + env.BUILD_ID,
                                      issuetype: [name: 'Bug']]]
    
                response = jiraNewIssue issue: testIssue
                
                jiraAssignIssue idOrKey: response.data.key, userName: 'adityajain3896'

                echo response.successful.toString()
                echo response.data.toString()
                
                currentBuild.result = 'FAILURE'
                }
            }
        }
        else{
            //ADDING A JIRA COMMENT
            stage('ADDING A JIRA COMMENT'){
                withEnv(['JIRA_SITE=jenkins_jirasteps']) {
                    jiraAddComment idOrKey: 'MAYAN-4', comment: 'Some error!'
                }
            }
        }
   }
}
