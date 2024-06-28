# 1. Set up Jenkins Server

We will run Jenkins in a docker container, we will also need docker running inside that docker container **(for building docker images from source code during containerize stage in pipeline)**,  we will install kubectl  and so we will create a custom Jenkins image from the Jenkins base Image and install docker during boot up.

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

Build Docker image
>docker image build -t custom-jenkins-docker .

Run docker container of custom Jenkins image
>docker run -d -p 8081:8080 -p 50000:50000  -v jenkins_home_1:/var/jenkins_home custom-jenkins-docker

It will print the admin password with below statement
>Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

Copy that password, you can also get the password by checking the docker logs
>docker logs <container_id>

Now open **localhost:8081** in browser to configure Jenkins, enter the password.
It will ask to install plugins, Install suggested plugins.
Next it will ask to create an admin user. Create one
 
 We will now install some Plugins in Jenkins, which we will need later in the deploy stage, where we will need to connect to Kubernetes cluster for executing kubectl/helm commands on our Kubernetes cluster

click on Manage Jenkins
