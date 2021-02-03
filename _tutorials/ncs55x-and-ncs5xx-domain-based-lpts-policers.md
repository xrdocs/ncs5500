---
published: true
date: '2021-02-01 10:07 +0530'
title: NCS55x and NCS5xx Domain Based LPTS Policers
author: Tejas Lad
excerpt: This document will explain the domain based LPTS policers and its use cases
tags:
  - NCS5500
  - NCS500
  - LPTS
  - Control Plane Protection
  - LPTS Domain Based Filtering
position: hidden
---
{% include toc %}

## Introduction

It has been a while since we wrote the [first](https://xrdocs.io/ncs5500/tutorials/introduction-to-ncs55xx-and-ncs5xx-lpts/ "first") article on LPTS. There we introduced the concept of LPTS and how it is implemented on NCS55xx and NCS5xx family of routers. We also saw with examples how LPTS entries are created in the hardware and how they can be altered as per different requirements. In this document we will explore **Domain based LPTS Policers** and understand the use case of the feature.

![Screenshot 2021-02-01 at 12.07.56 PM.png]({{site.baseurl}}/images/Screenshot 2021-02-01 at 12.07.56 PM.png)

## Problem Definition

The usual LPTS implementation treats the traffic in a single domain and packet rate is controlled via single LPTS policer profile. Policer is implemented per flow-type which is classified by LPTS. But customer needs more granular control for LPTS configuration. They require to have separate policer values for different ports in the router. Some LCs in ASR9K support per NPU LPTS policer profiles, where for a set of ports under 1 NPU, a different policer profile can be defined. But the disadvantage is that again we do not have granularity per port. All the ports in the same NPU have to adhere the same LPTS policer defined. What if we want to have different policer for interfaces in the same NPU. Therefore on NCS55xx and NCS5xx we have the provision to achieve separate policer profiles for a set of ports or domain. Domain would be logical grouping of ports. Domains will provide the capability to select LPTS policer profile independently for the defined domains. This will provide better control of the ingress packets in the router.

## Advantages of using per domain LPTS 

![Screenshot 2021-02-01 at 5.28.04 PM.png]({{site.baseurl}}/images/Screenshot 2021-02-01 at 5.28.04 PM.png)

Let us take an example of NCS 5501. It has 48 1G/10G ports and 6 40G/100G ports. The first set of ports are located in core 0 and second set in core 1. The advantage of using per domain LPTS are:    
  - We can use existing concept of known/default policers
  - Per port resources are available on NCS55xx and NCS5xx 
  - It strengthens the existing security plus gives added granularity
  - Domain creates individual isolation of ports (e.g VRF)
  - This can scale even with just having internal TCAM
 
Note: By default all the ports will be classified in default domain if no user defined domain is configured. 
{: .notice--info}


## Domain Based LPTS Architecture
 
![Screenshot 2021-02-01 at 7.49.31 PM.png]({{site.baseurl}}/images/Screenshot 2021-02-01 at 7.49.31 PM.png)
 
The above figure represents the architecture of the domain based LPTS. For understanding of the terminology like port arbitrator, Pre-iFIB, iFIB etc and the basic flow of LPTS, it is highly recommended to read this [Article](https://xrdocs.io/ncs5500/tutorials/introduction-to-ncs55xx-and-ncs5xx-lpts/ "article"). As per the flow described in that artcile, first the entries are downloaded in LPTS Pre-IFIB. LPTS HW Pre-iFIB creates/updates/delete with domain index in the key of Pre-iFIB entry. LPTS domain and its associated interfaces are added/deleted to update port variable of interfaces. LPTS HW Policer is updated or programmed for a specific flow type under a specific LPTS domain index. LPTS Pre-iFIB PD does hardware programming of Domain based policers and HW Pre-iFIB entries to database table. LPTS Pre-iFIB then programs PMF, policer and port variable. Management client access the Pre-iFIB HW entry or policer data via the SysDB.


## Sample Use Case

![Screenshot 2021-02-01 at 8.21.20 PM.png]({{site.baseurl}}/images/Screenshot 2021-02-01 at 8.21.20 PM.png)

For example take the above network. It has been logically partioned into 2 domains i.e. core and peering. The core domain needs the traffic to be policed at a lower rate than the peering domain. To achieve this, concept of domain space partition in LPTS for the ports is very useful. It helps to utilize the port orientation in the network and will enable separate controllable policer profile per domain. 

Below output shows the default policer value under default domain for BGP-known

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:R1#show lpts pifib hardware police location all | in BGP 
Tue Feb  2 18:13:04.100 UTC
BGP-known              32116   Static  2500      2975      0         0-default</mark>
BGP-cfg-peer           32117   Static  2000      2000      0         0-default
BGP-default            32118   Static  100       8         0         0-default
</code>
</pre>
</div>

Below are some of the hardware entries programmed under default core.

![Screenshot 2021-02-02 at 11.57.02 PM.png]({{site.baseurl}}/images/Screenshot 2021-02-02 at 11.57.02 PM.png)

Now let us configure one interface under the domain core and see how it impacts the values and programming

```
lpts pifib hardware domain core
 interface TenGigE0/0/0/0
!
lpts pifib hardware police
 domain core
  flow bgp known rate 2000
 !
!
```
<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:R1#show lpts pifib hardware police location all | in BGP           
Tue Feb  2 18:44:46.931 UTC
BGP-known              32116   Static  2500      2975      0         0-default</mark>
BGP-cfg-peer           32117   Static  2000      2000      0         0-default
BGP-default            32118   Static  100       8         0         0-default
<mark>BGP-known              32216   Global  2000      2398      0         1-core</mark>
RP/0/RP0/CPU0:R1#
</code>
</pre>
</div>

![Screenshot 2021-02-03 at 12.19.25 AM.png]({{site.baseurl}}/images/Screenshot 2021-02-03 at 12.19.25 AM.png)

We can see 2 different entries are created. One in default domain and other in the core domain.

## Feature Support

- Domain configuration is supported only on physical and bundle main interfaces.
- The configuration will be rejected if we apply on sub-interfaces.
- Only 2 domains are allowed. One is default and other is user configured.
- It is supported on NCS540/NCS560 and NCS5500(J/J+/J2)

## Memory Impact

There would be a slight increase in memory for pifibm_server_rp/lc process due to this functionality. Some heap memory would be utilized in keeping the domain states and for caching the entries within the process which would be dynamically updated into platform as and when needed 
For normal programming of entries some extra checks on the entries would be added if this functionality is enabled to ensure domain information population.Overall in normal flow there would be very less impact while programming TCAM entries. With any configuration change there would be control plane churn as TCAM reprogramming is triggered. The TCAM entries in hardware would depend on the configuration used for ports and the scale of L3-routable entries (as L3 entries gets duplicated if additional domain is configured). 

 
## Scale
 
 The total number of domains supported would be as per platform capability.
