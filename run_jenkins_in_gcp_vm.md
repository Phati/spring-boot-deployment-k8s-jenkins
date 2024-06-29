
# 1. Install Jenkins on a VM in GCP

1. Provision a Ubuntu VM on Google Cloud Compute Engine
	Login to GCP console and navigate to **Compute Engine-> VM instances**
	Click on **Create instance**
	![enter image description here](https://i.postimg.cc/Mps6SmSH/Screenshot-2024-06-29-144633.png)
	
	Select Ubuntu as OS
	![enter image description here](https://i.postimg.cc/RFt4wrnJ/Screenshot-2024-06-29-144600.png)
2. Make sure to expose port 8080 and allow http, https traffic
![enter image description here](https://i.postimg.cc/QMHXxK8b/Screenshot-2024-06-29-144651.png)

4. SSH into the VM and install below applications

 Install Jenkins
 https://www.cherryservers.com/blog/how-to-install-jenkins-on-ubuntu-22-04
 Install Docker
 https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04
 Install DockerCompose
 https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04
 Install helm
 https://helm.sh/docs/intro/install/
 Install kubectl
 https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
 
***Login to jenkins UI from browser***
Copy the admin password from logs
>sudo systemctl status jenkins

Copy that password.

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
1. add dockerhub credentials as (secret text) with id : registry-pass
2. add github creds as (usename and password) with id : github-creds

Now our Jenkins Master is ready.
