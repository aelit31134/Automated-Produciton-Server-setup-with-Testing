# Automated-Produciton-Server-setup-with-Testing
Automated deployment system using Jenkins + Docker + Git + Github

# Aim
This setup aims at creating an automated system which deploys a production server and updates itself when a developer makes changes, tests the production server and notifies the developer if any fault is found. This setup uses Jenkins as the automation tool, Docker for Web Server Provisioning and, Git/Github as SCM.

This setup is a small glimpse of Continuous Integration that can be achieved using Jenkins

This project can be called a successor of my previous project which can be found here.

# Introduction
This setup will run jenkins on a docker as a separate dedicated container, which will start jenkins as soon as the container is. This will be achieved by creating our own docker image using the Dockerfile. After running this container, we will setup jenkins and download some plugins for working with git, github, build pipeline, and email notification. Then, we will begin creating jobs, 5 jobs for this setup in build pipline.

# Getting started
## First of all, we need to create a Dockerfile, which we will use to build our own jenkins dedicated image as follows:

![Dockerfile](https://github.com/aelit31134/Automated-Produciton-Server-setup-with-Testing/blob/master/Setup_images_project3/dockerfile.PNG)

* Create the above file in any folder(/root/project3 in my case) you would like and save it as Dockerfile (make sure the D is upper-case)
* Now we will build this image using the following command and at the end of it we would have created our own jenkins docker image:
        
      docker build -t jenkins:v1 /root/project3/

* Now, the build process will begin and it might take some time so be patient.
* Now, after the build is done successfully, we will have the image created with the name jenkins:v1

* we can see that the image has been created using

      docker image ls

![dockerimages](https://github.com/aelit31134/Automated-Produciton-Server-setup-with-Testing/blob/master/Setup_images_project3/imagesdocker.PNG)

* Now, we need to download some docker images that we will be needing for deployment, you can find images at Docker Hub

    docker pull httpd
    docker pull php:7.1-apache

* Now, we will run our docker container which will directly start jenkins as soon as it is run and help us create jobs for an automatic deployment and testing system:

      docker run -it -p 1234:8080 
                     -v /var/run/docker.sock:/var/run/docker.sock
                     -v /usr/bin/docker:/usr/bin/docker
                     -v /prod:/home
                     --name jens
                            jenkins:v1

![jenkinsrun](https://github.com/aelit31134/Automated-Produciton-Server-setup-with-Testing/blob/master/Setup_images_project3/jenscommandpass0.PNG)

* The -p tag is used for PATing where we are exposing the container to outside world using the port 1234
* We are using the -v tag three times, the first two times are to help us access docker and run containers from inside our jenkins container(a kind of docker-in-docker). Instead of downloading docker insider the container, we are just binding the outside docker libraries insider the container two let it access docker installed on the host system. This is beneficial as we our image will be less bulky and we can use the images already downloaded on the host system, also use the host storage for other contianers we run from inside(this will become more clear as we proceed with the process)
* The third -v tag is used to bind a host folder to a container folder for sharing files with the inside docker containers
* Copy the first time admin password

![adminpass](https://github.com/aelit31134/Automated-Produciton-Server-setup-with-Testing/blob/master/Setup_images_project3/jenscommandpass.PNG)

* Go to a browser and enter the IP of your machine with port number and after jenkins loads, enter the password given above

![jenspass](https://github.com/aelit31134/Automated-Produciton-Server-setup-with-Testing/blob/master/Setup_images_project3/jenspass.PNG)

* Then proceed with skipping the pugins installation and start jenkins
* Now go to configure jenkins and change the password for the user, and login again

![configpass](https://github.com/aelit31134/Automated-Produciton-Server-setup-with-Testing/blob/master/Setup_images_project3/jensconfigurepass.PNG)

![passconfig](https://github.com/aelit31134/Automated-Produciton-Server-setup-with-Testing/blob/master/Setup_images_project3/jensconfigpass.PNG)

* Now install the plugins from configure>manage plugins

![plugins](https://github.com/aelit31134/Automated-Produciton-Server-setup-with-Testing/blob/master/Setup_images_project3/jenspluginsinstall.PNG)

* download the following plugins:
   * Github
   * Build Pipeline
   * Email Extension
* click on install and restart

No alt text provided for this image

* scroll down and select restart after install

## This is the most crucial part of our automated system where we will create jobs in jenkins:
### JOB 1:

This job will continuously keep checking github for new changes and as soon as it sees some changes are made, it will download the changes and copy them to our deployment folder(/home) in our case for access by the container we will run later.

Setup the Job1 as follows:

No alt text provided for this image
No alt text provided for this image
No alt text provided for this image
No alt text provided for this image

### JOB 2:

This is a very interesting job, where the language of the code cloned will be detected and a respective container from a dedicated docker image will a be launched corresponding to the language of the code. This container will deploy the service according to the respective code.

Setup Job 2 as following:

No alt text provided for this image
No alt text provided for this image
No alt text provided for this image

### JOB 3

This job will test the production server(the website), if it is working or not and and will used as testing environment which in turn will work with another job two send an email to the developer to notify him that the production server is down.

Setup job3 as follows:

No alt text provided for this image
No alt text provided for this image
No alt text provided for this image

### JOB 4:

This job will send a mail to the developer or the other recipients we want if the service fails to run. It will send the build messages and a custom message too if we want to set one.

This job will use the Email extension plugin we have downloaded before, Set it up as follows:

* Go to Manage Jenkins> Configure System

No alt text provided for this image

* Now scroll down to till u reach the Extended email notification settings:

No alt text provided for this image
No alt text provided for this image

* Now we can proceed with setting up the job:

No alt text provided for this image
No alt text provided for this image
No alt text provided for this image
No alt text provided for this image

* Select editable Email Notification, the one that is greyed in the above image

No alt text provided for this image
No alt text provided for this image

* Add the above two greyed categories after clicking on advanced
* If this job fails, automatically an email will be sent to the recipient, if it builds successfully, no mail will be sent,

No alt text provided for this image

### JOB 5

This job will keep monitoring our production environment every minute and if by any chance, it goes down(the container running the service stops or terminates), it will automatically start and new one and since the container uses a persistent mounted volume, no data will be lost and same website will be deployed within a minute.

Setup the job as following:

No alt text provided for this image
No alt text provided for this image
No alt text provided for this image
No alt text provided for this image

## Now, we have almost created our final working setup setup and if all goes well, it should look something like this:

No alt text provided for this image

## We will now create a pipeline view of this to have a better graphical representation of job chaining we have created here(all jobs are in a chain, depending on each other):

* Click on the plus symbol besides All shown in the above picture, and select Pipeline:

No alt text provided for this image

* We will get an aesthetic representation of the job chaining using build pipeline:

No alt text provided for this image

### For any queries, feel free to reach me, also check out my other two articles about a Docker based setup and the predecessor of this project for better understsanding.

## REFERENCES:

* https://forums.docker.com/t/how-can-i-run-docker-command-inside-a-docker-container/337
* https://docs.docker.com/storage/volumes/
* https://docs.docker.com/engine/reference/builder/
* https://stelligent.com/2017/04/13/containerizing-jenkins-with-docker-and-centos/
* https://www.edureka.co/blog/email-notification-in-jenkins/
