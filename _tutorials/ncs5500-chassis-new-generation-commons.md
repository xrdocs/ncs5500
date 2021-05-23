---
published: true
date: '2021-05-23 00:25 +0200'
title: NCS5500 Chassis New Generation Commons
author: Nicolas Fevrier
excerpt: >-
  New route processor, new power supply and new fan trays / fabric cards for the
  NCS 5500 chassis
position: top
---
{% include toc icon="table" title="New Commons for NCS5500 Chassis" %} 

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Among the innovations brought with IOS XR 7.3.1, we introduced a lot of new software features but also new hardware.  
We now support new fixed-form systems (1xRU pizza box) like: [NCS57B1-6D24 / NCS57B1-5DSE](https://www.youtube.com/watch?v=MyqmIlozL8M). And also new chassis line cards and new commons. By "commons", we mean elements like power supply, fabric cards, fan trays, route processors or system controllers (we don't actually have a new SC, but we have new generations for all others).

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/D57C38pcfyo" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}
.  

**Mea Culpa**: At multiple occasions, I said "RP-2" instead of "RP-E", certainly creating some confusions to the watcher. So the chronology of the route processor options are: RP, RP-E and RP2-E. That's the last one we are introducing today.
{: .notice--info}

## New parts for NCS5500 Chassis

### NC55-RP2-E

This is the third generation of Route Processor, after the RP and RP-E. It's supported in all types of chassis: 4-slot, 8-slot and 16-slot.  
The second generation (RP-E and not "RP-2" as mentioned in the video, RP2 doesn't exist) supported Class-B timing quality. And the only noticeable difference with the new RP2-E will be the support of Class-C. To enable this feature, we will need the new RP but also specific line cards, for example the NC55-32T16Q4H-A (available in IOS XR 7.2.2 and 7.3.1) and the NC57-36H6D-S (coming in IOS XR 7.3.2). 

![RP2-E-01.png]({{site.baseurl}}/images/RP2-E-01.png){: .align-center}

Timing is the only difference with the previous RP-E.  
It does support the same scalability, memory and CPU power. So it will not offer higher route scale or fast convergence time.  

Note: it's not possible to mix different generations of route processor in the same chassis. RP2-E can only work with RP2-E.  

### NC55-5504-FC2 and NC55-5504-FAN2

The v2 fabric cards and fan trays are available for 8-slot and 16-slot chassis for a while (IOS XR 6.6.25). They are mandatory to support line cards equipped with Jericho2 NPU.  
In IOS XR 7.2.2 and 7.3.1, we complete the offer with the v2 fabric and fans for the 4-slot chassis. The same rules described in the past apply to this 4-slot version too:  
- you need them installed before inserting any J2 line card
- you can't mix v1 and v2 fan trays and fabric cards
- you can't upgrade from v1 to v2 in-service, you need to shut down the system, replace the parts and reboot
- Once equipped with v2 commons, you can add J2-based line cards. If the system needs to run both J/J+ and J2 line cards, you will use the default mode called "compatibility". If the system is populated with J2-cards exclusively, it's possible to enable the "native" mode via configuration.

We invite you to check the following articles and videos to get more details:  
- [https://xrdocs.io/ncs5500/tutorials/ncs-5500-fabric-migration/](https://xrdocs.io/ncs5500/tutorials/ncs-5500-fabric-migration/)
- [https://www.youtube.com/watch?v=XMQumuTkzmg](https://www.youtube.com/watch?v=XMQumuTkzmg)
- Comp/Native modes: [https://www.youtube.com/watch?v=oUdIBAghjgk](https://www.youtube.com/watch?v=oUdIBAghjgk)

**Note**: It's not currently possible to integrate the fabric cards in the Cisco Power Calculator (CPC). If you need an estimate of the power consumption, please reach out to your Cisco Representative, they will be able to use internal tools to run the simulation.
{: .notice--info}

![Ramon.png]({{site.baseurl}}/images/Ramon.png){: .align-center}

The v2 fabric cards are using "Ramon" Fabric Engines from Broadcom:  
- 3 per NC55-5516-FC2
- 2 per NC55-5508-FC2
- 1 per NC55-5504-FC2

As mentioned above, they can connect to J2 line cards with PAM4 50G SERDES and to J/J+ line cards with NRZ 25G SERDES.

### NC55-PWR-4.4KW-DC

![Patongv2.png]({{site.baseurl}}/images/Patongv2.png){: .align-center}

Latest addition to the power module supported in the modular NCS5500 chassis. It's a DC-only power supply, offering up to 4400W, 3 inputs from -48V to -60V DC, and it's supported on all types of chassis (4-slot: 4 PSUs, 8-slot: 8 PSUs and 16-slot: 10 PSUs).

![PSUs.png]({{site.baseurl}}/images/PSUs.png){: .align-center}

This level of power (4400W) is a significant improvement compared to the former generations (3000W in DC and 3300W in HVDC), something mandatory to support both N+1 and N+N redundancy with the new generations of line cards based on J2.  
Introduced with IOS XR 7.3.1, we don't support the mix with previous PSU generations.

These new PSU has 6 lugs connectors for 3 DC inputs:

![PSU-connection1.png]({{site.baseurl}}/images/PSU-connection1.png){: .align-center}

It allows the connection of each PSU to grid A and grid B, and offers a third pair. We suggest to alternate the connection of this third feed between A and B.  
In such scenario, all PSUs are providing 4.4kW each.
That will guarantee full feed redundancy in case of outage like the loss of Grid A in the example below:

![PSU-connection2.png]({{site.baseurl}}/images/PSU-connection2.png){: .align-center}

With this logic, whether we lose grid A or grid B, each pair of PSUs can provide 4400 + 2200 = 6.6kW.

## Conclusion

We introduced new components of the chassis to support new requirements: class-C timing, 400G requirements in the 4-slot chassis, N+N power supply redundancy with the new power needs.  
Next article and video will be dedicated to the new line cards introduced in IOS XR 7.3.1.  
Stay tuned.
