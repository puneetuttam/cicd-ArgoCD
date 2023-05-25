# Jenkins CI/CD pipeline to deploy Spring Boot java based web application on EKS using ArgoCD

This pipeline will have multiple stages which are as follows

- Git Checkout
- Maven Build
- Code Analysis Tool for code smells(SonarQube)
- Docker Image Build
- Docker Image Push to Dockerhub
- Updating image in Deployment manifest file

Apart from all this we will have OLM(operator Life Manager) to manage Argocd Opertor that we will install in our Ec2 Machine

Also, we are gonna create EKS in which we will deploy our application


## Create a EC2 instance 
- Go to AWS Console
- Go to EC2 instance
- Go to Launch Instance
- Give any name to your EC2 machine
- Select your AMI ( we are using Ubuntu)
- Select instance type as t2.medium or higher
- You can select defaul Networking configuration
- Now, Hit Launch Instance


## Connect to your Ec2 Instance.
- Go to AWS Console
- Instances(running)
- Select your instances
- Connect

## Install Jenkins.
Now, we will install jenkins in our machine but for it we need java as Pre-Requisites

- Java (JDK)
Run the below commands to install Java.
Install Java

```
sudo apt update
sudo apt install openjdk-11-jre
```

Verify Java is Installed
```
java -version
```
Now, you can proceed with installing Jenkins
```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

**NOTE:** Now, Jenkins runs on port 8080 but it is not accessible to outer world so open port for 8080 in security group.


## Modify Security Group
- Go to AWS console
- EC2 > Instances > Click on
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules (you can just allow TCP 8080, in my case, I allowed All traffic).


## Login to Jenkins using the below URL:
http://<ip-address>:8080 [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

After you login to Jenkins Run the command to copy the Jenkins Admin Password 
``` 
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Enter the Administrator password in jenkins
  

- Click on install Suggested Plugin
- You can create a user or you can login as admin
  
## Installation of plugins in Jeknins
- Go to Jenkins Dashboard
- Manage Jenkins
- Plugins
- Select Available plugins
- Install plugins **Docker Pipeline, Sonarqube Scanner**.
  
## Installation of Docker in our machine
  Run the below command to intall docker
  ```
  sudo apt update
  sudo apt install docker.io
  ```
## Grant Jenkins user and Ubuntu user permission to docker deamon.
  Run the below command to provide permission
  ```
  sudo su - 
  usermod -aG docker jenkins
  usermod -aG docker ubuntu
  systemctl restart docker
  ```
## Installing and Configuring Sonar Server.
  
  
  
