---
published: false
date: '2022-08-18 21:58 +0530'
title: Understanding MDB (Modular Databases) on NCS5700 systems
---
## A New Post

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.

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







