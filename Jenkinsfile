pipeline {
    agent any
    
    tools { 
        maven 'maven' 
        jdk 'jdk1.8' 
    } 
    
   stages {    
        
      stage('SCM') {
         steps {
            git 'https://github.com/raghavendradurgampudi/spring-petclinic.git'
         }
      } 
        
      stage('Build Project') {
         steps {
            sh '''
            echo 'cd spring-petclinic'
            mvn clean install
            '''
         }
      }
        
      stage('SonarQube') { 
         steps { 
            sh ''' 
            echo "SonarQube"
            '''   	
         }    
      } 
        
    stage('Docker Image'){
        steps{
            sh '''
            cd sm-petclinic
            sudo docker build -t raghavendradurgampudi/spring-petclinic .
             '''
           }
    }
    stage('Push Images to repository'){
        steps{
            withCredentials([string(credentialsId: 'Docker_repository', variable: 'dockerPassword')]) {
                sh "sudo docker login -u raghavendradurgampudi -p${dockerPassword}"
            }
            sh 'echo "Push Image to repository" '
            sh 'sudo docker push raghavendradurgampudi/spring-petclinic'
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
        stage('AWS Connection and Deployment'){
            steps{
                   sh 'AWS Connection and Deployment'
                   sshagent(['AWS_Deployment_Server']) {
                   sh "ssh -o StrictHostKeyChecking=no ubuntu@54.84.13.62 sudo docker-compose up -d"
                 }
            }
        }
        
   } 
   post {
        always {
            //archiveArtifacts artifacts: 'generatedFile.log', onlyIfSuccessful: true
          
            emailext attachLog: true,
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
                recipientProviders: [developers(), requestor()],
                subject: "Jenkins Build :- ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            
        }
    }
}
