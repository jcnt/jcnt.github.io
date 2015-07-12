---
layout: post
title:  "Getting Started with Cloud Manager for AWS"
date:   2015-07-12 13:28:42+0100
categories: 
tags: 
- AWS
- NetApp
- ONTAP
- Cloud
---
Why do you need Cloud Manager? This piece of software can be used to manage different ONTAP instances around 
public cloud services: 

- NetApp Private Storage for Public Cloud (AWS, Azure, SoftLayer, etc)
- Cloud ONTAP for AWS

![Cloud Arch](/assets/2015-07-12-getting-started-cloud-manager/cloud-arch.png)

Cloud Manager is pretty easy to setup, in fact, in this article I'm going to show the easiest option to consume
it. Gererally speaking there are two ways to get Cloud Manager: 

- Download from the NetApp Support site
- Deploy from Amazon Marketspace 

The second option is as easy as launching an instance with a few clicks,  Cloud Manager is available on the 
AWS Marketspace. Please note, however this sofware is free, it needs a t2.medium or m3.medium type of instance
to run (first is cheaper, $0.076 per Hour in EU Ireland + 0.11 per GB per Month for EBS). Detailed information 
is [here][cloud-manager-instance]

With the deployment option there's no real "setup" of the Cloud Manager software. After the instance has been
initialized, it is available through the web interface. The only thing we need to do is to open a browser and
point it to "localhost". 

![Cloud Arch](/assets/2015-07-12-getting-started-cloud-manager/setup1.png)

After defining a couple of optional parameters like HTTP proxy and Site/Company details we need to create an 
admin user. This will be the method to authenticate to Cloud Manager later.

![Create Admin](/assets/2015-07-12-getting-started-cloud-manager/create-admin.png)

Additionally, we can define our AWS credentials including Cost S3 Budget and we can create our first tenant. Creating
multiple tenants will help to isolate organizations from each other and also we can use cost centers which can 
be integrated with OnCommand Insight for chargeback for example. 

![NSS Details](/assets/2015-07-12-getting-started-cloud-manager/nss-details.png)

All the products managed by Cloud Manager are fully supported by NetApp Global Support (Cloud ONTAP, NetApp Private
Storage). At the NSS page we can set up our support portal credentials. 

![No Working](/assets/2015-07-12-getting-started-cloud-manager/no-working.png)

Once we finished the setup wizard, we'll find the dashboard. This is the same default page which will load next time
we log into Cloud Manager. As we just finished the setup, there are no working instances to manage. We can use the 
"Add environment" option to connect to an existing running Cloud ONTAP, NetApp Private Storage, or to create a new
instance. 

As said, setting up Cloud Manager is very easy. It's not more than a few clicks and it's ready to use. We'll walk 
through how to create a new instance in an upcoming post. 


[cloud-manager-instance]: https://aws.amazon.com/marketplace/pp/B00OMA42XU?ref=cns_srchrow

<!--more-->

