# Spring Boot based Java web application
 
This is a simple Sprint Boot based Java application that can be built using Maven. Sprint Boot dependencies are handled using the pom.xml 
at the root directory of the repository.

This is a MVC architecture based application where controller returns a page with title and message attributes to the view.

## Installation on EC2 Instance
Install Jenkins, configure Docker as agent.

## AWS EC2 Instance
Go to AWS Console
Instances(running)
Launch instances

## Install Jenkins.
Pre-Requisites:

Java (JDK)
Run the below commands to install Java and Jenkins
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
Now, Jenkins runs on port 8080 but it is not accessible to outer world so open port for 8080 in security group.



EC2 > Instances > Click on
In the bottom tabs -> Click on Security
Security groups
Add inbound traffic rules (you can just allow TCP 8080, in my case, I allowed All traffic).


## Login to Jenkins using the below URL:
http://<ip-address>:8080 [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

After you login to Jenkins Run the command to copy the Jenkins Admin Password 
``` 
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Enter the Administrator password in jenkins

