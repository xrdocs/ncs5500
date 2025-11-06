---
published: false
date: '2025-11-05 15:52 +0530'
title: Understanding Stats Resource Reassignment & Enhancements in 25.4.1
---
## A New Post

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.

Introduction :
A hardware counter engine in Cisco NCS 540 Large’s ASIC is a dedicated hardware component responsible for managing and organizing hardware counters. These counters track various statistics, such as packet and byte counts, directly within the router Interfaces. The hardware counter engine enables flexible and efficient allocation, reassignment, and management of these counters to different features or functions as needed. This optimization ensures accurate and efficient statistics collection while minimizing impact on the router’s forwarding performance. In this discussion we will see how the NCS540’s  NPU hardware counters are organised and utilized. we will also discussion how we can have these counter resources reassigned based on the scale requirements for other features. 

Default Stats Allocations :

To check the default statistics for the NPU resources, we have to go the router cli and post, 
#sh controllers npu resources stats internal instance all location all

RP/0/RP0/CPU0:Arches#sh controllers npu resources stats internal instance all $
Mon Sep 22 09:57:46.561 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
=================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
lpts-pbr                4     In use   8K     4096            53               
aclrx-mcrouterx         5     Free     8K     8192            0                
acltx                   6     Free     8K     8192            0                
qos rx                  9     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                

The table consist of 6 columns , showing the feature , its engine Id , size allotted for the specific engine, and total counter entries available and in use. Remember , Total counter entries depends on the counter set each feature consumes. We will know more about counter sets in the following discussion.


Below output table shows details of each of the feature : 

aclrx-mcrouterx	Specifies the Ingress ACL Rx and McRoute feature option
acltx			Specifies the Egress ACL Tx feature option
l2rx			Specifies the Ingress L2 Rx Interface feature option
l2tx-mctx		Specifies the Egress L2 Tx, BVI, and McTx Interface feature option
l3rx			Specifies the Ingress L3 Rx and BVI Interface feature option
l3tx			Specifies the Egress L3 Tx Interface feature option
lpts-pbr		Specifies the LPTS and PBR feature option
mplsrx			Specifies the SR Rx feature option
policer			Specifies the Ingress QoS feature option
tetx			Specifies the MPLS Te Tx feature option
trap			Specifies the trap feature option
voq				Specifies the VoQ feature option


Below table shows the Re-ordered Arrangement according to increasing Engine size:


Feature                Eng    State     Eng   Total Counter   Core-0 Entries
                       Id               Size  Entries         InUse
============================================================================
trap-aclpolicer         0     Free       4K    2048           0
tetx-srv6loc            1     Free       4K    4096           0
l3tx                    2     Free       4K    4096           0
l3tx2                   3     In use     4K    2048           34
lpts-pbr                4     In use     8K    4096           53
aclrx-mcrouterx         5     Free       8K    8192           0
acltx                   6     Free       8K    8192           0
l2rx                    7     Free       8K    8192           0
voq                     8     In use     8K    4096           51
qos rx                  9     Free       8K    4096           0
l2tx-mctx              10     Free      16K    16384          0
l3rx                   11     Free      16K    16384          0
POLICER1 (CHILD)       12     Reserved  16K    16384          0
POLICER2 (PARENT)      13     Reserved  16K    4096           0


We have a total of 14 counter engines, shared in 4k ,8k and 16k capacity as follows, 

Engine ID	Capacity
0 - 3		  4K
4 - 9		  8K
10 - 13		 16K	

Each feature is allocated with variable amount of counter engines and each of them consumes different counter sets form the counter engine , The below table explains how many counters each feature has and default counter set that it consumes. 



Note : Counter sets are shared and balanced across the feature depending on the scale consumption.
Here scale is calculated based on Engine size / counter set. 

To reassign un-used counter engines of one feature with counter engine of other features, we use the stats-resource reassign command in XR Config mode. In the coming discussion we will see , how we can perform the stats resource re-assignment.  

Note : Before reassigning the NPU engine counters, we must make sure that they are unused(FREE State).
Feature                Eng    State     Eng   Total Counter   Core-0 Entries
                       Id               Size  Entries         InUse
============================================================================
trap-aclpolicer         0     Free       4K    2048           0
tetx-srv6loc            1     Free       4K    4096           0
l3tx                    2     Free       4K    4096           0
l3tx2                   3     In use     4K    2048           34
lpts-pbr                4     In use     8K    4096           53
aclrx-mcrouterx         5     Free       8K    8192           0
acltx                   6     Free       8K    8192           0
l2rx                    7     Free       8K    8192           0
voq                     8     In use     8K    4096           51
qos rx                  9     Free       8K    4096           0
l2tx-mctx              10     Free      16K    16384          0
l3rx                   11     Free      16K    16384          0
POLICER1 (CHILD)       12     Reserved  16K    16384          0
POLICER2 (PARENT)      13     Reserved  16K    4096           0
 













Stats Resource Re-assignment 

In this chapter we will look at how stats re-assignment works and what are the new enhancements we have got in the latest release for the NCS540 Large Series of Routers.

We have divided the following discussion into two sections, 

Section 1 : Learning Current Resource Re-Assignment and its Limitation
Section 2 : Resource Re-Assignment with Latest release

Section 1 ; Before updating the Router: Learning Current Resource Re-Assignment and its Limitation

Device Info:

RP/0/RP0/CPU0:ios#sh platform 
Tue Oct  7 06:38:06.268 UTC
Node              Type                     State                    Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        N540-24Q8L2DD-SYS(Active) IOS XR RUN               NSHUT
0/FT0             N540-FAN                 OPERATIONAL              NSHUT
RP/0/RP0/CPU0:ios#sh version 
Tue Oct  7 06:38:08.224 UTC
Cisco IOS XR Software, Version 24.3.1.25I LNT
Copyright (c) 2013-2024 by Cisco Systems, Inc.

Build Information:
 Built By     : swtools
 Built On     : Sat Jun 15 22:17:28 UTC 2024
 Build Host   : iox-ucs-054
 Workspace    : /auto/ioxdepot6/weekly_giso/giso_swtools_24_3_1_25I_SIT_IMAGE_ncs540l
 Version      : 24.3.1.25I
 Label        : 24.3.1.25I-PROD_BUILD_24_3_1_25I_SIT_IMAGE

cisco NCS540L (D1519 @ 1.50GHz)
cisco N540-24Q8L2DD-SYS (D1519 @ 1.50GHz) processor with 16GB of memory
ios uptime is 6 days, 21 hours, 26 minutes
Cisco NCS540 Series, Fixed Router 2x400G, 8x50G, 24x25G Chassis



Note : Each policer is accompanied by policer statistics, which currently presents a limitation due to restricted stats availability.










To Reiterate Default Counter Processor Configuration are as follows:

RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Tue Sep 30 09:15:50.862 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
lpts-pbr                4     In use   8K     4096            53               
aclrx-mcrouterx         5     Free     8K     8192            0                
acltx                   6     Free     8K     8192            0                
qos rx                  9     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Note: Free and Unassigned Counter engines can be reassigned to other features

Below Table shows Banks used by specific features:

Note :- In the below field ,  Rx represents Ingress direction & Tx represents Egress direction. 

Feature	Banks
LPTS-PBR	lpts-pbr                
TRAP-ACL	trap-aclpolicer
ACL	acltx
QoS	qosrx
L3 Interface/Subinterface 	l3rx,l3tx
L2 Interface/Subinterface	l2rx, l2tx
Tunnels,SRv6	tetx-srv6loc


Let us now try a re-assignment.

Config : 

stats-resource reassign location 0/RP0/CPU0
 internal feature trap eng-ids 6
!
End


Before re-assignment, we need to ensure that the engine state is “Free”  for the feature from which stats are being re-assigned. The CLI command above will assign engine ID 6 to the trap-aclpolicer feature, which was previously used by acltx. This will increase the trap-aclpolicer feature's stats engine size from 4k to 12k, as we are reassigning 8k from acltx.



Thus, with the new re-assignment the Counter Processor Configuration Profile updates as follows,

RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Tue Sep 30 09:21:10.787 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
trap-aclpolicer         6     Free     8K     4096            0                
lpts-pbr                4     In use   8K     4096            53               
aclrx-mcrouterx         5     Free     8K     8192            0                
qos rx                  9     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
==================================================================================

Before - After View : Comparing the Output


Before: 

Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
lpts-pbr                4     In use   8K     4096            53               
aclrx-mcrouterx         5     Free     8K     8192            0                
acltx                   6     Free     8K     8192            0                
qos rx                  9     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                


After , 

Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
trap-aclpolicer         6     Free     8K     4096            0
trap-aclpolicer         6     Free     8K     4096            0                                
lpts-pbr                4     In use   8K     4096            53               
aclrx-mcrouterx         5     Free     8K     8192            0                
qos rx                  9     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                

Below is another example of resource reassignment. Do follow this example, this leads to the full understanding of the reassignment. 


!! IOS XR Configuration 24.3.1.25I
stats-resource reassign location 0/RP0/CPU0
 internal feature voq eng-ids 5
!
end

RP/0/RP0/CPU0:ios(config)#commit 
Tue Sep 30 10:33:10.918 UTC
RP/0/RP0/CPU0:Sep 30 10:33:10.966 UTC: fia_driver[352]: %FABRIC-STATS_INFRA-4-CLI_CONFIG_WARNING : Warning: Please ensure Total VOQ Stats Resources are symmetric across all the linecards to avoid inconsistent stat behaviour. 

RP/0/RP0/CPU0:ios(config)#     

RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Tue Sep 30 10:33:44.363 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
trap-aclpolicer         6     Free     8K     4096            0                
trap-aclpolicer         6     Free     8K     4096            0                
lpts-pbr                4     In use   8K     4096            53               
qos rx                  9     Free     8K     4096            0                
voq                     5     Free     8K     4096            0                
voq                     5     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Note: Free and Unassigned Counter engines can be reassigned to other features
====================================================================================

Note : REPETED ENTRY TELLS US THAT there is a RE_ASSIGNED ENGINE: i.e that engine id was previously assigned to some other feature. This does not mean that we have 16 Engine size for voq feature, its 8k which is taken form aclrx-mcrouterx and given to voq.      


Understanding the re-assignment mechanism further: 

Consider the below example. 
RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature voq eng-ids 1
RP/0/RP0/CPU0:ios(config)#commit 
Tue Sep 30 10:47:34.928 UTC
RP/0/RP0/CPU0:Sep 30 10:47:34.974 UTC: fia_driver[352]: %FABRIC-STATS_INFRA-4-CLI_CONFIG_WARNING : Warning: Please ensure Total VOQ Stats Resources are symmetric across all the linecards to avoid inconsistent stat behaviour. 
RP/0/RP0/CPU0:ios(config)#end


Here, we have successfully re-assigned the voq feature with engine id 1, which was previously utilized by the “tetx-srv6loc” feature. As a result, when we assign one more stats engine to voq, the previous assignment becomes null and void, and the engine is moved to the Empty Bank. This means we have previously assigned Engine 5 will move to Free Pool, as shown below. i.e This bank will not revert to the default feature it was assigned to, which in this case was “aclrx-mcrouterx”. 
Below we can see Engine ID 5 in the free pool.    
RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Tue Sep 30 10:47:43.692 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
trap-aclpolicer         6     Free     8K     4096            0                
trap-aclpolicer         6     Free     8K     4096            0                
lpts-pbr                4     In use   8K     4096            53               
qos rx                  9     Free     8K     4096            0                
voq                     1     Free     4K     2048            0                
voq                     1     Free     4K     2048            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Free and Unassigned Counter Engine Resource Information
====================================================================================
Eng   Eng   
Id    Size  
====================================================================================
5       8K    
====================================================================================
Note: Free and Unassigned Counter engines can be reassigned to other features
====================================================================================



Now lets , Try to re-assign both engine 5 and engine 1 to VOQ , by vacating all the engines assigned to VOQ and then reassign them.


RP/0/RP0/CPU0:ios(config)#no stats-resource reassign location 0/RP0/CPU0 internal feature voq 
RP/0/RP0/CPU0:ios(config)#commit 
Tue Sep 30 10:52:56.660 UTC
RP/0/RP0/CPU0:Sep 30 10:52:56.705 UTC: fia_driver[352]: %FABRIC-STATS_INFRA-4-CLI_CONFIG_WARNING : Warning: Please ensure Total VOQ Stats Resources are symmetric across all the linecards to avoid inconsistent stat behaviour. 
RP/0/RP0/CPU0:ios(config)#end

As a result, we see: 


RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Tue Sep 30 10:53:02.367 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
trap-aclpolicer         6     Free     8K     4096            0                
trap-aclpolicer         6     Free     8K     4096            0                
lpts-pbr                4     In use   8K     4096            53               
qos rx                  9     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Free and Unassigned Counter Engine Resource Information
====================================================================================
Eng   Eng   
Id    Size  
====================================================================================
1       4K    
5       8K    
====================================================================================
Note: Free and Unassigned Counter engines can be reassigned to other features
====================================================================================


Now , lets try to Reassignment at once to the voq and see if it is successful:

RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature voq eng-ids 1 5
RP/0/RP0/CPU0:ios(config)#commit 
Tue Sep 30 10:54:22.817 UTC
RP/0/RP0/CPU0:Sep 30 10:54:22.862 UTC: fia_driver[352]: %FABRIC-STATS_INFRA-4-CLI_CONFIG_WARNING : Warning: Please ensure Total VOQ Stats Resources are symmetric across all the linecards to avoid inconsistent stat behaviour. 
RP/0/RP0/CPU0:ios(config)#end 


As we can see here, both engine 5 and 1 are assigned to VOQ as required. This is how multiple banks can be assigned to a single feature. Below we can see the new re-assignments as configured.

RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Tue Sep 30 10:54:31.381 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
trap-aclpolicer         6     Free     8K     4096            0                
trap-aclpolicer         6     Free     8K     4096            0                
lpts-pbr                4     In use   8K     4096            53               
qos rx                  9     Free     8K     4096            0                
voq                     1     Free     4K     2048            0                
voq                     1     Free     4K     2048            0                
voq                     5     Free     8K     4096            0                
voq                     5     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Note: Free and Unassigned Counter engines can be reassigned to other features
====================================================================================

NOTE : Remember, dual entry means, the specific features has Engines allocated from other features (only Eng 8 was default, rest all are reassigned, refer the default/1st Table in this discussion for reference). 

Now, let’s Try one more reassignment of QOSRX [Which is ‘not’ allowed by design in this release]:

RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature qosrx eng-ids 10
RP/0/RP0/CPU0:ios(config)#commit 
Tue Sep 30 11:05:28.890 UTC
RP/0/RP0/CPU0:Sep 30 11:05:29.949 UTC: config[65891]: %MGBL-CONFIGCLI-3-COMMIT_FAILURE : Configuration commit running under 'con0_RP0_CPU0': by :'lab' failed,commit results stored in '/cfs/cfg/lr/failed/seamless/1000000038_failed.cfg' 

% Failed to commit one or more configuration items during a pseudo-atomic operation. All changes made have been reverted. Please issue 'show configuration failed [inheritance]' from this session to view the errors
RP/0/RP0/CPU0:ios(config)#show configuration failed inheritance 
Tue Sep 30 11:05:34.820 UTC
!! SEMANTIC ERRORS: This configuration was rejected by 
!! the system due to semantic errors. The individual 
!! errors with each failed configuration command can be 
!! found below.


stats-resource reassign location 0/RP0/CPU0
 internal feature qosrx eng-ids 10
!!% A SysDB client requested a function that the server or EDM does not currently support: Unsupported CLI input. Qos Rx Stat and Policer Engines are reserved.
!
End

Note : As Expected, the commit fails for this release with error “Qos Rx Stat and Policer Engines are reserved”
To Free Up all the Engines that are allowed to be freed for re-assignment by design (i.e except QOSRX [eng_id 9], ACL Policer [eng_id 0] and Policer 1 and 2 [12 & 13]) in the resource Banks we must follow the below steps:

Steps to Free Engines:
 
	Step 1: Re- Assign the Free Engines to one of the other features, eg. like VOQ(which we have already done in the examples so far).
RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature voq eng-ids 1 2 5 6 7 10 11
RP/0/RP0/CPU0:ios(config)#commit
Note : the above reassignment has to be done when voq feature is not assigned with any other feature bank.

	Step 2: Negate the assignment, this will put the Banks into free pool. 

  RP/0/RP0/CPU0:ios(config)#no stats-resource reassign location 0/RP0/CPU0 internal feature voq 
  RP/0/RP0/CPU0:ios(config)#commit 

And we can see below that removing voq engine sends all the allotted engines to free bank.

RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Tue Sep 30 12:35:22.059 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     Free     4K     2048            0                
lpts-pbr                4     In use   8K     4096            53               
qos rx                  9     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3tx2                   3     In use   4K     2048            34               
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Free and Unassigned Counter Engine Resource Information
====================================================================================
Eng   Eng   
Id    Size  
====================================================================================
1       4K    
2       4K    
5       8K    
6       8K    
7       8K    
10      16K   
11      16K   
Thus, we have moved all the free state engines to the free pool.
Note: Engines with State as “In use” also cannot be reallocated. 

This completes Section 1 of the discussion.
 

Section 2 ; After updating the Router: [Enhancement Done : Policer scale increase & Sub-interface Scale increase , with qos stats unlock ]

With the IOS XR release 25.3.1 we have removed the lock, which previously doesn’t allow the qos rx and acl policer engines to be re-assigned. Let us now look at how the removal of this lock helps us in scaling better with the NCS540 Large.


RP/0/RP0/CPU0:ios#sh platform 
Tue Oct  7 10:04:15.006 UTC
Node              Type                     State                    Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        N540-24Q8L2DD-SYS(Active) IOS XR RUN               NSHUT
0/FT0             N540-FAN                 OPERATIONAL              NSHUT

RP/0/RP0/CPU0:ios#sh version 
Tue Oct  7 10:04:16.627 UTC
Cisco IOS XR Software, Version 25.3.1.31I LNT
Copyright (c) 2013-2025 by Cisco Systems, Inc.

Build Information:
 Built By     : cisco
 Built On     : Sun Jul 13 21:48:21 UTC 2025
 Build Host   : iox-lnx-016
 Workspace    : /auto/iox-lnx-016-san1/prod/25.3.1.31I.SIT_IMAGE/ncs540l/ws/
 Version      : 25.3.1.31I
 Label        : 25.3.1.31I

cisco NCS540L (D1519 @ 1.50GHz)
cisco N540-24Q8L2DD-SYS (D1519 @ 1.50GHz) processor with 16GB of memory
ios uptime is 1 hour, 36 minutes
Cisco NCS540 Series, Fixed Router 2x400G, 8x50G, 24x25G Chassis


Policer Scale [Policer Scale has been increased in the newer release form 8k to 16k] 
New [25.3.1.31I]
QoS PP Policer Banks Information For Location: 0/RP0/CPU0

System information for NPU 0:
Core 0:
                                Used(   %)       Free      Total       Class-map size
      Policer Banks info:          0(  0%)      16000      16000                    1




Old [24.3.1.25I]
QoS PP Policer Banks Information For Location: 0/RP0/CPU0

System information for NPU 0:
Core 0:
                                Used(   %)       Free      Total       Class-map size
      Policer Banks info:          0(  0%)       4096       4096                    2




Stats Resources by sub-interfaces & policer scale


Let us now understand the Relevance of sub-interface and Policer scale together. 

Note: Policer and l2/l3 sub-interface scale depends of MDB profile used. 

For Q2A(Arches/SP) in older release:

Sub-interface Scale	Total Policer Scale	Release 
L3Max	L2Max	L3Max	L2Max	
4k	8k	8k	8k	24.3.1.25I
4k	8k	16k	16k	25.3.1.31I


Note: Policer Scale is a function of policy maps and Class-maps per policy map, 	
Total Policer Scale = Configured Policy Maps x Configured Class-maps per policy 

 J2/Q2A based NCS540 has hw-module profile defined to set variable scale of policer based on the class map selected, variable class map can be configured as follows. 

	
RP/0/RP0/CPU0:ios(config)#hw-module profile qos max-classmap-size ?
  1   Max 1 class-maps per policy
  2   Max 2 class-maps per policy
  3   Max 4 class-maps per policy
  8   Max 8 class-maps per policy
  16  Max 16 class-maps per policy
  32  Max 32 class-maps per policy
	To check the current size selected either check the hw-module profiles active or use the below command method.

The below command gives the details of the entire qos resource output,

RP/0/RP0/CPU0:ios#sh controllers npu resources qos all instance all location all 

Let’s take a section from this output to verify, 

RP/0/RP0/CPU0:SP2#sh controllers npu resources qos all instance all location all | begin QoS PP Policer Banks Information For Location: 0/RP0/CPU0 
Wed Oct  8 07:53:01.846 UTC
QoS PP Policer Banks Information For Location: 0/RP0/CPU0

System information for NPU 0:
Core 0:
                                Used(   %)       Free      Total       Class-map size
      Policer Banks info:          2(  0%)        254        256                   32


As we can see here , Total of 256 PM can be configured with CM of 32, 

         i.e Total Policer Scale = 256 x 32 
				        		           = 8192


Adding to this further, we have a total l2 Sub-interface Scale of 4096,now  to accommodate all the 4096-sub interface with different policers we have to make sure that the total sub interface scale matches the total policer scale of the box , and for that we have to make sure that the class map selection is accordingly. Thus, for the above example. For 4k sub int, we need 4k policer (1 policer per sub interface), thus for 4k policer, Class-maps per policy must be 2.

CM per policer {MAX} = 2



Now, how much QOS Counters [qos rx] will be Consumed?

All the sub interfaces will need qos rx to monitor the interface stats each sub - interface consume 2 Counter set consumed 1 for matched and 1 for dropped traffic from the qos rx Bank.

Case 1: Customer need -> l2Subintf = 4k Policers = 8k CM = 2 

qos rx counter Consumption: 8k needed.
 
Thus, 

Counter Processor Configuration Profile: Default
===========================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                    Eng    State   Eng   Total Counter    Core-0 Entries
                           Id             Size    Entries          InUse       
===========================================================================
Qos rx    8 Free 8K 4096  0          - ===========================================================================


In the above case we have sufficient Counter Engines.

Case 2: l2Subintf = 16k Policer = 16k CM = 1 
 
We have MDB profiles which allows us to have increased sub interface scale across NCS54x/55xx/57xx Portfolio, 

In this case where we need 16k policer we don’t have enough qos rx counter Entries for 16k policer with class map size of 1.

Thus, we need to re-assign engines to the qos rx.[which was not allowed previous release]
With the Enhancement in 25.4.1, We need to have 16k Counter entries for qos rx.

Note : All the above examples we have used EFT 25.3.1 release for demo of this feature.





Let’s configure,

RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature qosrx eng-ids 5 
RP/0/RP0/CPU0:ios(config)#root
RP/0/RP0/CPU0:ios(config)#show
Wed Oct  8 10:24:47.664 UTC
!! Building configuration...
!! IOS XR Configuration 25.3.1.31I
stats-resource reassign location 0/RP0/CPU0
 internal feature qosrx eng-ids 5
!
end

RP/0/RP0/CPU0:ios(config)#commit 
Wed Oct  8 10:24:51.565 UTC
RP/0/RP0/CPU0:ios(config)#end

#Commit Successful [this capability was not present previously].

As now we can see, 
 
RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Wed Oct  8 10:25:01.879 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     In use   4K     2048            147              
lpts-pbr                4     In use   8K     4096            54               
acltx                   6     Free     8K     8192            0                
qos rx                  5     Free     8K     4096            0                
qos rx                  8     Free     8K     4096            0                
voq                     7     In use   8K     4096            51               
l3rx                    10    Free     16K    16384           0                
l2rx                    9     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               11    Free     16K    16384           0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Note: Free and Unassigned Counter engines can be reassigned to other features
====================================================================================

Thus we have 16 k Counter Entries for qos rx , which server the customer purpose.






Case 3: Freeing Multiple Larger Engines and Expanding Single Smaller Engine 






Note: Make sure you design how you want to swap before trying on the router as shown above. Here we are giving 8k each to qos rx from l2tx-mcrx and l3rx.

In this case we will see an example how we can expand engines to a single feature with smart reallocation.

Default Condition : 

RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all loc$
Mon Oct 27 07:01:06.053 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     In use   4K     2048            147              
lpts-pbr                4     In use   8K     4096            52               
aclrx-mcrouterx         5     Free     8K     8192            0                
acltx                   6     Free     8K     8192            0                
qos rx                  9     Free     8K     4096            0                
voq                     8     In use   8K     4096            51               
l3rx                    11    Free     16K    16384           0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Note: Free and Unassigned Counter engines can be reassigned to other features


Freeing up engine 9 & 11.

RP/0/RP0/CPU0:ios#conf t
Mon Oct 27 07:07:39.567 UTC
RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature qosrx eng-ids 9
RP/0/RP0/CPU0:ios(config)#commit 
Mon Oct 27 07:08:05.532 UTC
RP/0/RP0/CPU0:ios(config)#no stats-resource reassign location 0/RP0/CPU0 internal feature qosrx eng-ids 9
RP/0/RP0/CPU0:ios(config)#commit      

RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature l3rx eng-ids 11
RP/0/RP0/CPU0:ios(config)#commit 
Mon Oct 27 07:21:35.516 UTC
RP/0/RP0/CPU0:Oct 27 07:21:35.568 UTC: fia_driver[319]: %FABRIC-STATS_INFRA-4-CLI_CONFIG_L3RX_WARNING : This re-assignment will impact the scale of L3RX counters used for L3 main/subinterface/BVI RX accounting. 
RP/0/RP0/CPU0:ios(config)#no stats-resource reassign location 0/RP0/CPU0 internal feature l3rx
RP/0/RP0/CPU0:ios(config)#commit 
Mon Oct 27 07:21:50.978 UTC
RP/0/RP0/CPU0:Oct 27 07:21:51.029 UTC: fia_driver[319]: %FABRIC-STATS_INFRA-4-CLI_CONFIG_L3RX_WARNING : This re-assignment will impact the scale of L3RX counters used for L3 main/subinterface/BVI RX accounting. 
RP/0/RP0/CPU0:ios(config)#exit
RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Mon Oct 27 07:22:02.194 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     In use   4K     2048            147              
lpts-pbr                4     In use   8K     4096            52               
aclrx-mcrouterx         5     Free     8K     8192            0                
acltx                   6     Free     8K     8192            0                
voq                     8     In use   8K     4096            51               
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               10    Free     16K    16384           0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Free and Unassigned Counter Engine Resource Information
====================================================================================
Eng   Eng   
Id    Size  
====================================================================================
9       8K    
11      16K   
====================================================================================
Note: Free and Unassigned Counter engines can be reassigned to other features
====================================================================================

RP/0/RP0/CPU0:ios# 

Now assigning qosrx 9 and 11 and giving free banks to l2tx-mctx and l3rx.
RP/0/RP0/CPU0:ios#conf t
Mon Oct 27 07:28:29.076 UTC
RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature qosrx eng-ids 10 11
RP/0/RP0/CPU0:ios(config)#commit 
Mon Oct 27 07:28:43.279 UTC
RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature l2tx-mctx eng-ids 6
RP/0/RP0/CPU0:ios(config)#commit
Mon Oct 27 07:29:33.175 UTC
RP/0/RP0/CPU0:ios(config)#stats-resource reassign location 0/RP0/CPU0 internal feature l3rx eng-ids 9
RP/0/RP0/CPU0:ios(config)#commit
Mon Oct 27 07:29:41.986 UTC
RP/0/RP0/CPU0:Oct 27 07:29:42.038 UTC: fia_driver[319]: %FABRIC-STATS_INFRA-4-CLI_CONFIG_L3RX_WARNING : This re-assignment will impact the scale of L3RX counters used for L3 main/subinterface/BVI RX accounting. 
RP/0/RP0/CPU0:ios(config)#exit
RP/0/RP0/CPU0:ios#sh controllers npu resources stats internal instance all location all 
Mon Oct 27 07:29:49.340 UTC
HW Stats Information For Location: 0/RP0/CPU0
System information for NPU 0: 
Counter Processor Configuration Profile: Default
===========================================================================================
Assigned Counter Engine Resource Information
===========================================================================
Feature                Eng    State   Eng   Total Counter    Core-0 Entries
                       Id             Size    Entries          InUse       
===========================================================================
trap-aclpolicer         0     In use   4K     2048            147              
lpts-pbr                4     In use   8K     4096            52               
aclrx-mcrouterx         5     Free     8K     8192            0                
qos rx                  10    Free     16K    8192            0                
qos rx                  11    Free     16K    8192            0                
voq                     8     In use   8K     4096            51               
l3rx                    9     Free     8K     8192            0                
l2rx                    7     Free     8K     8192            0                
l3tx                    2     Free     4K     4096            0                
l3tx2                   3     In use   4K     2048            34               
l2tx-mctx               6     Free     8K     8192            0                
tetx-srv6loc            1     Free     4K     4096            0                
POLICER1 (CHILD)        12    Reserved 16K    16384           0                
POLICER2 (PARENT)       13    Reserved 16K    4096            0                
====================================================================================
Note: Free and Unassigned Counter engines can be reassigned to other features
====================================================================================

Thus we have achieved , 






This completes Section 2 of the discussion.





Conclusion :

In this document, we explored the concept of hardware counters ,their purpose, operation, and implementation within the Cisco NCS 540 Large series routers. Building on that foundation, we analysed how these counters are allocated within the NPU across various features. This understanding enabled us to determine how counter engines can be reassigned between features based on specific requirements. Additionally, we reviewed the new enhancements introduced in the latest IOS XR release, which provide improved policy and sub-interface scalability, along with the capability to reassign counter engines within the QoS RX bank.


Note : This document is also applicable for NCS55/5700 - Twins & Eyrie.





