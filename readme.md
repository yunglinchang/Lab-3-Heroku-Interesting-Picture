# 95-702 Distributed Systems for Information Systems Management
# Lab 3 - Creating Containers and Deploying to the Cloud

## Part 1: Running a Docker Image Locally

### 1.1 Install Docker

1. Go to 

https://docs.docker.com/install/  

and Install Docker CE (Community Edition) according to your system.  Scroll down to find links for Mac and Windows downloads.

2. After installation, make sure you have the Docker daemon running background.
Use the command docker ps to check if it is running.  You should see table headers with empty rows:

        CONTAINER ID    IMAGE   COMMAND CREATED STATUS  PORTS   NAMES


3. Run the Hello World test image to see if you docker runs correctly:

        docker run hello-world

4. If you see the following message, that means you have installed docker correctly.


        Hello from Docker!

        This message shows that your installation appears to be working correctly.

        To generate this message, Docker took the following steps:
        1. The Docker client contacted the Docker daemon.
        2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
        3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
        4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

        To try something more ambitious, you can run an Ubuntu container with:
         $ docker run -it ubuntu bash

        Share images, automate workflows, and more with a free Docker ID:
         https://hub.docker.com/

        For more examples and ideas, visit:
         https://docs.docker.com/get-started/

The ubuntu command they suggest trying runs a container with a command shell.

### 1.2 Creating a Custom Docker Container

1. Creating a war file (see https://www.jetbrains.com/help/idea/creating-and-running-your-first-java-ee-application.html)

- create a directory called "docker"; remember where it is.
- in IntelliJ, go to your InterestingPicture project and select File -> Project Structure.
	- click on Artifacts
	- click +, select Web Application:Archive, and select "For InterestingPicture: war exploded"
	- if the message "Library 'lib' required .. the artifact" and a Fix button appear, DO NOT CLICK on Fix.
	- click Create Manifest and agree to the suggested location
	- click OK in the Project Structure
	- choose Build -> Build Artifacts. Point to InterestingPicture:war and choose Build
- right click on it to bring up its Properties, where you can see its directory
  path (or choose Reveal in Finder on a Mac).
- you should see the war file in the Project tab under out -> artifacts -> InterestingPicture_war, named InterestingPicture_war.war
- copy the war file to your docker directory, but re-name it ROOT.war

2. Creating a custom docker container using Dockerfile
- in the docker directory, copy the file named "Dockerfile" (note: capitalized)
  from github. Save it to your docker directory. This file contains Docker commands to create a docker container with openjdk12 and Tomcat. It then removes the default web app and copies your war file to the tomcat/webapps directory. Directions:

- save this file – but if you’re using Windows, do these two things:
- make sure the file uses UNIX/Linux line endings. In Notepad++, use Edit > EOL Conversion > UNIX Format.
- ***make sure the file does NOT have a .txt extension*** if you're
  using Windows. For example, in Notepad++, choose Save As, type the name in
  quotes as "Dockerfile", change the File Type from Text Documents (\*.txt) to
  All Files (\*.\*), and click Save.

### 1.3. Test your Docker image locally:

-In the folder docker, you should have the following structure:

    docker/
    ├── Dockerfile
    └── ROOT.war 

- from the docker directory in a terminal or CMD window, build the docker image with the command (note the space and period at the end); this will take a few seconds, and you'll see docker output scrolling by.

        docker build -t interesting_picture .

- you should see all the images by running:

        docker images

It will display something like this (details will vary):

        REPOSITORY            TAG       IMAGE ID        CREATED          SIZE
        interesting_picture   latest    861ece9f7deb    2 minutes ago    108MB

- deploy the image in a container locally. The -p flag maps the actual port on
  your machine to the Docker container's port. If 8080 is in use, change the first number to something else (say, 9000) and use that number in the browser instead.

        docker run --rm -it -p 8080:8080 interesting_picture

- open a browser window with the address:

        http://localhost:8080/

- you should see your app running. Test it to make sure it works correctly (again, use the port number from the run command if 8080 didn’t work for you) 

### :checkered_flag: **Checkpoint:** This is the Checkpoint for Lab 3.

## Part 2: Running on the Cloud

### 2.1 Get started with the Heroku cloud provider
1. Create a Heroku account: Go to https://signup.heroku.com/login and register
an account. You may choose your role as Student.  

2. Follow the instructions and install heroku-cli on your system. Note that you must have Git installed in your system.  See https://devcenter.heroku.com/articles/heroku-cli

3. This website also contains the link to configure Git if you don't have it.
- Git installation
- First-time Git setup

4. After installation, open your terminal or CMD window and type

        heroku login

5. Press enter and it will prompt and navigate you to the browser. Then click log in. Your computer should have access to Heroku services, including a dashboard (although the steps below use the cli).

### 2.2 Creating and Pushing a Container

1. In a new directory named "heroku" (just to keep it separate from the local Dockerfile
above in the docker directory), copy the Dockerfile from part 1and make the following two changes – see the note above about Windows files: this needs to be a text file without the .txt extension, and it must use UNIX/Linux line endings.

***Change # 1:***

Before the last line (which says CMD ["catalina.sh", "run"] ), un-comment the following lines (remove the hashtags):

        ADD tomcat_starter.sh /home/
        CMD chmod +x /home/tomcat_starter.sh; /home/tomcat_starter.sh

***Change # 2:***

Comment out the last line so that it looks like this (deleting it is okay, too):

        #CMD ["catalina.sh", "run"]

2. Create this bash shell script, a text file named “tomcat_starter.sh”; again,
in Windows, follow the directions above about line endings; this is a text file,
but with the .sh extension, *NOT* .txt or .sh.txt. Note: the first line is ***required*** to begin
with a hashtag; it's not a comment.

        #!/bin/bash
        # Change the configuration of Tomcat so that it listens to
        # the port assigned by Heroku
        sed -i s/8080/$PORT/ /usr/local/tomcat/conf/server.xml
        # delete the default ROOT directory so ROOT.war is used
        rm -rf /usr/local/tomcat/webapps/ROOT
        # start the server
        catalina.sh run

3. Copy the ROOT.war file from the docker directory (or from IntelliJ, again). So there should be three files in this directory: ROOT.war, Dockerfile, tomcat_starter.sh

4. Run this series of heroku commands, one at a time; this may take a few
minutes, so be patient:

        heroku container:login
        heroku create

**->** This will display an system generated app-name; the commands below use "serene-basin-70362",
but yours will differ: heroku assigns this name to your app; copy it carefully.

        heroku container:push web -a serene-basin-70362
        heroku container:release web -a serene-basin-70362
        heroku open -a serene-basin-70362

The InterestingPicture app should show up in your browser.

A few utilities if things go wrong:

        heroku apps # which apps you've pushed
        heroku apps:destroy serene-basin-70362 # you're only allowed 5 on the free tier
        heroku logs --tail -a serene-basin-70362 # if you get a heroku error message in the browser


## Part 3: Cloud and Containers Concepts

1. Read this article:

https://www.infoworld.com/article/3077875/linux/containers-101-docker-fundamentals.html

2. Read "A Descriptive Literature Review and Classification of Cloud Computing Research", journal pages 36 through 39 (PDF pages 3 through 6).

http://aisel.aisnet.org/cgi/viewcontent.cgi?article=3672&context=cais

You must be on campus, or using the campus VPN, to view this article.

3. Answer these two questions. This is not part of getting credit, but you need to know this.

 i) Is a service like AWS an example of IaaS, Paas, or SaaS?

 ii) What property makes Docker containers suitable for version control?


### :checkered_flag: **LAB CREDIT:  To get full lab credit, show a TA:
  a) InterestingPicture running in local Docker - checkpoint

  b) InterestingPicture running on Heroku – the rest

