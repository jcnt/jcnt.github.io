---
layout: post
title:  "Getting Started with Cloud ONTAP for AWS"
date:   2015-08-02 09:28:42+0100
categories: 
tags: 
- AWS
- NetApp
- ONTAP
- Cloud
---
The previous [article](/2015/07/12/getting-started-cloud-manager.html){:target="_blank"} described Cloud Manager 
and how to get started using it. Cloud Manager is not a standalone software, it is used to manage NetApp Private 
Storage for Public Cloud (NPS) instances or Cloud ONTAP which is only available for Amazon Web Services (AWS) this 
time. This artcile will focus on the latter, Cloud ONTAP, how to create the first instance with the first volume 
in a few minutes. 

![No Working](/assets/2015-07-12-getting-started-cloud-manager/no-working.png)

After starting the Cloud Manager instance, this is the status we finished last time. Cloud Manager instance is 
up and running (on a Windows operating system), but it has no working environments configured yet. 

<!--more-->

![Add environment](/assets/2015-08-02-getting-started-cloud-ontap/setup1.png)


After using the "Add environment" button on the main screen you have two options to select:

- New Cloud ONTAP
- New NetApp Private Storage

Let's select the Cloud ONTAP and see what will happen. 

![Name](/assets/2015-08-02-getting-started-cloud-ontap/setup2.png)

First, you need to give our environment a name. 

![Location](/assets/2015-08-02-getting-started-cloud-ontap/setup3.png)

Then you need to select the location of our new Cloud ONTAP deployment. Location is not only the AWS region (EU Ireland
in my case) but also within the region you need to select the VPC (Amazon Virtual Private Cloud) and the Subnet too. 

Virtual Private Cloud feature within AWS lets you create multiple isolated environments, each with multiple subnets and 
IP ranges. More information on VPC [here.](http://aws.amazon.com/vpc/){:target="_blank"}

Next up, the tough part, figuring out which Cloud ONTAP licensing to use (...and which AWS instance type): 

![Name](/assets/2015-08-02-getting-started-cloud-ontap/setup4.png)

So let's look into the details of these three. All the costs I mention below are valid at the time of the article and 
using EU Ireland to deploy Cloud ONTAP. 

- Explore (hourly): This is the one getting your hands dirty and trying out what it is. Capacity is limited to 2TB EBS RAW
(i.e. you can store more using ONTAP deduplication and compression). It only supports the m3.xlarge instance, software and
infrastructure cost is about $1.543 per hour. This means we pay hourly fees for both NetApp Software and AWS. Detailed page
with all the required information is available on [AWS Marketplace](https://aws.amazon.com/marketplace/pp/B00OMA456E?ref=cns_srchrow){:target="_blank"}

- Standard (hourly): Similar to Explore, but it's a more flexible option. It provides more capacity, up to 10TB underlying 
EBS RAW. More importantly it gives choices on performance. We can select from four different instance types (m3.xlarge, 
m3.2xlarge, r3.xlarge, r3.2xlarge). From a pricing point of view, NetApp Software license will cost exactly the same for all 
of these options, but of course AWS price will be different. The least expensive option is m3.xlarge where the combined cost is
$1.943 per hour. Details [here](https://aws.amazon.com/marketplace/pp/B00OMA48DO?ref=cns_srchrow){:target="_blank"}

- Standard (BYOL): Bring Your Own Licence gives us the option to not pay the Software cost hourly, but rather get it from standard
NetApp channel. The hourly costs for AWS are the same as Standard (hourly). Details [here](https://aws.amazon.com/marketplace/pp/B00OMA46T0/ref=cns_srchrow){:target="_blank"}

It's also important to understand, these costs are the hourly ones for both the AWS infrastucture and for the Software (except 
for BYOL), additionally you need to calculate the monthly cost what you need to pay for the actual capacity (underlying EBS RAW) 
you use. In all cases that's going to be $0.10 per GB per month. 

![admin](/assets/2015-08-02-getting-started-cloud-ontap/setup5.png)

On the next page you need to specify the admin password for the Cloud ONTAP system. It's basically the admin password for Data 
ONTAP, you will use it to authenticate in the CLI, GUI or over Cloud Manager. Also, Cloud ONTAP needs a support instance, which 
is an individual instance running in the same VPC. For this, you need to specify our authentication key pair to log in. Also, 
you can select here to use EBS encryption for our volumes, which is generally a good idea in Public Cloud environments. 

![vol](/assets/2015-08-02-getting-started-cloud-ontap/setup6.png)

There we go! This is the last step you need to do. At the end of the day, it's a storage appliance, so let's provision capacity!
Pretty easy steps: 

1. name the volume 
2. select the protocol, NFS, SMB or iSCSI - no Fibre Channel in the Cloud, hey!
3. size it
4. set access control, based on the selected protocol
5. usage profile, this is about the performance vs. efficiency game
6. protection: Snapshots in the Cloud! Easy and quick recovery when it is needed!

![set](/assets/2015-08-02-getting-started-cloud-ontap/setup7.png)

We are all set! Cloud ONTAP is up and running, one volume provisioned over NFS for all the instances in my subnet. 
Now how does it look like from one of the instances? 

    ubuntu@ip-172-31-36-91:~$ sudo mount 172.31.4.3:/testvol /mnt
    ubuntu@ip-172-31-36-91:~$ 
    ubuntu@ip-172-31-36-91:~$ df
    Filesystem          1K-blocks   Used Available Use% Mounted on
    /dev/xvda1            8115168 801132   6878760  11% /
    none                        4      0         4   0% /sys/fs/cgroup
    udev                   503188     12    503176   1% /dev
    tmpfs                  101632    340    101292   1% /run
    none                     5120      0      5120   0% /run/lock
    none                   508144      0    508144   0% /run/shm
    none                   102400      0    102400   0% /run/user
    172.31.4.3:/testvol   9961472    192   9961280   1% /mnt
    ubuntu@ip-172-31-36-91:~$ 
    ubuntu@ip-172-31-36-91:~$ dd if=/dev/zero of=/mnt/file bs=8K count=1024
    1024+0 records in
    1024+0 records out
    8388608 bytes (8.4 MB) copied, 0.2634 s, 31.8 MB/s
    ubuntu@ip-172-31-36-91:~$

I know, I know, dd is a horrible test, but good enough to see we can read and write the volume. 

At this point I would like to mention two additional things. Looking at the Cloud ONTAP instance page, there is a menu on 
the right, you can change the instance type here. 

![system manager](/assets/2015-08-02-getting-started-cloud-ontap/setup8.png)

Using this menu, you can access the advanced part, which hides System Manager. Opening it up you can manage Cloud ONTAP 
the same way as you manage any of your ONTAPs around!

![system manager](/assets/2015-08-02-getting-started-cloud-ontap/setup9.png)

That is all! Probably it takes more time to read this article than setting up a Cloud ONTAP instance with its first volume. 

