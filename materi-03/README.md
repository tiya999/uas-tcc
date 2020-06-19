Application Containerization and Microservice Orchestration
In this tutorial we will learn about basic application containerization using Docker and running various components of an application as microservices. We will utilize Docker Compose for orchestration during the development. This tutorial is targeted for beginners who have the basic familiarity with Docker. If you are new to Docker, we recommend you check out Docker for Beginners tutorial first.
We will start from a basic Python script that scrapes links from a given web page and gradually evolve it into a multi-service application stack. The demo code is available in the Link Extractor repo. The code is organized in steps that incrementally introduce changes and new concepts. After completion, the application stack will contain the following microservices:
- A web application written in PHP and served using Apache that takes a URL as the input and summarizes extracted links from it
- The web application talks to an API server written in Python (and Ruby) that takes care of the link extraction and returns a JSON response
- A Redis cache that is used by the API server to avoid repeated fetch and link extraction for pages that are already scraped

Stage Setup
Let’s get started by first cloning the demo code repository, changing the working directory, and checking the demo branch out.
-   git clone https://github.com/ibnesayeed/linkextractor.git
    cd linkextractor
    git checkout demo
Step 0: Basic Link Extractor Script
Checkout the step0 branch and list files in it.
-   git checkout step0
    tree
The linkextractor.py file is the interesting one here, so let’s look at its contents:
- cat linkextractor.py
However, this seemingly simple script might not be the easiest one to run on a machine that does not meet its requirements. The README.md file suggests how to run it, so let’s give it a try:
- ./linkextractor.py http://example.com/
When we tried to execute it as a script, we got the Permission denied error. Let’s check the current permissions on this file:
- ls -l linkextractor.py
This current permission -rw-r--r-- indicates that the script is not set to be executable. We can either change it by running chmod a+x linkextractor.py or run it as a Python program instead of a self-executing script as illustrated below:
- python linkextractor.py
Step 1: Containerized Link Extractor Script
Checkout the step1 branch and list files in it.
-   git checkout step1
    tree
We have added one new file (i.e., Dockerfile) in this step. Let’s look into its contents:
- cat Dockerfile
So far, we have just described how we want our Docker image to be like, but didn’t really build one. So let’s do just that:
- docker image build -t linkextractor:step1 .
We have created a Docker image named linkextractor:step1 based on the Dockerfile illustrated above. If the build was successful, we should be able to see it in the list of image:
- docker image ls
This image should have all the necessary ingredients packaged in it to run the script anywhere on a machine that supports Docker. Now, let’s run a one-off container with this image and extract links from some live web pages:
- docker container run -it --rm linkextractor:step1 http://example.com/
Let’s try it on a web page with more links in it:
- docker container run -it --rm linkextractor:step1 https://training.play-with-docker.com/

Step 2: Link Extractor Module with Full URI and Anchor Text
Checkout the step2 branch and list files in it.
-   git checkout step2
    tree
In this step the linkextractor.py script is updated with the following functional changes:
Paths are normalized to full URLs
Reporting both links and anchor texts
Usable as a module in other scripts

Let’s have a look at the updated script:
- cat linkextractor.py
Now, let’s build a new image and see these changes in effect:
- docker image build -t linkextractor:step2 .
We have used a new tag linkextractor:step2 for this image so that we don’t overwrite the image from the step0 to illustrate that they can co-exist and containers can be run using either of these images.
- docker image ls
Running a one-off container using the linkextractor:step2 image should now yield an improved output:
- docker container run -it --rm linkextractor:step2 https://training.play-with-docker.com/
Running a container using the previous image linkextractor:step1 should still result in the old output:
- docker container run -it --rm linkextractor:step1 https://training.play-with-docker.com/
Step 3: Link Extractor API Service
Checkout the step3 branch and list files in it.
-   git checkout step3
    tree
The following changes have been made in this step:
    Added a server script main.py that utilizes the link extraction module written in the last step
    The Dockerfile is updated to refer to the main.py file instead
    Server is accessible as a WEB API at http://<hostname>[:<prt>]/api/<url>
    Dependencies are moved to the requirements.txt file
    Needs port mapping to make the service accessible outside of the container (the Flask server used here listens on port 5000 by default)

Let’s first look at the Dockerfile for changes:
- cat Dockerfile
The linkextractor.py module remains unchanged in this step, so let’s look into the newly added main.py file:
- cat main.py
It’s time to build a new image with these changes in place:
- docker image build -t linkextractor:step3 .
Then run the container in detached mode (-d flag) so that the terminal is available for other commands while the container is still running. Note that we are mapping the port 5000 of the container with the 5000 of the host (using -p 5000:5000 argument) to make it accessible from the host. We are also assigning a name (--name=linkextractor) to the container to make it easier to see logs and kill or remove the container.
- docker container run -d -p 5000:5000 --name=linkextractor linkextractor:step3
If things go well, we should be able to see the container being listed in Up condition:
- docker container ls
Running a one-off container using the linkextractor:step2 image should now yield an improved output:
- docker container run -it --rm linkextractor:step2 https://training.play-with-docker.com/
Running a container using the previous image linkextractor:step1 should still result in the old output:
- docker container run -it --rm linkextractor:step1 https://training.play-with-docker.com/

Step 3: Link Extractor API Service
Checkout the step3 branch and list files in it.
-   git checkout step3
    tree
Let’s first look at the Dockerfile for changes:
- cat Dockerfile