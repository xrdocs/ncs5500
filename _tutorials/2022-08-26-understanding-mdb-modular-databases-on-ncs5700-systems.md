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


