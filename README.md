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
  
  Run the below command to unzip package
  ```
  apt install unzip
  ```
  
  To add sonarqube user run
  ```
  adduser sonarqube
  ```
  
  Now, login as sonarqube user
  ```
  sudo su - sonarqube
  ```
  
 To download sonarqube binary
 ```
 wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
 unzip *
 ```
 Give required permission to sonarqube
 ```
  chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
  chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
 ```
 Now Start your sonarqube server
 ```
  cd sonarqube-9.4.0.54424/bin/linux-x86-64/
  ./sonar.sh start
 ``` 
   Now you can access the SonarQube Server on http://your-ip-address:9000
  
  Now we will configure some credential in jenkins
  ## Generate Github Token
    - Go to www.github.com
    - Login to your account
    - Go to setting
    - Select developer Setting
    - Go to Personal Access Token > Tokens(classic)
    - Generate new Token > Generate new Token(classic)
    - Enter you github password
    - Give any note to it
    - Tick repo
    - Generate Token
    
  ## Configure Github Token in Jenkins
    - Go to jenkins Dashboard
    - Manage jenkins
    - Go to Credentials under security section
    - Under Stores scoped to Jenkins select system
    - Select global Credentials
    - Add Credentials
    - Under Kind Select Secret Text
    - Enter git token under secret
    - Give id name 
    - Create
  
  ## Configure DockerHub Crential in Jenkins
     - Go to jenkins Dashboard
    - Manage jenkins
    - Go to Credentials under security section
    - Under Stores scoped to Jenkins select system
    - Select global Credentials
    - Add Credentials
    - Under Kind Username and Password
    - Enter Username and Password
    - Give id name 
    - Create
  ## Similarly as above you can configure Credentials for SonarQube as well

  # Now will do Deployment 
  There are some pre-requisite before deployment which are as follows
   ## Install aws cli
    ```
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
    ```
  ## Install Kubectl 
  - To download latest release of Kubectl 
    ```
     curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    ```
  - Download kubectl checksum file
    ```
     curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
    ```
  - Validate the kubectl binary against the checksum file:
    ```
    echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
    ```
  - To install kubectl
    ```
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    ```
  - Test to ensure the version you installed is up-to-date:
    ```
    kubectl version --client
    ```
  ## Configure AWS cli
  To configure Aws cli
  ```
  aws configure
  ```
  Enter your **Access key** and **Secret key** and **Region**
  
  ## Create EKS
  
  ## Update config file of kubernetes
  ```
  aws eks update-kubeconfig --region region-code --name my-cluster
  ```
  
  ## Install Argocd
  Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.
  ```
  curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.24.0/install.sh | bash -s v0.24.0
  ```
  Install the operator by running the following command:
  ```
  kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
  ```
  After install, watch your operator come up using next command.
  ```
  kubectl get csv -n operators
  ```
  The following example shows the most minimal valid manifest to create a new Argo CD cluster with the default configuration.
  ```
  apiVersion: argoproj.io/v1alpha1
  kind: ArgoCD
  metadata:
    name: example-argocd
    labels:
      example: basic
  spec: {}
  ```
  Save the about content in argocd-basic.yaml
  Now, Create Argo CD cluster
  ```
  kubectl create  -f argocd-basic.yaml
  ```
  ## Create app in Argocd
  - click on Create App
  - Give Application Name
  - Select project name as dfault
  - Sync Policy as Automatic
  - Under source provide your git repo URL
  - Under path give path of Manifest directory
  - Give namespace as default
Your Deployment has been done now 
## Now to access the application 
  - Go to your ec2 machine
  - Run following command
  ```
  kubectl get svc 
  ```
  - Copy the External IP and paste in browser. 

# Thanks :)
 
  
  
