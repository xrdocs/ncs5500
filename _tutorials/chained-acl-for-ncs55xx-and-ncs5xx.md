---
published: true
date: '2020-09-21 10:32 +0530'
title: Chained ACL for NCS55xx and NCS5xx
author: Tejas Lad
excerpt: >-
  This document will discuss the concept and use cases of Chained ACL  for
  NCS55xx and NCS5xx
tags:
  - NCS55xx
  - NCS5xx
  - NCS5500
  - ACL
  - Chained ACL
position: top
---
{% include toc %}

## Introduction

In our previous tech-notes, we introduced the ACL support ([ACL Introduction](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/ "ACL Introduction")), discussed different matching criteria's ([Matching Fragments](https://xrdocs.io/ncs5500/tutorials/acl-ip-fragments-matching-ncs55xx-and-ncs5xx/ "Matching Fragments"), [Packet Length Matching](https://xrdocs.io/ncs5500/tutorials/acl-packet-length-matching-ncs55xx-and-ncs5xx/ "Packet Length Matching")), and explored the concepts of different TCAM keys ([UDK](https://xrdocs.io/ncs5500/tutorials/user-defined-key-udk-for-ncs55xx-and-ncs5xx/ "UDK"), [UDF](https://xrdocs.io/ncs5500/tutorials/user-defined-field-ncs55xx-and-ncs5xx/ "UDF"))for the NCS55xx and NCS5xx portfolio. In this tech-note, we will talk about Chained ACL and see its use cases.


## Overview

![Screenshot 2020-09-21 at 7.37.35 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-21 at 7.37.35 PM.png)

Prior to the IOS-XR release 7.2.1, the packet filter (pfilter) only supported one ACL to be applied per direction and per protocol on any given interface. In live production networks, there are instances where they have ACL's applied on multiple interfaces (including physical and sub-interfaces) and most of the time it happens, many ACL's end up having similar ACE's. For example, consider an edge box of an ISP which has 2 sets of ACE's. One set may be common ISP specific ACE's to protect ISP's infrastructure as a whole and other would be interface specific ACE's which might be for blocking customer related address block. This is done to protect the ISP infrastructure against attacks by allowing only valid address blocks while denying the suspicious ones. To achieve this, we have to configure unique ACL per interface. By doing this we may end up having multiple ACL's having common ACE's across all/most of them.

Modying ACL rules which are common to provider infrastructure will require changes to be done on each and every interface. This leads to manageability issues if there are any common ACL entries which are needed on every/most interface/interfaces. Also this ACL provisioning and modification can be very cumbersome for any operator or enterprise as any changes to the ACE impacts every customer interface. Another important thing to note is, as we have unique ACL's per interface, it also wastes the valuable hardware resources, as the common ACEs are being replicated in all ACL's. 

To avoid the impact to multiple customer interface due to modifications, there have been multiple request to support a feature, which can help accomodate more than one ACL a single interface. The goal is to separate various types of ACLs for specific reasons, yet apply both of them on the same interface, in a defined order. Therefore from IOS-XR release 7.2.1, we bring in the support for Chained ACL also known as Common ACL, across NCS55xx and NCS5xx portfolio. This feature will be supported on platforms having **Q-MX, Jericho, Jericho+ and Jericho2 (Native and Compatible)**.

![Screenshot 2020-09-22 at 11.29.00 AM.png]({{site.baseurl}}/images/Screenshot 2020-09-22 at 11.29.00 AM.png)
 
 
## Feature Support

- Only 1 common/chained IPv4 and IPv6 ACL is supported per line card.
- The common/chained ACL can be applied to any type of interfaces which supports interface ACL's (i.e VRF enabled interfaces, VLAN interfaces, Bundle interfaces).
- The common/chained ACL is supported in the ingress direction only. 
- The common/chained ACL is searched first before the interface ACL.
- Edit of common/chained ACL is supported.
- Statistics of common/chained ACL is supported.
- The is no change in scale after applying the common/chained ACL.
- There is no performance impact with both the common and interface ACL applied.
- ACL with ABF is supported with common ACL from IOS-XR 7.6.1

## What's Not Supported

- This feature is not supported in egress direction.
- This feature is not supported on Layer 2 interfaces.
- ACL with object groups is not supported with common ACL. 
- It cannot be configured on the same line card which has compression configured.
- Atomic replace of the common ACL is not supported.

## Common ACL behaviour with hw-module profile

| HW-module Profile                                                                | Common ACL Configuration |
|----------------------------------------------------------------------------------|--------------------------|
| hw-module profile tcam format access-list is used and common-acl is included     | Allowed                  |
| hw-module profile tcam format access-list is used and common-acl is not included | Not Allowed              |
| hw-module profile tcam format access-list is not used                            | Allowed                  |

## Configuring a common ACL

![Screenshot 2020-09-21 at 3.44.22 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-21 at 3.44.22 PM.png)

This is how you configure the common ACL along with interface specific ACL. We will see the detailed configurations in later section.

## Hardware Programming

Consider you have configured below ACL's on 2 interfaces along with a common ACL.

```
interface TenGigE0/0/0/0
ipv4 access-group common ACL_Comm ACL1 ingress

interface TenGigE0/0/0/1
ipv4 access-group common ACL_Comm ACL2 ingress

```

![Screenshot 2020-09-21 at 7.33.57 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-21 at 7.33.57 PM.png)

The above figure shows how the hardware programming will happen in this case. The common ACL is programmed once in a TCAM and is located at the top of the TCAM. Interface ACL's are programmed below the common ACL. The TCAM search order is from top to bottom which gives the common ACL precedence over the interface ACL. The single instance of the common ACL in a TCAM ensures scalability when thousands of interfaces are enabled on an NP.  However, since the hardware resources for the common ACL must be reserved, a static number of TCAM entries are allocated. 

Note: An interface may contain only the common ACL, only an interface ACL, or both the common and interface ACL.
{: .notice--info}


## Common ACL Implementation

Now that we have talked about the feature and its use case, let us move on to see how this feature is implemented. We have below 2 ACL's configured as an example.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>ipv4 access-list ACL1_full</mark>
 10 deny ipv4 any 62.6.69.88 0.0.0.7
 15 deny ipv4 62.6.69.88 0.0.0.7 any
 20 deny ipv4 any 62.6.69.112 0.0.0.15
 25 deny ipv4 62.6.69.112 0.0.0.15 any
 30 deny ipv4 any 62.6.69.128 0.0.0.15
 35 deny ipv4 62.6.69.128 0.0.0.15 any
 40 deny ipv4 any 62.80.66.128 0.0.0.15
 45 deny ipv4 62.80.66.128 0.0.0.15 any
 50 deny ipv4 any 62.134.38.0 0.0.0.127
 60 permit tcp any eq bgp host 1.2.3.1
 70 permit tcp any host 1.2.3.1 eq bgp
 80 deny ipv4 any host 1.2.3.1
 90 deny ipv4 any 212.21.217.0 0.0.0.255
 100 permit ipv4 any any
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>ipv4 access-list ACL2_full</mark>
 10 deny ipv4 any 62.6.69.88 0.0.0.7
 15 deny ipv4 62.6.69.88 0.0.0.7 any
 20 deny ipv4 any 62.6.69.112 0.0.0.15
 25 deny ipv4 62.6.69.112 0.0.0.15 any
 30 deny ipv4 any 62.6.69.128 0.0.0.15
 35 deny ipv4 62.6.69.128 0.0.0.15 any
 40 deny ipv4 any 62.80.66.128 0.0.0.15
 45 deny ipv4 62.80.66.128 0.0.0.15 any
 50 deny ipv4 any 62.134.38.0 0.0.0.127
 60 permit tcp any eq bgp host 7.8.9.6
 70 permit tcp any host 7.8.9.6 eq bgp
 80 deny ipv4 any host 7.8.9.6
 90 permit ipv4 any any
</code>
</pre>
</div>

**Applying it on 2 separate interfaces**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 usage pfilter location 0/0/CPU0 
<mark>Interface : TenGigE0/0/0/0.10 
    Input  ACL : Common-ACL : N/A  ACL : ACL1_full</mark>  
    Output ACL : N/A
<mark>Interface : TenGigE0/0/0/0.20 
    Input  ACL : Common-ACL : N/A  ACL : ACL2_full</mark> 
    Output ACL : N/A
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show controllers npu internaltcam location 0/0/CPU0 
Mon Sep 21 11:22:15.856 UTC
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   pmf-0       1883     107     36   INGRESS_LPTS_IPV4
0    0      160b   pmf-0       1883     18      45   INGRESS_RX_ISIS
0    0      160b   pmf-0       1883     23      54   INGRESS_QOS_IPV4
0    0      160b   pmf-0       1883     15      56   INGRESS_QOS_MPLS
0    0      160b   pmf-0       1883     2       60   INGRESS_EVPN_AA_ESI_TO_FBN_DB
<mark>0    1      160b   pmf-0       1962     34      47   INGRESS_ACL_L3_IPV4</mark> 
</code>
</pre>
</div>

From the above we can see 34 entries are being programmed in the TCAM.

If you closely look at the 2 ACL's we have most of the ACE's in common

![Screenshot 2020-09-21 at 5.03.06 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-21 at 5.03.06 PM.png)

Let us apply the same ACL's along with common ACL.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>ipv4 access-list common-acl</mark>
 10 deny ipv4 any 62.6.69.88 0.0.0.7
 15 deny ipv4 62.6.69.88 0.0.0.7 any
 20 deny ipv4 any 62.6.69.112 0.0.0.15
 25 deny ipv4 62.6.69.112 0.0.0.15 any
 30 deny ipv4 any 62.6.69.128 0.0.0.15
 35 deny ipv4 62.6.69.128 0.0.0.15 any
 40 deny ipv4 any 62.80.66.128 0.0.0.15
 45 deny ipv4 62.80.66.128 0.0.0.15 any
 50 deny ipv4 any 62.134.38.0 0.0.0.127
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>ipv4 access-list ACL1_specific</mark>
 10 permit tcp any eq bgp host 1.2.3.1
 20 permit tcp any host 1.2.3.1 eq bgp
 30 deny ipv4 any host 1.2.3.1
 40 deny ipv4 any 212.21.217.0 0.0.0.255
 50 permit ipv4 any any
 </code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>ipv4 access-list ACL2_specific</mark>
 10 permit tcp any eq bgp host 7.8.9.6
 20 permit tcp any host 7.8.9.6 eq bgp
 30 deny ipv4 any host 7.8.9.6
 40 permit ipv4 any any
</code>
</pre>
</div>

![Screenshot 2020-09-21 at 5.17.46 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-21 at 5.17.46 PM.png)

## Resource Utilization with Common ACL.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show controllers npu internaltcam location 0/0/CPU0 
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   pmf-0       1883     107     36   INGRESS_LPTS_IPV4
0    0      160b   pmf-0       1883     18      45   INGRESS_RX_ISIS
0    0      160b   pmf-0       1883     23      54   INGRESS_QOS_IPV4
0    0      160b   pmf-0       1883     15      56   INGRESS_QOS_MPLS
0    0      160b   pmf-0       1883     2       60   INGRESS_EVPN_AA_ESI_TO_FBN_DB
<mark>0    1      160b   pmf-0       1970     26      47   INGRESS_ACL_L3_IPV4</mark>
</code>
</pre>
</div>

From the above output we can see that with the use of common ACL along with interface specific ACL's we are using only **26** entries in the TCAM as compared to previously **34** entries. If we need to make changes in ACE's which are common to interfaces, we just need to change it in the common-ACL and no need to make changes on each and every interface, making the manageability easier. This is a very simple example with only 2 interfaces. We can see its usage when we apply it to multiple interface at the same time.

## Reference

  - [CCO Config Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/76x/b-ip-addresses-cg-ncs5500-76x/m-implementing-access-lists-prefix-lists-ncs5500.html)
  
## Summary

In this tech-note we successfully demonstrated the concept of Chained or Common ACL. We saw how it makes ACL manageability easy and also helps in saving the TCAM resources. This is another capability of the NCS55xx and NCS5xx in terms of dataplane security.
