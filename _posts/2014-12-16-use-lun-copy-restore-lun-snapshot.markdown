---
layout: post
title:  "HOWTO: use LUN copy to restore a LUN from a Snapshot"
date:   2014-12-16 13:05:00
categories: 
- tech
tags: 
- NetApp
- ONTAP
- cDOT
---
With Clustered ONTAP 8.3 there’s a new feature for block environments: LUN copy/move. This feature gives a few more options to your day to day storage management: 

* copy or move LUNs between volumes (containers) anywhere within your SVM (tenant, Storage Virtual Machine)
* copy LUNs between volumes BETWEEN SVMs 

<br>
This feature also can be used to restore an older version of the LUN, sitting inside an earlier Snapshot copy. I will show this functionality in an example below. 

Let’s see first what kind of Snapshots I have, with limiting the output for a specific tenant and volume: 

    cdot::> snapshot list -vserver vserver02_hyperv -volume vserver02_LUNs 
    
                                                                     ---Blocks---
    Vserver  Volume   Snapshot                                  Size Total% Used%
    -------- -------- ------------------------------------- -------- ------ -----
    vserver02_hyperv 
             vserver02_LUNs
                      weekly.2014-12-14_0015                 44.92GB     1%    7%
                      daily.2014-12-15_0010                  41.88GB     1%    7%
                      hourly.2014-12-16_0005                 28.74GB     1%    5%
                      daily.2014-12-16_0010                  46.80GB     2%    7%
                      hourly.2014-12-16_0105                 851.9MB     0%    0%
                      hourly.2014-12-16_0205                  1.21GB     0%    0%
                      hourly.2014-12-16_0305                 634.6MB     0%    0%
                      hourly.2014-12-16_0405                  1.17GB     0%    0%
                      hourly.2014-12-16_0505                  1.03GB     0%    0%
    9 entries were displayed.

let’s see the LUNs in the same tenant for MSHOST1: 

    cdot::> lun show -vserver vserver02_hyperv *MSHOST1*
    Vserver   Path                            State   Mapped   Type        Size
    --------- ------------------------------- ------- -------- -------- --------
    vserver02_hyperv 
              /vol/vserver02_LUNs/MSHOST1_boot 
                                              online  mapped   windows   120.0GB

Now let’s start the copy process: 

    cdot::> lun copy start -vserver vserver02_hyperv -destination-path /vol/vserver02_LUNs/MSHOST1_boot_restore -source-path /vol/vserver02_LUNs/.snapshot/weekly.2014-12-14_0015/MSHOST1_boot 
    
    Following LUN copies have been started:                                                                                                   
    
    "vserver02_hyperv:/vol/vserver02_LUNs/.snapshot/weekly.2014-12-14_0015/MSHOST1_boot" copy to "vserver02_hyperv:/vol/vserver02_LUNs/MSHOST1_boot_restore"
    
You can monitor the process: 

    cdot::> lun copy show 
    Vserver   Destination Path                Status          Progress
    --------- ----------- ------------------- --------------- --------
    vserver02_hyperv 
              /vol/vserver02_LUNs/MSHOST1_boot_restore 
                                              Allocation-Map  81%

Alternatively you can use the clone create command with the same parameters: 

    cdot::> clone create -vserver vserver02_hyperv -destination-path /vol/vserver02_LUNs/MSHOST1_boot_restore2 -source-path /vol/vserver02_LUNs/.snapshot/daily.2014-12-15_0010/MSHOST1_boot

Now, the difference between these methods is the first one really copies all the data from the first LUN to the second one, where cloning does not move any data. 


<!--more-->

