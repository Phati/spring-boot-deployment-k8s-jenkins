# 1. Set up Jenkins Server

We will run Jenkins in a docker container, we will also need docker running inside that docker container **(for building docker images from source code during containerize stage in pipeline)**,  we will install **kubectl**  and **helm** so that we can use those in jenkins pipelines.  We will create a custom Jenkins image from the Jenkins base Image and install docker during boot up.

Create a Dockerfile as below: 

    FROM jenkins/jenkins:lts  
    USER root  
      
    # Download and install docker  
    RUN apt-get update -qq \  
        && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common  
    RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -  
    RUN add-apt-repository \  
       "deb [arch=amd64] https://download.docker.com/linux/debian \  
     $(lsb_release -cs) \ stable"RUN apt-get update  -qq \  
        && apt-get -y install docker-ce  
    RUN usermod -aG docker jenkins  
      
    # Download and install helm  
    RUN apt-get install wget  
    RUN wget https://get.helm.sh/helm-v3.6.1-linux-amd64.tar.gz  
    RUN ls -a  
    RUN tar -xvzf helm-v3.6.1-linux-amd64.tar.gz  
    RUN cp linux-amd64/helm /usr/bin  
    RUN helm version  
      
    # Download and install kubectl  
    RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"  
    RUN install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    
    USER jenkins

Build Docker image
>docker image build -t custom-jenkins-docker:latest .

Run docker container from custom Jenkins image
>docker run -d -p 8080:8080 -p 50000:50000 --name jenkins -v //var/run/docker.sock:/var/run/docker.sock  -v jenkins_home:/var/jenkins_home custom-jenkins-docker:latest

It will print the admin password with below statement
>Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

Copy that password=>  you can get the password by checking the docker logs
>docker logs -f jenkins

Now open **localhost:8080** in browser to configure Jenkins, enter the password.
It will ask to install plugins, Install suggested plugins.
Next it will ask to create an admin user. Create one

Click save and continue 
 
 We will now install some Plugins in Jenkins, which we will need later in the deploy stage, where we will need to connect to Kubernetes cluster for executing kubectl/helm commands on our Kubernetes cluster

click on Manage Jenkins
click on plugins
click on Available plugins
search for kubernetes
select ***Kubernetes, Kubernetes Credentials, Kubernetes CLI***
click on install

**Add credentials**
Go to manage Jenkins click on credentials
1. add dockerhub credentials with id : registry-pass
2. add github creds with id : github-creds

Now our Jenkins Master is ready.


# 2. Create Source Code Repository

Now we will create a sample-spring-boot-app and push it on Github.
Additionally we will add below files to the project
1. .env
	
2. Dockerfile
	
3. Jenkinsfile

4.  Jenkins scripts

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

https://github.com/Phati/jenkins-shared-library
