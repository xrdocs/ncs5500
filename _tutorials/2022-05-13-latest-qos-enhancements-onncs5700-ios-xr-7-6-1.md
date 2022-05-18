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


### ETM Data Path
The following diagram explains the data path for packets destined to a port enabled with ETM. In a modular system it can be briefly explained as six step process.

---- add diagram for ETM Data Path ---


## ETM Configuration Steps
Enabling and configuring QoS policies in ETM mode is a three step process. 
- Enabling ETM on the port
- Defining ETM Policy map
- Applying policy to intreface

### Enabling ETM
ETM needs to be enabled on the main port using controller optics configuration. Once enabled it erases the existing interface configuration and the same is shown as a warning during the configuration process. if there is breakout used then ETM needs to be enabled under the controller optics for the newly created ports.

```
config snippet from ETM

```

Once ETM is enabled, we can verify the same by checking the VoQ allocation. As we can see in the below output, there are two VoQ bases allotted to the ETM enabled port. the first one corresponds to the VoQ base for the NPU recycle port whereas the second base corresponds to the actual port VoQ where packets will be queued in the second pass.

```
Add VoQ output here

```

### Defining ETM policy
QoS policy Map for ETM ports is just like a regular policy with few exceptions.

#### classification
Ideally, queuing policy uses traffic-class for classifying traffic into different queues, these traffic-class values are set on the ingress itslef. With ETM, since we get the feature rich ingress pipeline again, the classification can be done just like an ingress policy map. i.e we can classify and queur based on QoS fields present in the packet header.

- L2 : cos, dei
- L3 : precedence/dscp/ACLs/fragments
- MPLS: EXP 

There is an way to match based on traffic-class as well which need a special hw-mdoule CLI to be enabled. 

#### Actions in the policy-map

For and ETM policy-map we can have queing actions like shaping, queue-limit, priority, BWR for WFQ and RED/WRED. Upto 4 priority levels are supported in an ETM policy. there is no support for policing and bandwidth command.

There must be  a marking action with "set traffic class <\>"  on each user defined class apart from class default. This is to choose the VoQ where the traffic will be queued. for class default TC value is 0, rest of the class can be allotted TC values between 1-7.


## ETM Related Facts

### ETM and Queuing Scale
when ETM is enabled VoQ resources across the system is saved as there is no need to replicate the same across the system. Thus queuing scale is increases signicantly for the system. 
### New QoS functionality with ETM
ETM makes the feature rich ingress pipeline available for the egress QoS function. Thus we are able to do classification based on parameters like cos/dscp/exp for egress. This adds support for QoS short pipe mode.

with ETM, multicast traffic is also scheduled and can be shaped along with unicast which is not the case for normal/non-ETM mode.

### ETM vs throughput & latency

ETM involves two pass where packet is recylced back on the egress NPU. This reduces the NOU level throughput and it may go down to 50% when ETM is enabled on all the ports.
The second pass will also add few microseconds of added latency for the traffic destined towards an ETM enabled port.

### ETM and shaper granularity
by default, shaper granularity on NCS 5700 system is ~4 mbps. With ETM, there is a low speed mode where more granular shaper ~122 kbps can be configured. This low rate mode is activated when any shpaer present in the policy-map is less than 5 Mbps.

### ETM with Bundle
On the NCS 5700 system, Egress policy on bundle is replicated per member interface. Therefore, all members of a bundle has to be either ETM or non ETM. we can't have bundle with mix of ETM and non ETM ports.


```
add Bundle diagram

```


## Conclusion
