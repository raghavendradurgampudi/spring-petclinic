pipeline {
 agent any
   stage("Build") {
     steps { 
        bat ''' 
    	    mvn install -DskipTests=true
	        // mvn clean install
           '''
         }
      }



   stage('Approval') {
            // no agent, so executors are not used up when waiting for approvals
            agent none
            steps {
                script {
                    def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'rkivisto,admin', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                    sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
                }
            }
        }
	
    stage('Build images') {
	  steps {
		bat '''
		  docker build -f "Dockerfile" -t raghavendradurgampudi/spring-petclinic:latest.
		'''
	      }
       }
   
     stage('Push Docker image') {
	  steps{
		    withDockerRegistry([ credentialsId: "Docker_Hub", url: "" ]){
			bat "docker push raghavendradurgampudi/spring-petclinic:latest"   
	  	   }
	   }
       } 
       stage('Deploy On Aws'){
	       steps{
			sshagent(['dev-server']) {
			bat "ssh -o StrictHostKeyChecking=no ec2-user@3.85.1.232 sudo dokcer run raghavendradurgampudi/spring-petclinic:latest"
			}
	   	}
   } 
	post {
        always {          
            emailext attachLog: true,
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
                recipientProviders: [developers(), requestor()],
                subject: "Jenkins Build :- ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            
        }
    }
}
