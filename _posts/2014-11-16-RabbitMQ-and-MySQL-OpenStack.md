---
layout: post
title:  "RabbitMQ and MySQL troubles with OpenStack"
date:   2014-11-16 00:48:42
categories:
- tech
tags: 
- OpenStack 
- Mysql 
- RabbitMQ
---
I’m playing around OpenStack these days to learn the infrastructure part of it. How it’s built, how the pieces work together, how do they communicate, etc. The funny thing I figured out is this two pieces have troubles all the time. It doesn’t matter which Linux distribution or which version of OpenStack (through Havana - IceHouse - Juno). 

**RabbitMQ**

RabbitMQ is a message broker which is used by the OpenStack modules to communicate with each-other, thus quite important part of the stack. RabbitMQ is one of the most common solutions in OpenStack (with ZeroMQ and Qpid). 

It typically fails on the networking side. RabbitMQ is a service mainly running on the controller node and mainly used by the compute nodes, so networking is a must. 

Ubuntu 14.04 LTS:

After the package installation, RabbitMQ starts up on IPv6 only. As my environment is basic IPv4, it means no connections. Although rabbitmq-server man page says: 

    RABBITMQ_NODE_IP_ADDRESS
    
    By default RabbitMQ will bind to all interfaces, on IPv4 and IPv6 if available. 
    Set this if you only want to bind to one network interface or address family.

Debian 7.7 Wheezy: 

After installation RabbitMQ starts on 127.0.0.1 only. It’s because Debian actually contains a config file for the environment variables: 

    # cat /etc/rabbitmq/rabbitmq-env.conf<br>
    RABBITMQ_NODE_IP_ADDRESS=127.0.0.1

**MySQL**

Most of the cases the MySQL installation ends up with a configuration without UTF8. Some of the services need UTF8 support in their databases, so they fail at some point. To avoid make sure the following lines are included in my.cnf: 

    [mysqld]
    
    collation-server = utf8_general_ci
    init-connect = ‘SET NAMES utf8’
    character-set-server = utf8

<!--more-->

