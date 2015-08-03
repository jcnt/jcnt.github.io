---
layout: post
title:  "Stuck with QSM? App consistent DR is still possible!"
date:   2015-04-11 13:18:42+0100
categories: 
tags: 
- NetApp
- DR
- SnapMirror
---
With my 9to5 job (oh wait...) I spend a lot of time with customer discussions. There are many of them 
who are going to keep their older NetApp boxes when they purchase a new, for the obvious reason: still 
good enough for Disaster Recovery!

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

*Note: "snapmirror update" also accepts the "-s snapname" option, so it is possible to use a specific app
consistent Snapshot to initialize the partnership*




