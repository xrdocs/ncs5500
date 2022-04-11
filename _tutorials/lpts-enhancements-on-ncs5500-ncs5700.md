---
published: true
date: '2022-04-11 12:10 +0530'
title: 'LPTS Enhancements on NCS5500/NCS5700 '
author: Tejas Lad
excerpt: >-
  This document will cover the latest enhancements on the newer generation
  NCS5500 platforms
tags:
  - iosxr
  - cisco
  - LPTS
  - NCS5500
  - NCS5700
  - control plane
  - CoPP
position: hidden
---
{% include toc icon="table" title="LPTS Enhancements on NCS5500/NCS5700" %} 

## Introduction

In our previous article, we had introduced the [LPTS architecture](https://xrdocs.io/ncs5500/tutorials/introduction-to-ncs55xx-and-ncs5xx-lpts/) on NCS5500 and NCS500 product family. There we introduced the concept of LPTS and its internal architecture. We also saw with examples how LPTS entries are created in the hardware and how they can be altered as per different requirements. We then followed it up with introduction to [Domain based LPTS Policers](https://xrdocs.io/ncs5500/tutorials/ncs55x-and-ncs5xx-domain-based-lpts-policers/) and understanding its use cases. In this article, we will discuss the LPTS latest enhancements on the newer generation products.

![Screenshot 2022-04-11 at 12.53.23 PM.png]({{site.baseurl}}/images/Screenshot 2022-04-11 at 12.53.23 PM.png)


## Overview

Before we move on to this topic, it would be recommended to visit our LPTS architecture document for understanding the implementation on the platform. As we covered, in the document LPTS is an integral component of **IOS-XR** systems which provides firewall and policing functionality. LPTS maintains per interface complete table in netio chain in Line card CPU, making sure that packets are delivered to their intended destinations. IOS XR software classifies all ‘For Us’ control packets into **97** different flows. Each flow has it own hardware policer to restrict the punt traffic rate for the flow type. We also discussed how the LPTS processes the for-us packets in the two pass in the hardware pipeline. For-us packets will go through the ASIC twice before getting punted to the CPU. In the current impplementation this already happens in iTCAM.

From IOS-XR 7.6.1, the implementation will be changed. To increase the scale of the LPTS entries, the second pass will happen in the eTCAM instead of iTCAM. This will help increase the LPTS routing packet entries to 12000 (from current support of 8k). This helps in scaling the other protocol entries up to 1.5 times the current scale.

## Platform Support

This enhancement is only supported on platforms based on Jericho2 and Jericho2C with external TCAM. Platforms that do not have external TCAM does not support this enhancement. This is supported only in native mode. It is not supported in compatibility mode. Earlier generation platforms based on Jericho/Jericho+ do not support this enhancement even if they have external TCAM.

For understanding Native vs Compatible mode please watch the following [video](https://www.youtube.com/results?search_query=native+vs+compatible+mode+NCS5500).

## Implementation 

Let us have a look at how this works internally at high level

### 1st Pass

- The packet ingresses on the network interface and a forwarding destination lookup.
- This gives the packet a valid compression id value or a FEC value. 
- Also the forwarding trap value for the packet may be set depending on the type of packet.
- In this stage, the trap value for the packet is modified to a user defined recycle trap id. 
- This happens for packets that have a valid compression ID value. 
- For packets with TTL=1, the TTL1 trap is set via INGRESS_IPV4/6 instead. 
- Both traps ensures that the packet is recycled back to the IRPP.


### 2nd Pass

- The recycled packet lookup for 2nd pass happens in the external tcam is done via a recycle context selection criteria. 
- This applies to all IPv4/v6 unicast and multicast packets and packets with the options attribute set. 
- The hardware will have 2 lookups internally. The first will be for the forwarding destination or compression ID the other will be for LPTS. 

Note: This happens transparently in the platforms once it is upgraded to IOS-XR 7.6.1 and operating in native mode
{: .notice--info}

## IOS-XR 7.6.1 Support Matrix



## Configurations

## 


