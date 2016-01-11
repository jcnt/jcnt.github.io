---
layout: post
title:  "Docker Swarm on AWS with Cloud ONTAP for persistent data"
date:   2016-01-10 18:28:42+0100
categories: 
tags: 
- AWS
- NetApp
- ONTAP
- Cloud
- Docker
- Swarm
- persistent
- storage
---
Most of the information covered in this post has been published earlier here, so it's more like a collection of howto's. But first of all, why would you need persistent data for your containers? Well, containers are certainly different from VMs, but still, some requires to work on persistent data which is available for more than one instance on more than one node or host (physical or virtual).  The easiest way to achieve this would be a file based protocol, like NFS which can be used as a docker volume attached to the instance at startup. Amazon Web Services provides NFS service, however there are certain benefits to use Cloud ONTAP instead, most importantly the following: 

- Efficiency of deduplication and compression - This sounds old, but definitely helps to decrease the costs attached to public cloud usage
- Easy data movement between different locations with SnapMirror or SnapVault
- Data privacy using Cloud ONTAP encryption 

![Cloud ONTAP initialization](/assets/2016-01-10-docker-swarm-with-cloud-ontap-aws/clntap-init.png)


<!--more-->

**1./ Setting up OnCommand Cloud Manager**

As a first step you need Cloud Manager to deploy Cloud ONTAP. The process is described in this [article](/2015/07/12/getting-started-cloud-manager.html){:target="_blank"}. It has changed a bit as the Microsoft Windows based Cloud Manager is not available on AWS anymore. The current version is Linux based, that also means, it's faster to setup and easier to access - for me, connecting over SSH is always a plus compared to RDP. At the end of the day it doesn’t really matter, as the tool is web based anyway… Once Cloud Manager is running, simply copy the public DNS address to your web browser and start using it. 

**2./ Deploy Cloud ONTAP**

Simply follow this [post](/2015/08/02/getting-started-cloud-ontap.html){:target="_blank"}. Once it's done, you should have a Cloud ONTAP instance running with an NFS share. 

**3./ Create a Docker Swarm cluster**

Although this [post](/2015/06/01/getting-started-docker-swarm-aws.html){:target="_blank"} is written for Docker 1.5 (or 1.6) it still works fine. Creating a Swarm cluster is not very difficult, it’s really just downloading a binary and running it in a container on every host. 

**4./ Final steps**

Now you are ready to run some containers using the data shared on the NFS volume. There’s a great article by [Andrew Sullivan](https://twitter.com/andrew_ntap){:target="_blank"} on the [NetApp Communities](http://community.netapp.com/t5/Technology/Docker-Volumes-Using-NetApp-Storage/ba-p/108159){:target="_blank"} how to use Docker with NFS on NetApp, there are great details there. First, you’ll need to mount the nfs volume on all nodes in the cluster:

	root@swarm1:~# mount -t nfs 172.31.1.22:/swarmtest /mnt
	root@swarm1:~# df -h | grep swarm
	172.31.1.22:/swarmtest   24G  192K   24G   1% /mnt
	root@swarm1:~#

Once it’s done, you can start the containers using the -v parameter like this: 

	root@swarm1:~# docker run -d --name=web1 -v /mnt/stuff:/mnt/nginx:ro -p 80:80 nginx
	8e6459a946731e35d33c55fb80036c2346f0422d5c090d4bb7d044c4bd175939
	root@swarm1:~#

As I have 3 nodes in my Swarm cluster and no containers are running, I can use the scheduler to start a container on each host (let’s say probably): 

	root@swarm1:~# for i in `seq 6`; do docker run -d --name=web$i -v /mnt/stuff:/mnt/nginx:ro -p 80:80 nginx; done
	667d44eae692dd7d618fa08a937a9a1dc5741103681fcb748d50f89a77213936
	5ada9b3900228f44f9a23db50fde984c2926a04f25e824392e2b015c8794c134
	02be7b97530f93037f188286e2bc8513e47451b2f02d83150855f23710797f16
	root@swarm1:~#

Most of these cases I get the containers running nicely distributed: 

	root@swarm1:~# docker ps
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
	02be7b97530f        nginx               "nginx -g 'daemon off"   30 seconds ago      Up 30 seconds       443/tcp, 172.31.9.250:80->80/tcp   swarm1/web3
	667d44eae692        nginx               "nginx -g 'daemon off"   44 seconds ago      Up 43 seconds       172.31.9.251:80->80/tcp, 443/tcp   swarm2/web1
	5ada9b390022        nginx               "nginx -g 'daemon off"   44 seconds ago      Up 44 seconds       172.31.9.252:80->80/tcp, 443/tcp   swarm3/web2

Basically, using NFS on ONTAP or Cloud ONTAP helps to distribute static content used by containers between multiple sites and AWS with simple tools like SnapMirror. 
