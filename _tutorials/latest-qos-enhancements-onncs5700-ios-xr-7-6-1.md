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

As explained in our previous article, the queuing model on NCS 5500 is Virtual output Queues (VoQ) based and it happens on the ingress NPU in the packet path. With IOS XR 7.6.1, there is a new queuing mode intordocued on NCS 5700 system where queuing is done on the NPU where the egress port belong. This improves the overall system scale in terms of QoS scale by restricting VoQ distribution and also allows better flexibility in terms of QoS functionality. This feature is applicable to NCS 5700 system with external TCAM.

This new mode is called Egress Traffic Manager (ETM), and can be enabled on port basis while non ETM port behaves the previous way. This article will cover in depth explanation on the implementation and configuration aspects of the newly introduced ETM mode for QoS. 


## Quick Recap of VoQ Model

As per the below diagram, The  NCS 5500 (or NCS 5700) system, there are 8 VoQs per attachment point. However, they are present at the ingress pipleine of the data path. Now, for a particular egress port/interface, traffic may ingess at any other port in the system. Therefore, the VoQ for the particular egress port is replicated on each ingress pipeline (NPU/LC) present in the system. The packets are forwarded to the egress port with exchange of credit messege from egress to ingress VoQ schedulars. 
![voq-non-etm.png]({{site.baseurl}}/images/voq-non-etm.png)


## Egress Traffic Manager (ETM) Architecture and Data Path

### VoQs with ETM

The new ETM mode, when enabled, restricts the replication of VoQ across the system. Rather packets are queued only on the egress NPU i.e VoQ on the ingress pipeline of the egress NPU. For non ETM port, the previous architercure holds good. For the ETM enabled ports, queues are created only on the local NPU. The recycle port VoQ on each NPU is replicated across the system where packets are queued first for ETM enabled ports. Therefore the VoQ replication with ETM can happen three ways,

- ETM enabled port VoQ replicated only on the local NPU
- non ETM port VoQs replicated across the system
- NPU recycle port VoQs replicated across the system

![voq-etm.png]({{site.baseurl}}/images/voq-etm.png)


### Life of a Packet with ETM
The following diagram explains the data path for packets destined to a port enabled with ETM. In a modular system it can be briefly explained as six step process.

![etm-data-path.png]({{site.baseurl}}/images/etm-data-path.png)


1. when packet enters the ingress interface the lookup at ingress points to remote RCY port
2. Packet forwarding to destination NPU
  (If Queuing needed it is on RCY port VoQ)
3. Packet reached to destination NPU egress pipeline (RCY port)
4. Packet is recycled back to ingress pipeline of destination NPU. Lookup at this stage points to actual port
5. Packet is forwarded to Egress Pipeline (actual port). Queuing at this stage is done on the actual port VoQ
6. Packet Goes out of egress Port 


## ETM Configuration
Enabling and configuring QoS policies in ETM mode is a three step process. 
- Enabling ETM on the port
- Defining ETM Policy map
- Applying policy to intreface

### Enabling ETM
ETM needs to be enabled on the main port using controller optics configuration. Once enabled it erases the existing interface configuration and the same is shown as a warning during the configuration process. In case of breakout, ETM needs to be enabled under the controller optics for the newly created ports.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NC57B1-57-II-5#configure terminal 
RP/0/RP0/CPU0:NC57B1-57-II-5(config)#controller optics 0/0/0/0 
RP/0/RP0/CPU0:NC57B1-57-II-5(config-Optics)#mode etm 
Wed May 18 09:05:48.532 UTC
!! Warning ! This will remove the existing interface configuration
RP/0/RP0/CPU0:NC57B1-57-II-5(config-Optics)#commit 
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
controller Optics0/0/0/0
 mode etm
!
</code>
</pre>
</div>

Once ETM is enabled, we can verify the same by checking the VoQ allocation. As we can see in the below output, there are two VoQ bases allotted to the ETM enabled port. the first one corresponds to the VoQ base (1792) for the egress port whereas the second base corresponds to  VoQ base (1024) of the recycle port where packets will be queued in the first pass

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NC57B1-57-II-5#show  controllers npu  voq-usage interface all  instance all location all 
Wed May 18 09:07:39.531 UTC

-------------------------------------------------------------------
Node ID: 0/RP0/CPU0
Intf         Intf     NPU NPU  PP   Sys   VOQ   Flow   VOQ    Port 
name         handle    #  core Port Port  base  base   port   speed
             (hex)                                     type        
----------------------------------------------------------------------
<mark>Hu0/0/0/0    3c000048   0   0    9   521   1792   6912 local   100G</mark>
<mark>Hu0/0/0/0    3c000048   0   0  156     9   1024   6160 local   100G</mark>
Hu0/0/0/1    3c000058   0   0   11    11   1072   6192 local   100G
Hu0/0/0/2    3c000068   0   0   13    13   1080   6208 local   100G
Hu0/0/0/3    3c000078   0   0   15    15   1088   6224 local   100G
Hu0/0/0/4    3c000088   0   0   17    17   1096   6240 local   100G
Hu0/0/0/5    3c000098   0   0   19    19   1104   6256 local   100G
</code>
</pre>
</div>

### Defining ETM policy
QoS policy Map for ETM ports is just like a regular policy with few exceptions.

#### classification
Ideally, queuing policy uses traffic-class for classifying traffic into different queues, these traffic-class values are set on the ingress. With ETM, since we get the feature rich ingress pipeline again, the classification can be done just like an ingress policy map. i.e we can classify and queue based on QoS fields present in the packet header.

- L2 : cos, dei
- L3 : precedence/dscp/ACLs/fragments
- MPLS: EXP 

There is an way to match based on traffic-class as well which need a special hw-mdoule CLI needs to be enabled.
<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile qos ipv6 short-etm</mark> 
</code>
</pre>
</div>
The unmatched traffic class in this case goes to class-default.


#### Actions in the policy-map

For and ETM policy-map we can have queing actions like shaping, queue-limit, priority, BWR for WFQ and RED/WRED. Upto 4 priority levels are supported in an ETM policy. There is no support for policing and bandwidth command.

There must be  a marking action with "set traffic class "  on each user defined class apart from class default. This is to choose the VoQ where the traffic will be queued. for class default TC value is 0, rest of the class can be allotted TC values between 1-7.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
class-map match-any prec4
 match precedence 4 
 end-class-map
! 
class-map match-any prec5
 match precedence 5 
 end-class-map
! 
class-map match-any prec6
 match precedence 6 
 end-class-map
! 
!
policy-map etm-policy
 class prec6
  shape average percent 2 
  priority level 1 
  set traffic-class 6
 ! 
 class prec5
  shape average percent 38 
  priority level 2 
  set traffic-class 5
 ! 
 class prec4
  bandwidth remaining percent 65 
  set traffic-class 4
 ! 
 class class-default
  bandwidth remaining percent 35 
 ! 
 end-policy-map
! 
</code>
</pre>
</div>

### Attaching Policy to Interface
An ETM policy can be applied to the main or the subinterface of the port enabled with ETM mode. Unlike previous releases with normal mode, we don't need to enable `hw-module profile qos hqos-enable` to add policy on subinterafce for ETM ports. In fact both ETM & hqos mode can't coexist together in the system.


<div class="highlighter-rouge">
<pre class="highlight">
<code>
interface HundredGigE0/0/0/0.1
 service-policy output etm-policy
 vrf test
 ipv4 address 57.1.0.1 255.255.255.0
 encapsulation dot1q 1
!
interface HundredGigE0/0/0/0.2 l2transport
 encapsulation dot1q 2
 service-policy output etm-policy
</code>
</pre>
</div>

## ETM Related Facts

### ETM and Queuing Scale
when ETM is enabled VoQ resources across the system is saved as there is no need to replicate the same across the system. Thus, queuing scale increases signicantly for the system. 

### New QoS functionality with ETM
ETM makes the feature rich ingress pipeline available for the egress QoS function. Thus we are able to do classification based on parameters like cos/dscp/exp for egress. This adds support for QoS short pipe mode.

with ETM, multicast traffic is also scheduled and can be shaped along with unicast which is not the case for normal/non-ETM mode.

### ETM vs throughput & latency

ETM involves two pass where packet is recylced back on the egress NPU. This reduces the NPU level throughput and it may go down to 50% when ETM is enabled on all the ports.
The second pass will also add few microseconds of added latency for the traffic destined towards an ETM enabled port.

### ETM and shaper granularity
by default, shaper granularity on NCS 5700 system is ~4 mbps. With ETM, there is a low speed mode where more granular shaper ~122 kbps can be configured. This low rate mode is activated when any shaper present in the policy-map is less than 5 Mbps. The below outpit shows programming of a low rate shaper and a normal shaper.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NC57B1-57-II-5#show  qos int hundredGigE 0/0/0/0.3 output 
NOTE:- Configured values are displayed within parentheses
Interface HundredGigE0/0/0/0.3 ifh 0x3c00800a  -- output policy
NPU Id:                        0
Total number of classes:       1
Interface Bandwidth:           100000000 kbps
Policy Name:                   4mbps
SPI Id:                        0x0
VOQ Base:                      1816
PFC enabled:                   0
Accounting Type:               Layer1 (Include Layer 1 encapsulation and above)
------------------------------------------------------------------------------
Level1 Class                             =   class-default
Egressq Queue ID                         =   1816 (Default LP queue)
Queue Max. BW.                           =   <mark?4028 kbps (4 mbits/sec)</mark>
Queue Min. BW.                           =   0 kbps (default)
Inverse Weight / Weight                  =   1 / (BWR not configured)
Guaranteed service rate                  =   4000 kbps
Peak burst                               =   32832 bytes (default)
TailDrop Threshold                       =   4864 bytes / 10 ms (default)

<mark>LOW SHAPER                               =   Enabled</mark>

WRED not configured for this class


RP/0/RP0/CPU0:NC57B1-57-II-5#show  qos int hundredGigE 0/0/0/0.4 output 
NOTE:- Configured values are displayed within parentheses
Interface HundredGigE0/0/0/0.4 ifh 0x3c008012  -- output policy
NPU Id:                        0
Total number of classes:       1
Interface Bandwidth:           100000000 kbps
Policy Name:                   5mbps
SPI Id:                        0x0
VOQ Base:                      1824
PFC enabled:                   0
Accounting Type:               Layer1 (Include Layer 1 encapsulation and above)
------------------------------------------------------------------------------
Level1 Class                             =   class-default
Egressq Queue ID                         =   1824 (Default LP queue)
Queue Max. BW.                           =   <mark>7812 kbps (5 mbits/sec)</mark>
Queue Min. BW.                           =   0 kbps (default)
Inverse Weight / Weight                  =   1 / (BWR not configured)
Guaranteed service rate                  =   5000 kbps
Peak burst                               =   36864 bytes (default)
TailDrop Threshold                       =   6144 bytes / 10 ms (default)
WRED not configured for this class
</code>
</pre>
</div>

### ETM with Bundle
On the NCS 5700 system, Egress policy on bundle is replicated per member interface. Therefore, all members of a bundle has to be either ETM or non ETM. we can't have bundle with mix of ETM and non ETM ports.


![etm-bundle.png]({{site.baseurl}}/images/etm-bundle.png)


## Conclusion
So with IOS-XR 7.6.1, we brought this  in new enhancements in QoS segment, as a part of our continous improvement and innovations. ETM as a function will adress QoS scalability and functionality in the NCS 5700 scaled system. Each release we keep on enhancing our sofwtare and hardware capabilities. These enhancements help strengthen our portfolio and helps in catering customer requirements. Stay tuned for new updates in future releases !!!
