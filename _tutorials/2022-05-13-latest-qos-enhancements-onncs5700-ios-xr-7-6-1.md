---
published: false
date: '2022-05-13 13:49 +0530'
title: Latest QoS Enhancements onNCS5700 - IOS-XR 7.6.1
author: Paban Sarma
excerpt: >-
  This Article covers the latest QoS Innovation brought to NCS 5700 Series
  Platforms with IOS XR 7.6.1
tags:
  - iosxr
  - cisco
  - NCS 5700
---
{% include toc %}

```
Revision History:
v1: Updated as of IOS XR 7.6.1
```


## Introduction

As explained in our previous article, the queuing model on NCS 5500 is Virtual output Queues (VoQ) based and it happens on the ingress NPU in the packet path. With IOS XR 7.6.1, there is a new queuing mode intorocued on NCS 5700 system where queuing is done on the NPU where the egress port belong. This improves the overall system scale in terms of QoS scale by restricting VoQ distribution and also allows better flexibility in terms of QoS functionality. This feature is applicable to NCS 5700 system with external TCAM.

This new mode is called Egress Traffic Manager (ETM), and can be enabled on port basis while non ETM port behaves the previous way. This article will cover in depth explantion on the implementation and configuration aspects of the newly introduced ETM mode for QoS. 


## Quick Recap of VoQ Model

As the below diagram on and NCS 5500 (or NCS 5700) system, there are 8 VoQs per attachment point. However, they are present at the ingress pipleine of the data path. Now, for a particular egress port/interface, traffic may ingess at any other port in the system. Therefore, the VoQ for the particular egress port is replicated on each ingress pipeline (NPU/LC) present in the system. The packets are forewarded to the egress port with exchange of credit messege from egress to ingress VoQ schedulars. 

---- add the diagram of VoQ architecture & Credit flows----

## Egress Traffic Manager (ETM) Architecture and Data Path

### VoQs with ETM

The new ETM mode, when enabled restricts the replication of VoQ across the system. Rather packets are queued only on the egress NPU i.e VoQ on the ingress pipeline of the egress NPU. For non ETM port, the previous architercure holds good. for the ETM enabled ports, queues are created only on the local NPU. The recycle port VoQ on each NPU is replicated across the system where packets are queued first for ETM enabled ports. Therefore the VoQ replication with ETM can happen three ways,

- ETM enabled port VoQ replicated only on the local NPU
- non ETM port VoQs replicated across the system
- NPU recycle port VoQs replicated across the system

---- Add diagram for the ETM/non ETM VoQ model -----


### ETM Model Data Path
The following diagram explains the data path for packets destined to a port enabled with ETM. In a modular system it can be briefly explained as six step process.

---- add diagram for ETM Data Path ---


## ETM Configuration Steps

## ETM Related Facts

## Conclusion
