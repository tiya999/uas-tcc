Section #1 - Networking Basics
Step 1: The Docker Network Command
The docker network command is the main command for configuring and managing container networks. Run the docker network command from the first terminal.
- docker network

Step 2: List networks
Run a docker network ls command to view existing container networks on the current Docker host.
- docker network ls

Step 3: Inspect a network
The docker network inspect command is used to view network configuration details. These details include; name, ID, driver, IPAM driver, subnet info, connected containers, and more.
Use docker network inspect <network> to view configuration details of the container networks on your Docker host. The command below shows the details of the network called bridge.
- docker network inspect bridge

Step 4: List network driver plugins
The docker info command shows a lot of interesting information about a Docker installation.
Run the docker info command and locate the list of network plugins.
- docker info

Section #2 - Bridge Networking
Step 1: The Basics
Every clean installation of Docker comes with a pre-built network called bridge. Verify this with the docker network ls.
- docker network ls
All networks created with the bridge driver are based on a Linux bridge (a.k.a. a virtual switch).
Install the brctl command and use it to list the Linux bridges on your Docker host. You can do this by running sudo apt-get install bridge-utils.
- apk update
- apk add bridge
Then, list the bridges on your Docker host, by running brctl show.
- brctl show
You can also use the ip a command to view details of the docker0 bridge.
- ip a
Step 2: Connect a container
The bridge network is the default network for new containers. This means that unless you specify a different network, all new containers will be connected to the bridge network.
Create a new container by running docker run -dt ubuntu sleep infinity.
- docker run -dt ubuntu sleep infinity
This command will create a new container based on the ubuntu:latest image and will run the sleep command to keep the container running in the background. You can verify our example container is up by running docker ps.
- docker ps
Run the brctl show command again.
- brctl show
You can inspect the bridge network again, by running docker network inspect bridge, to see the new container attached to it.
- docker network inspect bridge
Step 3: Test network connectivity
The output to the previous docker network inspect command shows the IP address of the new container. In the previous example it is “172.17.0.2” but yours might be different.
Ping the IP address of the container from the shell prompt of your Docker host by running ping -c5 <IPv4 Address>. Remember to use the IP of the container in your environment.
The replies above show that the Docker host can ping the container over the bridge network. But, we can also verify the container can connect to the outside world too. Lets log into the container, install the ping program, and then ping www.github.com.
First, we need to get the ID of the container started in the previous step. You can run docker ps to get that.
- docker ps
Next, lets run a shell inside that ubuntu container, by running docker exec -it <CONTAINER ID> /bin/bash.
Next, we need to install the ping program. So, lets run apt-get update && apt-get install -y iputils-ping.
- apt-get update && apt-get install -y iputils-ping
Lets ping www.github.com by running ping -c5 www.github.com
- ping -c5 www.github.com
Finally, lets disconnect our shell from the container, by running exit.
- exit

Step 4: Configure NAT for external connectivity
In this step we’ll start a new NGINX container and map port 8080 on the Docker host to port 80 inside of the container. This means that traffic that hits the Docker host on port 8080 will be passed on to port 80 inside the container.
Start a new container based off the official NGINX image by running docker run --name web1 -d -p 8080:80 nginx.
- docker run --name web1 -d -p 8080:80 nginx
Review the container status and port mappings by running docker ps.
- docker ps
If for some reason you cannot open a session from a web broswer, you can connect from your Docker host using the curl 127.0.0.1:8080 command.
- curl 127.0.0.1:8080
Section #3 - Overlay Networking
Step 1: The Basics
In this step you’ll initialize a new Swarm, join a single worker node, and verify the operations worked.
Run docker swarm init --advertise-addr $(hostname -i).
- docker swarm init --advertise-addr $(hostname -i)
Run a docker node ls to verify that both nodes are part of the Swarm.
- docker node ls

Step 4: Test the network
To complete this step you will need the IP address of the service task running on node2 that you saw in the previous step (10.0.0.3).
Execute the following commands from the first terminal.
- docker network inspect overnet
Run a docker ps command to get the ID of the service task so that you can log in to it in the next step.
- docker ps
Install the ping command and ping the service task running on the second node where it had a IP address of 10.0.0.3 from the docker network inspect overnet command.
- apt-get update && apt-get install -y iputils-ping
Step 5: Test service discovery
Now that you have a working service using an overlay network, let’s test service discovery.
If you are not still inside of the container, log back into it with the docker exec -it <CONTAINER ID> /bin/bash command.
Run cat /etc/resolv.conf form inside of the container.
docker exec -it yourcontainerid /bin/bash
- cat /etc/resolv.conf

Cleaning Up
Hopefully you were able to learn a little about how Docker Networking works during this lab. Lets clean up the service we created, the containers we started, and finally disable Swarm mode.
Execute the docker service rm myservice command to remove the service called myservice.
- docker service rm myservice
Execute the docker ps command to get a list of running containers.
- docker ps
Finally, lets remove node1 and node2 from the Swarm. We can use the docker swarm leave --force command to do that.
Lets run docker swarm leave --force on node1.
- docker swarm leave --force
Lets also run docker swarm leave --force on node2.
- docker swarm leave --force