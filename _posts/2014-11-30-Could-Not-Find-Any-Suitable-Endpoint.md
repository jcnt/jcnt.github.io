---
layout: post
title:  "Nova: Could not find any suitable endpoint. Correct region?"
date:   2014-11-30 09:33:42
categories:
- tech
tags: 
- OpenStack 
- Nova 
- Keystone
---
Nova setup is just done, database configuration is done, novadb database exist on the control node, but still, nova doesn’t work. 

    # nova service-list
    Could not find any suitable endpoint. Correct region?
    ERROR (EndpointNotFound):

One of the things worth to look at if nova exists in the keystone service list: 

    # keystone service-list
    +——————————————————————————————————+——————————+——————————+——————————————————————————-+
    |                id                |   name   |   type   |        description        |
    +——————————————————————————————————+——————————+——————————+——————————————————————————-+
    | e19528200f9c489bbacb09251ba781dd |  glance  |  image   |    Glance Image Service   |
    | 59739c5ea41f480687c1077cbd6ca171 | keystone | identity | Keystone Identity Service |
    +——————————————————————————————————+——————————+——————————+——————————————————————————-+

<!--more-->

If this is the case, nova-api needs to be reconfigured. Nova-api is responsible to register the nova service into keystone. Keystone auth token will be required for this step. Once it’s successful, nova should be listed in keystone services: 

    # keystone service-list
    +——————————————————————————————————+——————————+——————————+——————————————————————————-+
    |                id                |   name   |   type   |        description        |
    +——————————————————————————————————+——————————+——————————+——————————————————————————-+
    | e19528200f9c489bbacb09251ba781dd |  glance  |  image   |    Glance Image Service   |
    | 59739c5ea41f480687c1077cbd6ca171 | keystone | identity | Keystone Identity Service |
    | 98a5df5fa0db464ba4442bc85a35c82f |   nova   | compute  |    Nova Compute Service   |
    +——————————————————————————————————+——————————+——————————+——————————————————————————-+

If this was the root cause, nova should now work properly: 

    # nova service-list
    +————————+——————+——————+————————+——————-+————————————+
    | Binary | Host | Zone | Status | State | Updated_at |
    +————————+——————+——————+————————+——————-+————————————+
    +————————+——————+——————+————————+——————-+————————————+
    

