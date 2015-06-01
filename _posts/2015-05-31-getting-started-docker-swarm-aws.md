---
layout: post
title:  "Getting Started with Docker Swarm on AWS"
date:   2015-05-31 23:48:42+0100
categories: 
tags: 
- AWS
- Docker
- Swarm
---
Swarm is the future native clustering for Docker. Currently it is beta and not recommended for production, 
but it's a very interesting technology to play with. For this guide I'm using only the free tier of 
Amazon Web Services (AWS), that means building the cluster below works without additional cost, however
a credit card is required to set up an AWS account (this guide is not going to be detailed on how to use
AWS in general).

**1./ Creating instances, setting up the environment**

Simply use the 'Launch instance' from the AWS Console. The free tier allows us to run t2.micro instances
which are basically 1 vCPU and 1G of RAM. Not a monster, but it will do the job. I started my cluster with
3 nodes, but it's not a requirement. Using the default answers is absolutely OK, however I created a new
security group which I use with this cluster (let's name it "swarm"). 

The security group will need additional configuration. By default one single inbound rule is created to allow
SSH traffic from everywhere (0.0.0.0/0) to these instances. I usually change this to "All Traffic" (I use more
than just SSH) and the source to "My IP". Selecting "My IP" actually gets the public IP address of my computer and 
inserts into the rule. The second rule I use is to allow traffic between the instances. For this, you'll need to 
check the private IP of the instance (usually starting with 172) and the netmask (usually /28). 

Once all of these are done, instances are up and they can communicate to each other, we are ready to install 
Docker.

**2./ Installing Docker**

Installing Docker on Ubuntu is a very simple task, involves three steps. It's documented on the Docker website, 
the cookbook can be found [here][docker-install]. The default installation of Docker is using unix sockets for 
the communication between the client and the server process. In a Swarm environment TCP method will be used, so
after the installation it has to be changed. On Ubuntu it's simply done in the /etc/default/docker file: 

    DOCKER_OPTS="-H tcp://ip:2375"

At this point, it's very important to NOT use the public IP of the instance. The public IP is dynamically
allocated every time when the instance is started/restarted. After Docker is restarted with the "-H" option, 
the Docker client will not be able to connect, still checking the unix socket. To change the behaviour, an environment
variable can be used:

    export DOCKER_HOST=tcp://ip:2375

Next up, Swarm.

**3./ Install and configure Swarm**

Installing Swarm is not hard. In fact, Docker includes an official image to run Swarm, it basically contains the
"swarm" binary, ready to run. Let's pull it down:

    $ docker pull swarm

Docker 1.4 or higher is required to use Swarm. In the next step the uniqe cluster ID will be created. This step is 
only required to run once. 

    $ docker run --rm swarm create

This command will return the unique cluster ID. This ID will be used to join nodes and manage the cluster. 

    $ docker run -d swarm join --addr=IP:2375> token://<ID>

This will basically start a container on the specific node we want to join into the cluster. This container will 
run the swarm binary which will listen on the local IP on TCP port 2375 (again, don't use the instance public IP). 
After the Swarm container has been started, the output should look similar: 

    ubuntu@docker1:~$ docker ps
    CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
    4f4f230bf45b        swarm:latest        "/swarm join --addr=   2 hours ago         Up 2 hours          2375/tcp            swarm1              

As an additional option I use the "-\-name=swarm1" parameter on the first node, so if the docker daemon or the host 
is restarted, the Swarm can be kicked off with the simple "docker start swarm1" command. This step needs to be repeated on 
every node/instance in the Swarm cluster (with the same ID). 

**4./ Swarm Manager**

Swarm Manager is the tool used to manage the Swarm Cluster. It also comes with the Swarm image, so the installation
is extremely simple:

    $ docker run -d -p <swarm_port>:2375 swarm manage token://<ID>

The Manager needs to run on only one instance/node. Similar to the Swarm container, the "-\-name=" parameter can be
used, so after a restart it's easy to bring up. Once the Manager is running, we can use the Docker client to connect
to the port we defined and get the docker info: 

    ubuntu@docker3:~$ docker -H tcp://localhost:4444 info
    Containers: 33
    Strategy: spread
    Filters: affinity, health, constraint, port, dependency
    Nodes: 3
     docker1: 172.31.37.240:2375
      └ Containers: 23
      └ Reserved CPUs: 0 / 1
      └ Reserved Memory: 0 B / 1.018 GiB
     docker2: 172.31.37.241:2375
      └ Containers: 5
      └ Reserved CPUs: 0 / 1
      └ Reserved Memory: 0 B / 1.018 GiB
     docker3: 172.31.37.242:2375
      └ Containers: 5
      └ Reserved CPUs: 0 / 1
      └ Reserved Memory: 0 B / 1.018 GiB

...or docker ps: 

    ubuntu@docker3:~$ docker -H tcp://localhost:4444 ps
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    ee1c8c07c0ae        ubuntu:latest       "/bin/bash"         About an hour ago   Up About an hour                        docker1/ecstatic_hodgkin   
    10811b99ee9a        ubuntu:latest       "/bin/bash"         2 hours ago         Up 2 hours                              docker2/boring_fermat      
    94b416802dad        ubuntu:latest       "/bin/bash"         2 hours ago         Up 2 hours                              docker3/modest_curie       
    5b8f9de2c7d4        ubuntu:latest       "/bin/bash"         2 hours ago         Up 2 hours                              docker3/focused_carson     

The last column of the ps command shows on which node the container is actually running. Now, the fun only starts here. 
Swarm will use the Docker nodes/instances to create a single pool where the containers are started. Swarm will keep the
number of the containers equal between the nodes. For this, we need to connect to the Swarm Manager port:

    ubuntu@docker3:~$ docker -H tcp://localhost:4444 run -ti ubuntu /bin/bash

or using a different node: 

    ubuntu@docker1:~$ docker -H tcp://docker3:4444 run -ti ubuntu /bin/bash

the latest command actually created a container on the docker1 instance running the command from docker1 connecting to 
the manager container running on docker3. 

    ubuntu@docker1:~$ docker ps
    CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
    ac74d9338142        ubuntu:latest       "/bin/bash"            6 seconds ago       Up 6 seconds                            cocky_morse      

Please note, the "docker ps" on docker1 without parameters connects to the local Docker daemon instead of the Manager, 
that's why it lists only one running container. 


[docker-install]:   https://get.docker.io/ubuntu/

<!--more-->

