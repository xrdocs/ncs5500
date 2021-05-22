---
published: true
date: '2019-09-04 14:52 +0200'
title: Introducing 400GE on NCS5500 Series
author: Nicolas Fevrier
excerpt: >-
  Introducing two new 400GE capable line cards in the NCS5500 portfolio:
  NC55-24DD and NC55-18DD-SE
position: top
tags:
  - ncs5500
  - ncs 5500
  - 400GE
  - 400G
  - really fast ethernet
---
{% include toc icon="table" title="Introducing 400GE on NCS5500" %} 

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

They have been announced in June 2019 on this blog post: [https://blogs.cisco.com/sp/cisco-ncs-5500-welcomes-400ge-and-doubles-installed-base](https://blogs.cisco.com/sp/cisco-ncs-5500-welcomes-400ge-and-doubles-installed-base), it's now time to present in detail the two new line cards with 400 Gigabit Ethernet (400GE) capabilities.

And we will start with 8-min of videos to introduce the topic:

[YouTube: Introducing 400GE on NCS5500](https://www.youtube.com/watch?v=ckab8JfFUMw)

<iframe width="560" height="315" src="https://www.youtube.com/embed/ckab8JfFUMw" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}
.  

We will cover the following in this blog post:

- Pre-requisites before inserting the new line cards
- Description of the NC57-24DD and NC57-18DD-SE
- Interoperability with former generation
- New fabric bandwidth, speed-up and redundancy
- Features at FCS
- 400GE and QSFP-DD (with breakout to 100G)

We plan to add other sections to this article in the future. It will be a living document.

## Pre-requisites

Before inserting the new products, we need to verify the system is running the minimal IOS XR release and is equipped with the appropriate "commons".

### Hardware

It will be mandatory to upgrade the fabric cards and the fan trays in the chassis before any 400GE line card insertion.  
We differentiate the new fan trays and fabric cards by the number "2" at the end of the product IDs:
- NC55-5504-FC2 / NC55-5508-FC2 / NC55-5516-FC2 
- NC55-5504-FAN2 / NC55-5508-FAN2 / NC55-5516-FAN2


A few notes on these parts:
- 8-slot and 16-slot are supported since version IOS XR 6.6.25 but you will need a newer release for the 400GE line cards. With 6.6.25, you can still insert them and benefit from the lower power consumption with cards based on Jericho and Jericho+
- 4-slot support has been introduced in IOS XR 7.2.2 and 7.3.1
- we don't support in-service migration. You'll have to shut down the chassis and replace them.
- we can't mix different generations
	- v2 Fabric Cards can only operate with other v2 Fabric Cards
    - v2 Fan Trays can only operate with v2 Fan Trays
    - we can't mix v1 Fan Trays with v2 Fabric Cards, or vice versa. All must be v2.
    
To avoid incident, we have it clearly labelled on the new parts.

Regarding the other chassis parts: Route Processors, System Controllers and Power Supply Modules:
- RP and RP-E can both operate with the new line cards
- SCs exist in a single version today which operates with the new line cards
- number of power supply will depend on the numbers and types of cards, please refer to the Cisco Power Calculator for accurate calculation (today it only contains the v2 FC and FT, the new line cards will be added later on): [https://cpc.cloudapps.cisco.com/cpc/DS.cpc](https://cpc.cloudapps.cisco.com/cpc/DS.cpc)

### Software

The new fan trays and fabric cards are supported since IOS XR 6.6.25 but don't be confused by it. We will need a minimum version of __IOS XR 7.0.2__ to support the new line cards NC57-24DD and NC57-18DD-SE.  
Regarding the features supported at inception, please refer to the feature section of this blog post.

## New Line Cards

Before describing the two new cards and their respective port density, let's take a look at the ASIC powering them.

### NPU: J2

These new line cards are powered by 2x Jericho2 NPUs from Broadcom.

These ASICs are offering 4.8Tbps of interface bandwidth and 5.6Tbps to the fabric.

![J2-1.jpg]({{site.baseurl}}/images/J2-1.jpg){: .align-center}

Like its predecessors (used in NCS5500 Series), the J2 NPUs are made of two cores, an ingress pipeline and an egress pipeline. It's capable of approx 2000 MPPS.  
The main differences will be:
- the use of faster SERDES with a different encoding scheme
- the use of HBM instead of DRAM
- the use of MDB instead of multiple internal memories
- the use of a second generation external TCAM (OP2)

They are interconnected through Ramon ASICs located in the fabric cards.  
The SERDES between Ramon and J2 are around 53 Gbps each (between the v2 Fabric cards and the 400G line cards).  
Since they are backward compatible with former line card generations, they are still able to use 25Gbps SERDES between Ramon and Jericho/Jericho+ ASICs too.

The Jericho2 uses a High Bandwidth Memory (HBM) instead of a GDDR5. We have double the amount of packet buffers (8GB) and double the speed to access it (1.8Tbps). One of the benefits of this HBM is it does not consume "links" to connect to the NPU, offering more to the interfaces and fabric.  
Keep in mind that this HBM is only used to store packets in case of congestion (micro-burst or longer-term link saturation). Most of the time, the packets will transit only through the on-chip buffer (OCB) which is now twice the size of former generation (32MB) and they will not be stored in the HBM.  
You can check the studies done on J+ on this topic in other xrdocs.io articles:  
[https://xrdocs.io/ncs5500/tutorials/ncs5500-qos-part-2-verifying-buffering/](https://xrdocs.io/ncs5500/tutorials/ncs5500-qos-part-2-verifying-buffering/)

![J2-2.jpg]({{site.baseurl}}/images/J2-2.jpg){: .align-center}

Also, very different from the former Jericho generations (J/J+), we will use a Modular DataBase (MDB) to store the router information (prefixes, MAC, MPLS, adjacencies, next-hop, ...). As the name implies, this database can be carved at the boot up and allocate more or less space depending on the use-case. It permits to give more scale to L3 or to L2, etc.  
Finally, we will use a newer generation of external TCAM with these NPUs too.

![J2-3-mdb.jpg]({{site.baseurl}}/images/J2-3-mdb.jpg){: .align-center}

We will have two of these Jericho2s in each of the line cards we are introducing.

### NC57-24DD

![Vigor-3quart.jpg]({{site.baseurl}}/images/Vigor-3quart.jpg){: .align-center}

This first line card will offer 24 slots QSFPDD-capable. It means they will also be able to support QSFP+ (40G) and QSFP28 (100G) optics.  

The front view of the card:

![Vigor-face.jpg]({{site.baseurl}}/images/Vigor-face.jpg){: .align-center}

**Reminder**: the chassis must have been upgraded to IOS XR 7.0.2 minimum and must operate v2 Fabric Cards and Fan Trays before inserting these line cards.
{: .notice--info}

The currently estimated power consumption is: 1050W typical and 1350W max.

The line card is slightly longer than former generations as shown here:

![chassis-with-vigor.jpg]({{site.baseurl}}/images/chassis-with-vigor.jpg){: .align-center}

For NEBS compliancy, it will require new doors. They are currently in the roadmap.

Internally, the card is made of two J2 NPUs, each servicing half of the ports for a total of 4.8Tbps.

![Vigor-block-diagram.jpg]({{site.baseurl}}/images/Vigor-block-diagram.jpg){: .align-center}

### NC57-18DD-SE

The second line card is offering more physical ports, higher scale but less overall bandwidth.

![VigorSE-3quart.jpg]({{site.baseurl}}/images/VigorSE-3quart.jpg){: .align-center}

Despite the name 18DD, we have a total of 30 QSFP-DD ports on the front plate:

![VigorSE-face.jpg]({{site.baseurl}}/images/VigorSE-face.jpg){: .align-center}

Among these ports, some should be considered in pairs, some others should be considered individually.

![VigorSE-face-with-blocks.jpg]({{site.baseurl}}/images/VigorSE-face-with-blocks.jpg){: .align-center}

The ports from 18 to 23 (highlighted in yellow in the diagram below) are directly attached to Jericho#1 and, individually, they can be used with QSFP56-DD/400G optics.  
They will also host QSFP56/200G, QSFP28-DD/2x100G, QSFP28/100G or QSFP+/40G (depending on the software support).

![VigorSE-face-with-blocks2.jpg]({{site.baseurl}}/images/VigorSE-face-with-blocks2.jpg){: .align-center}

The other ports (highlighted in yellow in the diagram below) should be treated in pairs. That means what we can insert in port 1 is directly dependent of what has been inserted in port 0.  

![VigorSE-face-with-blocks3.jpg]({{site.baseurl}}/images/VigorSE-face-with-blocks3.jpg){: .align-center}

| Port N | Port N+1 |
|:------:|:------:|
| 400G | Must be emptied |
| 200G | 200G |
| Hi-Speed 400G / 200G | Can’t be used with Low-Speed 100G / 40G |
| 200G | Can’t be 2x 100G |
| 100G | 100G / 40G |

If you use only QSFP56-DD, only 18 ports can be populated, hence the line card name: 18DD-SE. Also, on the front plate, you can see in blue the ports that can be used with 400GE optics.

We will add more details on these subtleties in the next months while we are getting closer to IOS XR 7.0.2 release date.

The currently estimated power consumption is: 1000W typical and 1400W max.

For reference, here is the block diagram on the 18DD-SE line card:

![VigorSE-block-diagram.jpg]({{site.baseurl}}/images/VigorSE-block-diagram.jpg){: .align-center}

**Note**: None of the NC57-24DD or the NC57-18DD-SE are MACsec capable but the timing features are possible and will be enabled in future software releases (of course, it will require the proper RP-Es).
{: .notice--info}

## Fabric Speed-Up and Redundancy

The 8-slot fabric are made of 2 Ramon ASICs while the 16-slot fabric cards contains 3 of them.  
Each Jericho2 connects to each Fabric Module with a total of 18 SERDES at 53.125Gbps.  
They can be evenly distributed between one, two or three Fabric Engine (Ramon).  
- 8-slot: 9 SERDES connected to 2 Ramon/FC
- 16-slot: 6 SERDES connected to 3 Ramon/FC
- 4-slot: 18 SERDES connected to 1 Ramon/FC

For the 8-slot chassis:

![8-slot-fabric-j2.jpg]({{site.baseurl}}/images/8-slot-fabric-j2.jpg){: .align-center}

For the 16-slot chassis:

![16-slot-fabric-j2.jpg]({{site.baseurl}}/images/16-slot-fabric-j2.jpg){: .align-center}

The 53.125Gbps per SERDES represents the raw bandwidth. After cell tax, encoding and correction, we can actually transport 45.8Gbps.

To calculate the bandwidth available per Jericho2, we will use the following math:
- 6 Fabric Modules
	- 18x 53.125Gbps x 6FM = 5737Gbps (raw)
	- 18x 45.8Gbps x 6FM = 4946Gbps
- 5 Fabric Modules
	- 18x 53.125Gbps x 5FM = 4781Gbps (raw)
	- 18x 45.8Gbps x 5FM = 4122Gbps (85.8% of 4800Gbps)

If we lose one fabric card, the NC57-24DD can not be line rate (around 86%) while the NC57-18DD-SE will still be.  
It's important to state the obvious here, we are talking about lab corner-cases since the situation where 12 ports 400GE of the same card will be used at an average level exceeding 86%, at the exact moment we lose a fabric card is virtually impossible.

## 400GE and QSFP-DD

Cisco invested in QSFP-DD for all the 400 Gigabit Ethernet plans.  
For more details on the technology, the best place is certainly this CiscoLive session: [https://www.youtube.com/watch?v=46D4zs_TlrM&list=PLWgSsi4SkfuaJaY6GeCE7wG52dKn-NOir](https://www.youtube.com/watch?v=46D4zs_TlrM&list=PLWgSsi4SkfuaJaY6GeCE7wG52dKn-NOir)

I want to bring to the reader's attention that breakout of 400GE to 4 ports 100GE will not interoperate with most currently deployed technologies (based on SR4, LR4 and CWDM4).  
It may sound obvious, but it is still worth mentioning that 400GE is based on 4 lambdas of 100G each encoded in PAM4 while the technologies above are using 4 lanes of 25G encoded in NRZ. That makes the interconnection impossible.  
Breakout of 400GE is still possible, but it will imply to use 100G "one-lambda PAM4" optics on the other side:

![breakout.jpg]({{site.baseurl}}/images/breakout.jpg){: .align-center}

In the other hand, QSFP28-DD will be perfect to breakout in two SR4/LR4/CWDM4. A perfect use-case for the NC57-18DD-SE ports operating in pairs, it will allow a smooth transition from existing 100G backbones to 400G.

## Mixing Line Card Generations and Features at FCS

At FCS, in IOS XR 7.0.2, we will support the "compatibility mode" where Jericho/Jericho+ line cards will co-exist with Jericho2-based line cards.  
In this specific mode, the features and scales are aligned to:
- features for peering and core (including multicast and L3VPN)
- same scale supported in XR 6.5.1

The following list is still susceptible to change before the FCS date, but we will not support:  
- ERSPAN
- Lawful Intercept
- mLDP/P2MP Edge
- Timing
- 400G Auto-Negotiation
- EVPN
- L2 / BVI or L2 OAM
- Features in 6.6.X onward

Features will be added to the support list release after release in the usual iterative process.  
Also, in the future, we will add the "J2 Native Mode" where all the line cards inserted in the chassis are only J2-based. It will "unleash" the full capability of the ASIC. In the compatibility mode the features and scale are often limited by the lower common denominator, being the Jericho/Jericho+ NPUs.

## Conclusion

With the introduction of these two new line cards, we are boosting the NCS5500 chassis from 3.6Tbps to 9.6Tbps per slot. That's 153.6Tbps per router !!!  
More details will be added to this post or in new ones very soon.  
As usual, use the section below for comments and questions.
