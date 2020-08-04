---
published: true
date: '2020-08-04 11:16 +0530'
title: ACL Packet Length Matching - NCS55xx and NCS5xx
author: Tejas Lad
excerpt: >-
  This document describes the implementation of ACL packet length matching on
  NCS55xx and NCS5xx
position: hidden
tags:
  - cisco
  - NCS5500
  - NCS540
  - NCS560
  - ACL
  - NCS 5500
---
{% include toc %}

## Introduction

Access Control Lists have been implemented for a long time now and has been integral part of data plane security for several organizations.

![Screenshot 2020-08-04 at 11.03.21 AM.png]({{site.baseurl}}/images/Screenshot 2020-08-04 at 11.03.21 AM.png)

Though everyone would be aware of what an access control list is, I would like to brush up some basics before deepdiving into complex functionalities

ACL’s can be considered as a ordered list of conditions used to test the network traffic that traverses through the router interfaces. On the basis of the defined lists, the router decides which packets to accept and which to drop. ACL’s help in managing the traffic and secure the acsess to and from the network.

ACL’s can make permit/deny decisions based on source/destination address, source/destination ports, L3 protocols, L4 port numbers and many others.

## Overview

Secuirty ACL's introduction and the feature support is covered at high level in the [Link](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/ "Link")
In this document, we will deepdive how the NCS55xx and NCS5xx program the packet length in the TCAM and use to filter the packets.  

ACL’s on NCS55xx and NCS5xx, uses the Programmable Mapping and Filtering (PMF) functionality and TCAM (internal/exertnal) in both the Ingress Receive Packet Processing (IRPP) blocks and Egress Receive Packet Processing (ERPP) blocks. The line cards in these platforms are based on the Broadcom family of chipsets. These chipsets uses a pipeline architecture which has dedicated hardware blocks for performing various functions.

ACL’s contains one or more ACEs which are used to match packets and perform an action on those packets. Typical action of the ACE is to either Permit or Deny. The TCAM is programmed with tables, may be like a database on which the match criteria and action criteria are performed.

In hardware has TCAM databases that are unique to each feature. For ACL, we have further defined unique databases based on these fields:

  - Protocol (IPv4, IPv6, L2)
  - Direction (ingress/egress)
  - Compression (uncompressed/compressed)
  - Default TCAM key or user-defined (UDK) TCAM key

## Supported ACL Match Criteria 

  - Packet Length 
  - IP Fragmentation
  - Source and Destination Port
  - User Defined Keys - UDK
  - User Defined Field - UDF

```
Note: We will have dedicated posts for explaining each matching criterias.
```

## Header Definition - IPv4

  - In NCS55xx and NCS5xx, when we configure an ACE through CLI, the total IP packet includes only IPv4 header 
  - It does not include any L2 headers, including ethernet/vlan.  
  - So when matching the packets on the router, this needs to be taken into consideration.
  - Let us see an example of ACL configuration and see the TCAM programming and match the same via sending traffic

**We have different criterias for the matching** 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24(config-ipv4-acl)#40 permit ipv4 any any packet-length ?
   <mark>eq     Match only packets with a given value
  gt     Match only packet with a greater value
  lt     Match only packets with a lower value
  neq    Match only packets not on a given value
  range  Match only packets in the range of value</mark>
RP/0/RP0/CPU0:N55-24(config-ipv4-acl)#40 permit ipv4 any any packet-length 
</code>
</pre>
</div>

**Lets configure a simple ACL for matching packet length and attaching it to the interface**

```
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 test-acl-v4-pkt-length 
Thu Jul 23 06:46:11.884 UTC
ipv4 access-list test-acl-v4-pkt-length
 10 permit ipv4 any any packet-length eq 800
 20 permit ipv4 any any packet-length eq 1000
 30 permit ipv4 any any packet-length eq 1500
RP/0/RP0/CPU0:N55-24#

RP/0/RP0/CPU0:N55-24#show running-config interface tenGigE 0/0/0/0.10
Thu Jul 23 06:46:34.378 UTC
interface TenGigE0/0/0/0.10
 description using it for ACL testing
 ipv4 address 60.1.1.1 255.255.255.0
 load-interval 30
 encapsulation dot1q 10
 ipv4 access-group test-acl-v4-pkt-length ingress
!

RP/0/RP0/CPU0:N55-24#

```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
After 6.5.2 and later the packet length is not supported by default TCAM key and we need to configure a UDK to have it in the key. For example:
<mark>hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto packet-length frag-bit port-range</mark>
</code>
</pre>
</div>

**Hardware or TCAM programming of the ACL**

![Screenshot 2020-08-04 at 12.17.34 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-04 at 12.17.34 PM.png)

![Screenshot 2020-08-04 at 12.18.14 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-04 at 12.18.14 PM.png)

  - The above 2 output shows us that the IPv4 L3 ACL is programmed and a Database ID is created for it.
  - NPU details can be retrieved dedicated to interface where the ACL is applied
  - This data/values will help us in understanding the values configured in the hardware.
  
![Screenshot 2020-08-04 at 12.24.17 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-04 at 12.24.17 PM.png)

The above output shows the TCAM programming of the packet length configured in Hexadecimal

| Hexadecimal | Decimal |
|-------------|---------|
| 320         | 800     |
| 3E8         | 1000    |
| 5DC         | 1500    |


**Traffic Test**

Below is the traffic stream which is used. It has a packet length of 822 bytes. (800 bytes plus 18 bytes of the Ethernet header + 4 bytes VLAN header)

The traffic is matching the first ACE with packet length of 800 bytes. As its mentioned earlier that the TCAM doesnt consider L2 headers. So the traffic stream has to be sent accordingly. In real production network the ACE has to be configured accordingly so we can permit or deny legitimate packets. 

Permit ACL stats are not enabled by default. We need to enable the below hw-module profile to enable the same 

```
hw-module profile stats acl-permit

```

This can be done by configuring range command. It will be explained in the later section








