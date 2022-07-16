pipeline {
	agent any
   
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven-3.8"
	
	// jdk "JDK"
    }
    environment {
        POM_VERSION = getVersion()
        JAR_NAME = getJarName()
        AWS_ECR_REGION = 'us-east-1'
        AWS_ECS_CLUSTER = 'DemoCluster'
        AWS_ECS_SERVICE = 'Demo-api-service'
        AWS_ECS_TASK_DEFINITION = 'Demo-api-taskdefinition'
        AWS_ECS_COMPATIBILITY = 'FARGATE'
        AWS_ECS_NETWORK_MODE = 'awsvpc'
        AWS_ECS_CPU = '256'
        AWS_ECS_MEMORY = '512'
        AWS_ECS_EXECUTION_ROLE = 'arn:aws:iam::400385795902:role/AmazonECSTaskExecutionRolePolicy' 
        AWS_ECS_TASK_DEFINITION_PATH = './ecs/container-definition.json'
    }
    stages {

        
        stage('Pulling The Code From Git To Jenkins Server') {
            steps{
               git branch: 'main', credentialsId: 'Github', url: 'https://github.com/kprasanth999/ECS-scripts.git'
	        }
	    }	
	 
    }
    
}
def getJarName() {
    def jarName = getName() + '-' + getVersion() + '.jar'
    echo "jarName: ${jarName}"
    return  jarName
}

def getVersion() {
    def pom = readMavenPom file: './pom.xml'
    return pom.version
}

def getName() {
    def pom = readMavenPom file: './pom.xml'
    return pom.name
}