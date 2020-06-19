Section 1: What is Orchestration
So, what is Orchestration anyways? Well, Orchestration is probably best described using an example. Let’s say that you have an application that has high traffic along with high-availability requirements. Due to these requirements, you typically want to deploy across at least 3+ machines, so that in the event a host fails, your application will still be accessible from at least two others. Obviously, this is just an example and your use-case will likely have its own requirements, but you get the idea.

Deploying your application without Orchestration is typically very time consuming and error prone, because you would have to manually SSH into each machine, start up your application, and then continually keep tabs on things to make sure it is running as you expect.

But, with Orchestration tooling, you can typically off-load much of this manual work and let automation do the heavy lifting. One cool feature of Orchestration with Docker Swarm, is that you can deploy an application across many hosts with only a single command (once Swarm mode is enabled). Plus, if one of the supporting nodes dies in your Docker Swarm, other nodes will automatically pick up load, and your application will continue to hum along as usual.

If you are typically only using docker run to deploy your applications, then you could likely really benefit from using Docker Compose, Docker Swarm mode, or both Docker Compose and Swarm.

Section 2: Configure Swarm Mode
Real-world applications are typically deployed across multiple hosts as discussed earlier. This improves application performance and availability, as well as allowing individual application components to scale independently. Docker has powerful native tools to help you do this.

An example of running things manually and on a single host would be to create a new container on node1 by running docker run -dt ubuntu sleep infinity.
- docker run -dt ubuntu sleep infinity
This command will create a new container based on the ubuntu:latest image and will run the sleep command to keep the container running in the background. You can verify our example container is up by running docker ps on node1.
- docker ps

Step 2.1 - Create a Manager node
In this step you’ll initialize a new Swarm, join a single worker node, and verify the operations worked.
Run docker swarm init on node1.
- docker swarm init --advertise-addr $(hostname -i)
You can run the docker info command to verify that node1 was successfully configured as a swarm manager node.
- docker info

Step 2.2 - Join Worker nodes to the Swarm
You will perform the following procedure on node2 and node3. Towards the end of the procedure you will switch back to node1.
Now, take the entire docker swarm join ... command we copied earlier from node1 where it was displayed as terminal output. We need to paste the copied command into the terminal of node2 and node3.
It should look something like this for node2. By the way, if the docker swarm join ... command scrolled off your screen already, you can run the docker swarm join-token worker command on the Manager node to get it again.
Once you have run this on node2 and node3, switch back to node1, and run a docker node ls to verify that both nodes are part of the Swarm. You should see three nodes, node1 as the Manager node and node2 and node3 both as Worker nodes.
- docker node ls

Section 3: Deploy applications across multiple hosts
Now that you have a swarm up and running, it is time to deploy our really simple sleep application.
You will perform the following procedure from node1.
Step 3.1 - Deploy the application components as Docker services
Our sleep application is becoming very popular on the internet (due to hitting Reddit and HN). People just love it. So, you are going to have to scale your application to meet peak demand. You will have to do this across multiple hosts for high availability too. We will use the concept of Services to scale our application easily and manage many containers as a single entity.
You will perform this procedure from node1.
Let’s deploy sleep as a Service across our Docker Swarm.
- docker service create --name sleep-app ubuntu sleep infinity
Verify that the service create has been received by the Swarm manager.
- docker service ls

Section 4: Scale the application
Demand is crazy! Everybody loves your sleep app! It’s time to scale out.
One of the great things about services is that you can scale them up and down to meet demand. In this step you’ll scale the service up and then back down.
You will perform the following procedure from node1.
Scale the number of containers in the sleep-app service to 7 with the docker service update --replicas 7 sleep-app command. replicas is the term we use to describe identical containers providing the same service.
- docker service update --replicas 7 sleep-app
The Swarm manager schedules so that there are 7 sleep-app containers in the cluster. These will be scheduled evenly across the Swarm members.
We are going to use the docker service ps sleep-app command. If you do this quick enough after using the --replicas option you can see the containers come up in real time.
- docker service ps sleep-app
Scale the service back down to just four containers with the docker service update --replicas 4 sleep-app command.
- docker service update --replicas 4 sleep-app
Verify that the number of containers has been reduced to 4 using the docker service ps sleep-app command.
- docker service ps sleep-app

Section 5: Drain a node and reschedule the containers
Your sleep-app has been doing amazing after hitting Reddit and HN. It’s now number 1 on the App Store! You have scaled up during the holidays and down during the slow season. Now you are doing maintenance on one of your servers so you will need to gracefully take a server out of the swarm without interrupting service to your customers.
Take a look at the status of your nodes again by running docker node ls on node1.
- docker node ls
Let’s see the containers that you have running on node2.
- docker ps
Now let’s jump back to node1 (the Swarm manager) and take node2 out of service. To do that, let’s run docker node ls again.
- docker node ls
Switch back to node2 and see what is running there by running docker ps.
- docker ps
Lastly, check the service again on node1 to make sure that the container were rescheduled. You should see all four containers running on the remaining two nodes.
- docker service ps sleep-app

Cleaning Up
Execute the docker service rm sleep-app command on node1 to remove the service called myservice.
- docker service rm sleep-app

Execute the docker ps command on node1 to get a list of running containers.
- docker ps

Finally, let’s remove node1, node2, and node3 from the Swarm. We can use the docker swarm leave --force command to do that.
Lets run docker swarm leave --force on node1.
- docker swarm leave --force

Then, run docker swarm leave --force on node2.
- docker swarm leave --force

Finally, run docker swarm leave --force on node3.
- docker swarm leave --force

Congratulations! You’ve completed this lab. You now know how to build a swarm, deploy applications as collections of services, and scale individual services up and down.