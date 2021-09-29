---
published: true
date: '2021-09-28 15:34 +0530'
title: Egress IPv6  ACLs on NCS5500
author: Tejas Lad
excerpt: >-
  This document will describe IPv6 ACL Egress ACLs on NCS5500 J/J+ and highlight
  the enhancements when it comes to J2
tags:
  - iosxr
  - NCS5500
  - ncs5500
  - J2
  - Jericho2
  - ACL
  - IPv6
  - Egress ACL
position: hidden
---
{% include toc icon="table" title="Egress IPv6  ACLs on NCS5500" %} 

## Introduction

In the [previous article](https://xrdocs.io/ncs5500/tutorials/acl-s-on-ncs5500-bvi-interfaces/), we discussed the ACLs on NCS5500 BVI interfaces. We covered the ACL implementation and support for IPv4 and IPv6 ACLs in both ingress and egress directions. We also discussed the enhancements w.r.t IPv6 Egress ACLs. In this article we discuss the overall implementation of IPv6 Egress ACLs across NCS5500 product family (J/J+ and J2).

## Overview

![Screenshot 2021-09-29 at 2.16.58 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 2.16.58 PM.png)

Let us start with the earlier generations of NCS5500. As we already know, platforms based on J/J+ have limited hardware resources. They have very limited capabilities in the Egress PMF as well. PMF means Programmable Mapping and Filtering block in the ingress and egress pipeline. It is one of the blocks in the pipeline. It is the most programmable and the last programmable block in the pipeline. It has all the history of the packet from other blocks (incoming port, lookup results, etc). We can override here every decision taken along the pipeline. Here we do ACL, QoS, LPTS classification and set actions (counters, policers, Traffic Class).  Egress PMF is capable of doing internal TCAM lookup for egress ACLs

![Screenshot 2021-09-29 at 4.45.23 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 4.45.23 PM.png)

In order to support the match criteria that is needed for egress IPv6 ACL, resources in the Ingress PMF block will be used. Therefore, this requires us to recycle the IPv6 packets so that the ACL can be performed by the Ingress PMF. 

## Two Pass Design

Egress IPv6 ACL requires two passes of each IPv6 packet that is subject to ACL. The recycling of the IPv6 packets are controlled by entries in the Egress PMF.  Each packet will go through the following stages:

  - 1st pass ingress
  - 1st pass egress (recycle packet)
  - 2nd pass ingress
  - 2nd pass egress

### 1st Pass Ingress

![Screenshot 2021-09-29 at 5.28.14 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 5.28.14 PM.png)

During the first pass the normal ingress processing will be performed. This includes ingress PMF, QoS, and any other configured features. The forwarding decision, is determined based on the Destination System Port (DSP). Each DSP in the system has a corresponding set of VoQs. Since a FEC is pointing to the DSP, packets are sent to destination port on the destination NPU. The header contains DSP  as system port. The packets will be put in the DSP’s VoQ and will be scheduled by the DSP’s End-to-End (E2E) scheduler

### 1st Pass Egress

![Screenshot 2021-09-29 at 9.11.42 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 9.11.42 PM.png)

When an IPv6 ACL is attached to an egress interface, a set of egress PMF entries will created to redirect the IPv6 packet out the recycle port. Based the recycle port configuration, a new program will be selected which will take care of internal processing. At high level it will ensure a fixed packet offset after recycling. It will also build a dummy Ethernet header with Ethertype=IPv6 to prepare for 2nd pass ingress parsing

### 2nd Pass Ingress

![Screenshot 2021-09-29 at 9.20.30 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 9.20.30 PM.png)

After the IPv6 packet has been recycled, it will be received on the same NPU/Core as the egress DSP. A field group in PMF  will retrieve DSP, Traffic class and drop precedence from the recycled packet's system header. DSP is retrieved so that the recycled packet will go out the same Egress port.  We don't want to go through forwarding again and possibly choose a different egress port. The Traffic Class might have been set by QoS in the 1st pass Ingress processing. We want to preserve this TC. The Drop Precedence might have been set by QoS in the 1st pass ingress processing. We want to preserve this Drop Precedence. Ingress PMF will now perform the ACL. 
If matches on a Deny ACE, the normal processing will occur and the packet will be dropped or  forwarded to the Control Plane for ICMPv6 handling. If matches on a Permit ACE, the configured action will occur plus the additional actions to strip the recycled system header. Forwarding will use the retrieved DSP from the recycled system header. The packets will be put in the DSP’s VoQ and will be scheduled by the DSP’s end to end scheduler **a second time**. 


### 2nd Pass Egress

The original system headers will be used for normal egress processing


## TCAM Entries 

Egress IPv6 ACL has 2 databases 
  - EGRESS_ACL_IPV6 
  - RCY_ACL_L3_IPV6 

Because the packets are recycled, the EGRESS_ACL_IPV6 database has entries that will facilitate the recycling mechanism. The actual match entries are added at RCY_ACL_L3_IPV6.

EGRESS_ACL_IPV6:

(A)   1 static entry per NPU per core for the recycle-channel

(B)   3 entries for setting up the different kinds of TPIDs supported (IPv6 (0x86dd), VLAN (0x8100), or MPLS (0x8847))

 

RCY_ACL_L3_IPV6:

(C)   the match entries (computed) from the ACL

If you apply the same ACL to X egress interfaces, (B) and (C) will increase X times. While (A) will only increase when the interfaces are in different NPUs.

## Link Utilization due to recycle

Since the packet is scheduled twice by the DSP’s E2E schedule, a maximum of 50% link utilization will be supported as, bandwidth for the recycle channel will be set to be 50% of core bandwidth.

## J2 Enhancement




