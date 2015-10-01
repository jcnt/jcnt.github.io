---
layout: post
title:  "Stuck with QSM? Part two: sync back”
date:   2015-09-28 14:45:42+0100
categories: 
tags: 
- NetApp
- DR
- SnapMirror
---
In the previous [article](/2015/07/12/getting-started-cloud-manager.html){:target="_blank"} I demonstrated 
what to do when the primary and the secondary systems run very different Data ONTAP versions thus 
Volume SnapMirror is not available, but still, app consistency is a requirement. This article is focusing 
on the next required steps, showing how to sync back after running the application on the primary side, 
which means modifying the primary copy. 


With my 9to5 job (oh wait...) I spend a lot of time with customer discussions. There are many of them 
who are going to keep their older NetApp boxes when they purchase a new, for the obvious reason: still 
good enough for Disaster Recovery!

In the first article I used the -s option of SnapMirror to specify a SnapShot which is going to be replicated
to the secondary side. That SnapShot can be created by any kind of external software (SnapManagers, 
SnapCreator, scripting, etc…). 

As the replication in this case is done by Qtree SnapMirror you need to keep a few things in mind: 




**DR copy testing**



The common challenge they may have is when the older box is too old to run recent Data ONTAP versions. 
In these cases standard Volume Snapmirror is not really an option. Qtree SnapMirror could work well, 
but it is challenging to replicate app consistent data. If there are Snapshots created with software suites
like the SnapManagers, SnapCreator, or even a single script, those should be replicated in a consistent way. 

<!--more-->

**1./ First of all, creating Qtrees are NOT necessary!**

Volumes are Qtrees at the end of the day. The only difference is the way the SnapMirror relationship is 
created. 

	dest> snapmirror initialize -S source:/vol/source/- dest:/vol/dest/qtree

The dash (-) character indicates all non-Qtree data in the specified Volume. Basically in this case 
SnapMirror will replicate all data in /vol/source/ which is NOT in a Qtree.

**2./ Disable automatic SnapMirror updates**

In a Qtree SnapMirror the transfer is Qtree level. Snapshots are Volume level, so transferring a Qtree 
will not transfer the Snapshots created by any app (for consistency). On every update Qtree SnapMirror
will create a new Snapshot and will transfer that particular one, SnapManager/SnapCreator generated 
Snapshots are not included. 

**3./ Select the app consistent Snapshot to transfer**

By now, probably the SnapManager or SnapCreator is already up and creating its application consistent
Snapshots on the system. Great! SnapMirror allows to do a transfer using a specific Snapshot. 

	dest> snapmirror update -S source:/vol/source/- -s snapname dest:/vol/dest/qtree

This will update the DR with the selected app consistent Snapshot "snapname", thus the DR is app consistent. 

*Note: "snapmirror initialize" also accepts the "-s snapname" option, so it is possible to use a specific app
consistent Snapshot to initialize the partnership*




