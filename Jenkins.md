# Introduction to Jenkins 

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.

In this project we are going to start automating part of our routine tasks with a free and open source automation server - Jenkins. It is one of the mostl popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named “Hudson”.

Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub 
`https://github.com/<yourname>/`
tooling will be automatically be updated to the Tooling Website.

## Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how your update architecture will look like upon competion of this project:

![alt text](image1.jpg)


## Step 1 - Install Jenkins server
1.  Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it “Jenkins”
2. Install JDK (since Jenkins is a Java-based application)
```
sudo apt update
sudo apt install default-jdk-headless
```

3. Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```

4. Make sure Jenkins is up and running
```
sudo systemctl status jenkins

```


![alt text](image2.jpg)

5. By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your EC2 Security Group

6. Perform initial Jenkins setup.
From your browser access 
```
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
```

7. You will be prompted to provide a default admin password

Retrieve it from your server:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

```

Then you will be asked which plugings to install - choose suggested plugins.

![alt text](image3.jpg)

![alt text](image4.jpg)

## Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks

We will configure a simple Jenkins job.This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1.Enable webhooks in your GitHub repository settings
