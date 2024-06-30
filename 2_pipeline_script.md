#   Create Source Code Repository

Now we will create a sample-spring-boot-app and push it on Github.
Additionally we will add below files to root directory of the project

1. **.env**
	This file contains the env variables that will be used in jenkins, we will read this file in Jenkins pipeline and export those env variables
	
	    IMAGE=bill-pay-app
	    RELEASE_VERSION=5.0.1
	    COMPILE_ARGS=clean compile install -DskipTests
	    SKIP_TESTS=false
	    DOCKER_PORT=8080
	    JENKINS_JOB_NAME=bill-pay-app
	    TARGET_JAR_NAME_PREFIX=billpay

2. **Jenkinsfile**
		Here in the jenkins file we have used shared library, that we are going to create in the next steps. This shared library is a collection on groovy scripts that can be called as a function from this jenkins pipeline. You can see in each stage we have called those functions.
	
	    @Library('java_shared_library') _
	    
	    pipeline{
	    
	    agent any

	    environment {
	        DOCKER_HUB_PASSWORD = credentials('registry-pass')
	    }

	    stages{

	        stage('Pipeline Setup') {
	            steps {
	                sh "echo ****************PIPELINE SETUP STAGE****************"
	                loadEnv()
	            }
	        }

	        stage('Unit Tests'){
	            steps{
	                sh "echo ****************UNIT TESTING STAGE****************"
	                runJavaUnitTests()
	            }
	        }

	        stage('Build'){
	            steps{
	                sh "echo ****************BUILD STAGE****************"
	                runBuildJarFile()
	            }
	        }

	        stage('Containerize'){
	            steps{
	                sh "echo ****************CONTAINERIZE STAGE****************"
	                runDockerBuild()
	            }
	        }

	        stage('Push'){
	            steps{
	                sh "echo ****************PUSH STAGE****************"
	                runDockerPush()
	            }
	        }

	        stage('Deploy'){
	            steps{
	                sh 'echo ****************DEPLOY STAGE****************'
	                withKubeConfig(caCertificate: '', clusterName: 'test-cluster-jenkins', contextName: '', credentialsId: 'kubernetes-scret', namespace: 'cs', restrictKubeConfigAccess: false, serverUrl: 'https://34.44.63.28') {
	                    runDeployDockerImage()
	                }
	            }
	        }

	    }
	    }

3.  jenkins folder
	This folder contains bash script files that will be executed from the groovy scripts.
	These scripts are further stored in folders specific to the stage in pipeline.
	1. In /jenkins/build folder in the mvn.sh script we have `docker run --rm  -v $WORKSPACE:/app -v /root/.m2/:/root/.m2/ -w /app  maven:3.8.3-openjdk-17 "$@"` here we run a maven docker container and map jenkins app workspace dir to the working dir in container and we can pass mvn commands as argument to the script.
		This script is called from  **runBuildJarFile.groovy**  function from shared library, and the function is called from the build stage in pipeline. Since we have mapped the jenkins worksapce directory to the container when the container builds the jar file we will have the jar file in target folder of the jenkins workspace directory of this project.

	2. In /jenkins/build folder we have Dockerfile-Java and docker-compose-build.yaml file those will be used to build the docker image. you can see the docker-compose-build file contains key `image: $IMAGE:$RELEASE_VERSION"` these env variables are from the .env file.
	3. In the build.sh script file we copy the jar file from the target folder and then we call docker compose build

	Similary we have other scripts add those as well,(This scripts will work for java spring boot project)
	https://github.com/Phati/billpay/tree/dev/jenkins





**We will create another git repository for our Jenkins shared library code**

This shared library has groovy scripts which are basically functions that can be called from a Jenkins pipeline that imports this shared library. This is a good way to reuse code in different pipeline scripts.

> vars/loadEnv.groovy

This function is called in the jenkins pipeline in the begining and it reads the .env file and splits it by new line and creates the environment variables in Jenkins to be used later during the pipeline execution

    def call(){
    echo "loading .env file"
    def envFilePath = "${WORKSPACE}/.env"
    if (fileExists(envFilePath)) {
        def envContent = readFile(envFilePath).trim()
        envContent.readLines().each { line ->
        def (key, value) = line.split('=')
        env."${key}" = value
    }
    } else {
        echo "No .env file found"
    }
	}

check the github repository for other shared library code
https://github.com/Phati/jenkins-shared-library
