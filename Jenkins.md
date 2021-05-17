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

![alt text](image5.jpg)

2.Go to Jenkins web console, click “New Item” and create a “Freestyle project”

![alt text](image6a.jpg)

To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself.
In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.
Save the configuration and run the build. For now we can only do it manually. Click “Build Now” button, if you have configured everything correctly, the build will be successfull and you will see it under #1

![alt text](image6.jpg)

3.To automatically trigger a build when a push is done on github, Click “Configure” your job/project and add these two configurations

a. In the build triggers section, check the box beside 'Github hook trigger for GITscm polling'

![alt text](image8.jpg)

b.Configure “Post-build Actions” to archive all the files then save.


Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results - artifacts, saved on Jenkins server.

![alt text](image7.jpg)

By default, the artifacts are stored on Jenkins server locally

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

![alt text](image9.jpg)

## Step 3 - Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to `/mnt/apps` directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called “Publish Over SSH”.

1. Install “Publish Over SSH” plugin.
2. On main dashboard select “Manage Jenkins” and choose “Manage Plugins” menu item.

3. On “Available” tab search for “Publish Over SSH” plugin and install it

![alt text](image10.jpg)

4. Configure the job/project to copy artifacts over to NFS server.
On main dashboard select “Manage Jenkins” and choose “Configure System” menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:


a. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
b. Arbitrary name
c. Hostname - can be private IP address of your NFS server
d. Username - ec2-user (since NFS server is based on EC2 with RHEL 8)
Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.


![alt text](image11.jpg)

Save the configuration, open your Jenkins job/project configuration page and add another one “Post-build Action”

![alt text](image12.jpg)

Configure it to send all files produced by the build into our previously defined remote directory. In our case we want to copy all files and directories - so we use **.

![alt text](image13.jpg)

Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.Webhook will trigger a new job and in the “Console Output” of the job you will find something like this:

![alt text](image13.jpg)

NB: If you get this instead; 

```
SSH: Connecting from host [localhost.localdomain]
SSH: Connecting with configuration [love] ...
SSH: Disconnecting configuration [love] ...
ERROR: Exception when publishing, exception message [Permission denied]
Archiving artifacts
Finished: UNSTABLE

```
This means that jenkins doesn't have access to send all files to the `/mnt/apps` folder on your NFS server, you should change ownership of your `/mnt/apps` folder to the jenkins user. An elegant way of achieving this is to add a variable as a placeholder for your jenkins user.

Run this command:
`sudo chown -R $USER:$USER /mnt/apps`


To make sure that the files in `/mnt/apps` have been udated - connect via SSH/Putty to your NFS server and check `README.MD` file

Run `cat /mnt/apps/README.md`
