pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    stages {
        stage('Build') {
            steps {
                echo "Building the project"
                sh 'mvn clean install -f MyWebApp/pom.xml'
            }
        }

        stage('Code Quality') {
            steps {
                echo "Performing code quality check with SonarQube"
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
                }
            }
        }

        stage('JaCoCo') {
            steps {
                echo "Running JaCoCo for code coverage"
                jacoco()
            }
        }

        stage('Nexus Upload') {
            steps {
                echo "Uploading artifact to Nexus repository"
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'http://ec2-3-133-98-35.us-east-2.compute.amazonaws.com:8081/',
                    groupId: 'com.dept.app',
                    version: '1.0-SNAPSHOT',
                    repository: 'maven-snapshots',
                    credentialsId: '451b5637-7f5b-40f1-9879-922f7417f4fa',
                    artifacts: [
                        [
                            artifactId: 'MyWebApp',
                            classifier: '',
                            file: 'MyWebApp/target/MyWebApp.war',
                            type: 'war'
                        ]
                    ]
                )
            }
        }

        stage('DEV Deploy') {
            steps {
                echo "Deploying to DEV Environment"
                deploy adapters: [tomcat9(credentialsId: '737e6cb3-3878-4b2f-8c07-762557620644', path: '', url: 'http://ec2-3-144-228-211.us-east-2.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }

        stage('Slack Notification') {
            steps {
                echo "Sending Slack notification for DEV deployment"
                slackSend(channel: 'devops-project', message: "DEV deployment successful: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }

        stage('DEV Approve') {
            steps {
                echo "Requesting approval from DEV Manager for QA deployment"
                timeout(time: 7, unit: 'DAYS') {
                    input message: 'Do you want to deploy to QA?', submitter: 'admin'
                }
            }
        }

        stage('QA Deploy') {
            steps {
                echo "Deploying to QA Environment"
                deploy adapters: [tomcat9(credentialsId: '737e6cb3-3878-4b2f-8c07-762557620644', path: '', url: 'http://ec2-3-144-228-211.us-east-2.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }

        stage('QA Approve') {
            steps {
                echo "Requesting approval from QA Manager for production deployment"
                timeout(time: 7, unit: 'DAYS') {
                    input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
                }
            }
        }

        stage('Slack Notification for QA Deploy') {
            steps {
                echo "Sending Slack notification for QA deployment"
                slackSend(channel: 'devops-project', message: "QA deployment successful: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }
    }
}
