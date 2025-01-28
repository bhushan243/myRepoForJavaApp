pipeline {
    agent any
    
    tools {
     maven 'Maven3'
   }
   
    
      stage ('Build') {
         steps {
          sh 'mvn clean install -f MyWebApp/pom.xml'
          }
      }
      
      stage ('Code Quality') {
      steps {
        withSonarQubeEnv('SonarQube') {
        sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
         }
       }
    

      }
      
      stage ('JaCoCo') {
      steps {
      jacoco()
      }
    }
    stage ('Nexus Upload') {
      steps {
      nexusArtifactUploader(
      nexusVersion: 'nexus3',
      protocol: 'http',
      nexusUrl: 'ec2-3-133-98-35.us-east-2.compute.amazonaws.com:8081/',
      groupId: 'com.dept.app',
      version: '1.0-SNAPSHOT',
      repository: 'maven-snapshots',
      credentialsId: '451b5637-7f5b-40f1-9879-922f7417f4fa',
      artifacts: [
       [artifactId: 'MyWebApp',
        classifier: '',
         file: 'MyWebApp/target/MyWebApp.war',
         type: 'war']
      ]
     )
       }
     }
     
    stage ('DEV Deploy') {
      steps {
      echo "deploying to DEV Env "
      deploy adapters: [tomcat9(credentialsId: '737e6cb3-3878-4b2f-8c07-762557620644', path: '', url: 'http://ec2-3-144-228-211.us-east-2.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
      }
    }
    stage ('Slack Notification') {
      steps {
        echo "deployed to DEV Env successfully"
        slackSend(channel:'devops-project', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }
    
    stage ('DEV Approve') {
      steps {
      echo "Taking approval from DEV Manager for QA Deployment"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to deploy?', submitter: 'admin'
        }
      }
    }
    
     stage ('QA Deploy') {
      steps {
        echo "deploying to QA Env "
        deploy adapters: [tomcat9(credentialsId: '737e6cb3-3878-4b2f-8c07-762557620644', path: '', url: 'http://ec2-3-144-228-211.us-east-2.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
        }
    }
    
    stage ('QA Approve') {
      steps {
        echo "Taking approval from QA manager"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
        }
      }
    }
    
    stage ('Slack Notification for QA Deploy') {
      steps {
        echo "deployed to QA Env successfully"
        slackSend(channel:'devops-project', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    } 
      
    
}
