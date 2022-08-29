---
published: true
date: '2022-08-26 14:11 +0530'
title: Understanding MDB (Modular Databases) on NCS5700 systems
author: Deepak Balasubramanian
excerpt: MDB profile on J2 based systems
position: hidden
tags:
  - iosxr
---
## A New Post

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.

|Deepak Balasubramanian, Technical Marketing Engineer (deebalas@cisco.com)|

## Preface


In Today’s Network deployments we position our routers in various roles pertaining to different use cases. And it comes with its own interesting set of features and importantly different scale requirements. 
So how to fit in the same product addressing these different scale requirements? 
We bring in that modularity in the form of MDB (Modular Database) profiles in our NCS5700 Fixed systems and NC57 Line cards on the modular NCS5500 operating in Native mode.


## Introduction to MDB

What is a Modular Database? 

In the NCS5500/NCS5700 routers we have various on-chip databases for different purposes like storing the IP prefix table, MAC information, Labels, Access-list rules and may more.
On the scale variant we have an external-TCAM (ex- Optimus Prime2 in NCS5700 family) to offload storage for prefixes, ACL rules, QOS, etc. for a higher scale compared to the base systems.

We also have the on-chip buffers and off-chip HBM buffers used for queuing the packets. 
They are out of the scope for the MDB discussion.  
Our focus of discussion will be on the memory databases available on the Jericho2 family of asics.


## Jericho On-Chip Databases

A quick refresher of on-chip resources in Jericho asic in the below table. 
This shows the applications/features mapped to the important databases in the asic. 

![01-MDB.png]({{site.baseurl}}/images/01-MDB.png)

_Table #1 Jericho on-chip databases_

We use different asics from the same ASIC family on our NCS5500 and NCS5700 routers which are Qumran-MX, Jericho, Jericho+ and the latest Jericho2, Jericho2C, Qumran2C and Qumran2A asics.
Out latest platform in works will come with new Jericho2C+ asic from Broadcom.

The on-chip databases mentioned in the table #1 are common across all these different ASICs. But the flexibility to carve resources for these databases are only supported with the NCS5700 systems or line-cards using Jericho2 family of asics (Jericho2, Jericho2C, Qumran2C and Qumran2A)

|**NCS5500/5700 Products** | **MDB supported**  
|------------------------|------------------------------------------------------------------|
| Fixed Systems          | NCS-57B1-6D24H-S NCS-57B1-5D24H-SE NCS-57C3-MOD-S NCS-57C3-MOD-SE-S NCS-57C1-48Q6D-S NCS-57D2-18DD-S*
| Line cards Supported on NCS5504, NCS5508, NCS5516 (Native Mode only)       | NC57-18DD-SE NC57-24DD NC57-36H-SE NC57-36H6D-S NC57-MOD-S NC57-48Q2D-S* NC57-48Q2D-SE-S*

_Table #2 NCS5700 PIDs support for MDB (* PIDs in works for future releases)_


Let’s look into the forwarding ASIC layout. The blue boxes (which are circled) are some of the important on-chip memories which are used for storing information like IP prefixes, labels, MAC tables, next-hop information and may more.  

While we have other components like buffers and blocks which are essentially used for packets buffering and processing won’t be impacted by MDB carving.


![02-MDB.png]({{site.baseurl}}/images/02-MDB.png)

_Picture #1 ASIC layout_


In Jericho2 based platforms we give user the flexibility to carve resources for the circled on-chip databases in picture #1 based on the MDB profiles which are configured during the system bootup.
In the below pictorial representation (picture #2), we can see how the static carving of resources for on-chip databases have been made modular.


![03-MDB.png]({{site.baseurl}}/images/03-MDB.png)

_Picture #2 Databases having MDB impact_


On the left we have Jericho1 based systems where the database carving is always static which is now made modular in the Jericho2 based platforms. 
If we have a scaled system with external-TCAM , the on-chip resource carving for MDB is designed in way considering the features usage of the resources in the OP2 external TCAM.



## Benefits of MDB


In today’s network deployments we position our routers in various use cases ranging from metro access, aggregation, core, Peering, DCI, Cloud Interconnect and many more.
Each use case comes with its own interesting set of features and importantly the scale requirements.

Like in the Peering use case, where we position our high-density aggregation devices in the edge of the network with high eBGP sessions scale being metro, Datacenters, Internet Edge. We need features rich in Layer3 like high-capacity Routing scale, security features VRFs, ACL, FS and many more.

While for Business Aggregation use cases based on carrier-ethernet deployments are Layer2 heavy with requirements of higher MAC scale, l2vpn or Bridge-Domain scale 



![04-MDB.png]({{site.baseurl}}/images/04-MDB.png)
_Picture #3 Network deployment use cases_

So, it’s obvious that the requirements are not same for these different use cases.
Rather than just having a fixed profiles why not give users some flexibility in adding more resources to the databases which fits in for their scale requirements. 
**That flexibility is available with the MDB feature!**


## Path to MDB

During the initial release of NCS5700 platform, we started with shipping NCS57 based line cards on the NCS5500 system running in Compatible mode along with previous generation line cards based on Jericho1. We had default system profile to tune the scale and restrict the scale of system resources based on Jericho1 scale parameters.

Then in the subsequent release we started supporting native mode with all LCs on a modular NCS5500 are Jericho2(ncs57) for higher scale than the compatible mode. 
We supported both base and scale variants of Jericho2 LCs with custom scale profiles.

And in the next release(73x) we had the MDB infra developed in the XR software and introduced default profiles with higher scale. 
They were balanced (for base systems) and balanced-SE (for scale systems). 
And we made these as default profiles on the new SoC (system on chip) routers which were released in 73x.  
Please note the balanced/balanced-SE profiles were enabled by default and were not user configurable

![05-MDB.png]({{site.baseurl}}/images/05-MDB.png)

_Picture #4 Default MDB Profile - 73x_

Above picture #4 depicts the behavior during 73x release time on J2 based modular and fixed systems operating in native mode.

In 74x, the balanced profiles were reincarnated as L3MAX and L3MAX-SE profiles with better scale optimizations. 
On NCS5500 modular systems on native mode, default was L3MAX and we introduced a “hw-module CLI” to enable the L3MAX-SE if all the line cards on the system are scale (-SE) cards. 
On the SoC Jericho2 boxes, based on the availability of eTCAM we enable the right profile (base or SE) and they always operate in native mode. 


![06-MDB.png]({{site.baseurl}}/images/06-MDB.png)
_Picture #5   Modular ncs5500: MDB modes during 74x_


![07-MDB.png]({{site.baseurl}}/images/07-MDB.png)
_Picture #6   SoC systems default modes_



Then in release 752/761 we came up with layer-2 feature centric L2MAX and L2MAX-SE profiles for the base and scale variants of NCS5700 routers and line-cards.
The default mode of operation will be L3MAX (-SE) and if a user wishes to get L2 rich resource carving they are provided options to configure the L2MAX(-SE) profiles.  
Please note, these MDB profiles are supported on our systems with J2, J2C, Q2C & Q2A asics.
Also, MDB will be supported on the J2C+ based system being planned for XR781.

![08-MDB.png]({{site.baseurl}}/images/08-MDB.png)
_Picture #7   Q2C based fixed system default mode in 75x_

In latest releases (761 onwards) all the SoC and modular systems (in native mode) supports all  4 MDB profiles 


| No | MDB    |  Systems config  |  external TCAM presense
|---|----------|---------------------------|--------------------|
| 1 | L3MAX    | Base LCs & Fixed Systems  | No external TCAM 
| 2 | L3MAX-SE | Scale LCs & Fixed Systems | With external TCAM 
| 3 | L2MAX    | Base LCs & Fixed Systems  | No external TCAM 
| 4 | L2MAX-SE | Scale LCs & Fixed Systems | With external TCAM 

_Table #3 MDB profiles_


## MDB on Modular systems

This is a bit tricky. By default, the NCS5500 systems boots up in compatible mode.
We will need explicit “hw-mod profile” configuration to enable native mode provided we have all the line-cards being nc57 based.

With native mode configured (post reload) by default, system will operate in L3MAX mode in latest releases (74x and beyond).  
Users are given options to configure any of the MDB profiles based on their card configurations and requirements. (where L2 profiles are included in 761)


### Step by Step transition towards NCS5700 with MDB:

Let’s start with NCS5500 system having mix of Jericho1 and Jericho2 LCs. It will by default operate in compatible mode having a default scale profile.
1.	First step is to remove Jericho/Jericho+ LCs and convert it into Jericho2 only system. 
2.	Next step is to configure “native mode” and reload the system for the mode to take effect and the MDB profile will set to L3max 
(This is the right profile if we have a mix of scale and base J2 LCs)


![Picture #9   Compatible to native mode migration ]({{site.baseurl}}/images/09-MDB.png)

_Picture #9   Compatible to native mode migration_ 


3.	In the above state #2 (with mixed scale and base J2 LCs), user can configure L2max profile based on their requirements

![10-MDB.png]({{site.baseurl}}/images/10-MDB.png)

_Picture #10   L2max MDB configuration_

