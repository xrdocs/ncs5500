---
published: true
date: '2021-05-23 00:25 +0200'
title: NCS5500 Chassis New Generation Commons
author: Nicolas Fevrier
excerpt: >-
  New route processor, new power supply and new fan trays / fabric cards for the
  NCS 5500 chassis
position: hidden
---
{% include toc icon="table" title="New Commons for NCS5500 Chassis" %} 

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Among the innovations brought with IOS XR 7.3.1, we introduced a lot of new software features but also now hardware.  
We now support new fixed-form systems (1xRU pizza box) like: [NCS57B1-6D24 / NCS57B1-5DSE](https://www.youtube.com/watch?v=MyqmIlozL8M). And also new chassis line cards and new commons. But "commons", we mean elements like power supply, fabric cards, fan trays, route processors or system controllers (we don't actually have a new SC, but we have new generations for all others).

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/D57C38pcfyo" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}
.  

**Mea Culpa**: At multiple occasions, I said "RP-2" instead of "RP-E", probably not help clarifying the options available. So the chronology of the route processor options are: RP, RP-E and RP2-E. That's the last one we are introducing today.
{: .notice--info}

## New parts for NCS5500 Chassis

### NC55-RP2-E

This is the third generation of Route Processor, after the RP and RP-E. It's supported in all types of chassis: 4-slot, 8-slot and 16-slot.  
The second generation (RP-E and not "RP-2" as mentioned in the video, RP2 doesn't exist) supported Class-B timing quality. And the only noticeable difference with the new RP2-E will be the supported of Class-C. To enable this feature, we will need the new RP but also line cards capable of this timing quality, for example the NC55-32T16Q4H-A (available in IOS XR 7.2.2 and 7.3.1) and the NC57-36H6D-S (coming in IOS XR 7.3.2). 

![RP2-E-01.png]({{site.baseurl}}/images/RP2-E-01.png){: .align-center}

Timing is the only difference with the previous RP-E.  
It does support the same scalability, memory and CPU power. So it will not offer higher route scale or fast convergence time.  

Note: it's not possible to mix different generations of route processor in the same chassis. RP2-E can only work with RP2-E.  

### NC55-5504-FC2 and NC55-5504-FAN2

The v2 fabric cards and fan trays are available for 8-slot and 16-slot chassis for a while (IOS XR 6.6.25). They are mandatory to support line cards equipped with Jericho2 NPU.  
In IOS XR 7.2.2 and 7.3.1, we complete the offer with the v2 fabric and fans for the 4-slot chassis. The same rules described in these articles apply to this 4-slot version:  
- you need them installed before inserting any J2 line card
- you can't mix v1 and v2 fan trays and fabric cards
- you can't upgrade from v1 to v2 in-service, you need to shut down the system, replace the parts and reboot
- Once equipped with v2 commons, you can add J2-based line cards. If the system needs to run both J/J+ and J2 line cards, you will run the default mode called "compatibility". If the system is populated with J2-cards exclusively, it's possible to enable the "native" mode via configuration.

We invite you to check the following articles and videos to get more details:  
- [https://xrdocs.io/ncs5500/tutorials/ncs-5500-fabric-migration/](https://xrdocs.io/ncs5500/tutorials/ncs-5500-fabric-migration/)
- [https://www.youtube.com/watch?v=XMQumuTkzmg](https://www.youtube.com/watch?v=XMQumuTkzmg)
- Comp/Native modes: [https://www.youtube.com/watch?v=oUdIBAghjgk](https://www.youtube.com/watch?v=oUdIBAghjgk)

**Note**: It's not currently possible to integrate the fabric cards in the Cisco Power Calculator (CPC). If you need an estimate of the power consumption, please reach out to your Cisco Representative, they will be able to use internal tools to run the simulation.
{: .notice--info}

![Ramon.png]({{site.baseurl}}/images/Ramon.png){: .align-center}



---





| Port N | Port N+1 |
|:------:|:------:|
| 400G | Must be emptied |
