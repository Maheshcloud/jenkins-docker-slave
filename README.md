# jenkins-docker-slave

this repo contains Dockerfile for creating custom Jenkins docker slave.
you need to download the code and perform docker build.

Pre-requisites:
Jenkins Master is already setup and running
port 8080 opened in Jenkins EC2's firewall rule
Setup Docker host. Creating another EC2 instance is recommended to serve as a Docker Host. 
port 4243 opened in docker host machine 
32768 - 60999 opened in docker host machine 

Step 1 - Configure Docker Host with Remote API
Login to Docker host machine. Open docker service file. Search for ExecStart and replace that line with the following.
sudo vi /lib/systemd/system/docker.service


You can replace with below line:
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock

Restart Docker service
   sudo systemctl daemon-reload
   sudo service docker restart

Validate API by executing below curl command
curl http://localhost:4243/version


Step 2 - Build Jenkins slave Docker image
Download Dockerfile from below repo.
git clone https://github.com/akannan1087/jenkins-docker-slave; cd jenkins-docker-slave

Build Docker image

sudo docker build -t my-jenkins-slave .


Perform below command to see the list of docker images:

sudo docker images


Step 3 - Configure Jenkins Server with Docker plug-in
Now login to Jenkins Master. Make sure you install Docker plug-in in Jenkins.



Now go to Manage Jenkins -> Configure Nodes Cloud



Click on Docker Cloud Details

Enter docker host dns name or ip address
tcp://docker_host_dns:4243
Make sure Enabled is selected
Now click on Test Connection to make sure connecting with docker host is working. 




Step 4 - Configure Docker Agent Templates
Now click on Docker Agent templates:
Enter label as "docker-slave" and give some name
Click on Enabled
Now enter the name of the docker image you have built previously in docker host.
enter /home/jenkins as Remote file system root
 


Choose Connect with SSH as connection method:
Enter SSH credentials per your Dockerfile - jenkins/password



choose Never Pull as pull strategy as we have already image stored in DockerHost.

Click on Save.
Step 5 - Create build job in Jenkins
Now Create a pipeline job in Jenkins with below pipeline code:

pipeline {
    agent { 
        label "docker-slave"
     }
    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}

Click Apply and Save. 
Now build the job. Now you will see output like below:



