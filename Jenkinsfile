
pipeline {
	agent any
   
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven-3.8"
	
	// jdk "JDK"
    }

    environment {
        POM_VERSION = getVersion()
        JAR_NAME = getWarName()
        AWS_ECR_REGION = 'us-east-1'
        AWS_ECS_CLUSTER = 'DemoCluster'
        AWS_ECS_SERVICE = 'Demo-api-service'
        AWS_ECS_TASK_DEFINITION = 'Demo-api-taskdefinition'
        AWS_ECS_COMPATIBILITY = 'FARGATE'
        AWS_ECS_NETWORK_MODE = 'awsvpc'
        AWS_ECS_CPU = '256'
        AWS_ECS_MEMORY = '512'
        AWS_ECS_EXECUTION_ROLE = 'arn:aws:iam::400385795902:role/AmazonECSTaskExecutionRolePolicy' 
        AWS_ECS_TASK_DEFINITION_PATH = './container-definition.json'
        AWS_ECR_URL = '400385795902.dkr.ecr.us-east-1.amazonaws.com/ecr_testing_repo'
    }

    stages {

        
        stage('Pulling The Code From Git To Jenkins Server') {
            steps{
               git branch: 'main', credentialsId: 'GithubToken', url: 'https://github.com/kprasanth999/ECS-scripts.git'
	        }
	    }	
	 
	    
	    stage('Compiling the Code With Maven-3.8') {
	        steps{
	            echo "COMPILING THE CODE"
                script{
		            sh "mvn clean compile" 
		        }   
	        }
	    }			
        
       
	    
        stage('SonarQube Connection with Jenkins') {
            steps {
                withSonarQubeEnv('sonar-7') {
                    sh "mvn sonar:sonar \
                    -Dsonar.host.url=http://34.204.51.2:9000 \
                    -Dsonar.login=da2c37151854a8de06fe5cb14d6dd186a6ab40d3"
                }
            }
        }

        stage('Pulling the SonarQube Code Quality Status') {
            steps {
                timestamps {
                    script {
                        try {
                       		def sonar_api_token='da2c37151854a8de06fe5cb14d6dd186a6ab40d3';
                        	def sonar_project='com.example:java-maven';
                        	sh """#!/bin/bash +x
				            sleep 30
                        	echo "Checking status of SonarQube Project = ${sonar_project}"
                        	sonar_status=`curl -s -u ${sonar_api_token}: http://34.204.51.2:9000/api/qualitygates/project_status?projectKey=${sonar_project} | grep '{' | python -c 'import json,sys;obj=json.load(sys.stdin);print obj["'projectStatus'"]["'status'"];'`
                        	echo "SonarQube status = \$sonar_status"
                        	case \$sonar_status in
                                "ERROR")
                                echo "Quality Gate Failed - Major Issues > 0"
                                echo "Check the SonarQube Project ${sonar_project} for further details."
                                exit 1
                                ;;
                                "OK")
                                echo "Quality Gate Passed"
                                echo "Check the SonarQube Project ${sonar_project} for further details."
                                exit 0
                                ;;
                                esac
                                """

                                echo 'Code Quality Checks Complete.'
                                  //mark the pipeline as unstable and continue
                        }
			        
			            catch(e){
                            currentBuild.result = 'ABORTED'
                            result = "FAIL"
				            mail bcc: '', body: '''SonarQube Quality Gate failed''', 
			                cc: '', from: '', replyTo: '', subject: 'This Notification is to the Developers Team', to: 'kpvkpv67@gmail.com'
				        throw e
			            }
			         
                    }
                }
            }
        }		
        


        stage('Compile,Test & Package') {
	        steps{
		        script {
			
		            sh "mvn clean package" 
                    sh "env"
				                   
                }
	        }	
        }   
	
	    
	    stage('Nexus Artifactory Upload'){
	        steps {
             	nexusArtifactUploader artifacts: [[artifactId: 'java-maven', classifier: '', 
				    file: 'target/java-maven-${Version}.war', type: 'war']], 
			            credentialsId: 'Nexus-pw', 
			            groupId: 'com.example', 
			            nexusUrl: '34.204.51.2:8080/nexus', 
			            nexusVersion: 'nexus2', 
			            protocol: 'http', 
			            repository: 'releases/', 
				        version: '${Version}'
	            }
	    }      
	    
	    stage('Build Docker Image For Testing') {
            steps{
                sh "docker build -t ecr_testing_repo ."  
            }
        }
	    

        stage('Tagging the Docker Image with ECR Testing Repository Name') {
            steps{
                sh "docker tag ecr_testing_repo:latest 400385795902.dkr.ecr.us-east-1.amazonaws.com/ecr_testing_repo:latest"  
            }
        }
    
    
        stage('Uploading The Image into ECR in Testing Repository') {
            steps{
                script{
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 400385795902.dkr.ecr.us-east-1.amazonaws.com'
                    sh 'docker push 400385795902.dkr.ecr.us-east-1.amazonaws.com/ecr_testing_repo:latest'
                }
            }
        }
      
        stage('Notify UAT-Team through a Mail') {
	        steps {
                mail bcc: '', body: '''Please Pull the Image From ECR With this name for Testing
                "400385795902.dkr.ecr.us-east-1.amazonaws.com/ecr_testing_repo:latest"
		        Approval Required to Deploy the API into Production Repository''',
		        cc: '', from: '', replyTo: '', subject: 'This Notification is to the API-Testing Team', to: 'kpvkpv67@gmail.com'
            }
	    }	

	    
	    stage('approve') {
	        steps {
	            echo "Approval State"
                timeout(time: 7, unit: 'DAYS') {                    
			    input message: 'Do you want to deploy?', submitter: 'Prasanth'
		        }
	        }
        }

        
	    
	    
	    
	    
        stage('Email Notification for Successful Delivery of the API Container into ECR Registry') {
            steps{
                                 
		        mail bcc: '', body: ''' Container Registered in the Production Repository. Successfully Completed the CI-CD Pipeline. ''',
                cc: '', from: '', replyTo: '', subject: 'To the Developers-Team: Jenkins Pipeline Succeeded on the New Commit', to: 'kpvkpv67@gmail.com'
		        mail bcc: '', body: ''' Container Registered in the Production Repository. Successfully Completed the CI-CD Pipeline. ''',
                cc: '', from: '', replyTo: '', subject: 'To the UAT-Team: Jenkins Pipeline Succeeded on the New Commit', to: 'kpvkpv67@gmail.com'
	    }
        }
    

        stage('Deploy in ECS') {
            steps {
                //withCredentials([string(credentialsId: 'AWS_EXECUTION_ROL_SECRET', variable: 'AWS_ECS_EXECUTION_ROL'),string(credentialsId: 'AWS_REPOSITORY_URL_SECRET', variable: 'AWS_ECR_URL')]) {
                    script {
                        updateContainerDefinitionJsonWithImageVersion()
                        sh("/usr/local/bin/aws ecs register-task-definition --region ${AWS_ECR_REGION} --family ${AWS_ECS_TASK_DEFINITION} --execution-role-arn ${AWS_ECS_EXECUTION_ROLE} --requires-compatibilities ${AWS_ECS_COMPATIBILITY} --network-mode ${AWS_ECS_NETWORK_MODE} --cpu ${AWS_ECS_CPU} --memory ${AWS_ECS_MEMORY} --container-definitions file://${AWS_ECS_TASK_DEFINITION_PATH}")
                        def taskRevision = sh(script: "/usr/local/bin/aws ecs describe-task-definition --task-definition ${AWS_ECS_TASK_DEFINITION} | egrep \"revision\" | tr \"/\" \" \" | awk '{print \$2}' | sed 's/\"\$//'", returnStdout: true)
                        sh("/usr/local/bin/aws ecs update-service --cluster ${AWS_ECS_CLUSTER} --service ${AWS_ECS_SERVICE} --task-definition ${AWS_ECS_TASK_DEFINITION}:${taskRevision}")
                    }
                
            }
        }
    }
} 
def getWarName() {
    def warName = getName() + '-' + getVersion() + '.war'
    echo "warName: ${warName}"
    return  warName
}

def getVersion() {
    def pom = readMavenPom file: './pom.xml'
    return pom.version
}

def getName() {
    def pom = readMavenPom file: './pom.xml'
    return pom.name
}

def updateContainerDefinitionJsonWithImageVersion() {
    def containerDefinitionJson = readJSON file: AWS_ECS_TASK_DEFINITION_PATH, returnPojo: true
    containerDefinitionJson[0]['image'] = "${AWS_ECR_URL}:latest".inspect()
    echo "task definiton json: ${containerDefinitionJson}"
    writeJSON file: AWS_ECS_TASK_DEFINITION_PATH, json: containerDefinitionJson
}  
