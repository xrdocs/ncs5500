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
position: hidden
---
{% include toc %}

## Introduction

In our previous tech-notes, we introduced the ACL support ([ACL Introduction](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/ "ACL Introduction")), discussed different matching criteria's ([Matching Fragments](https://xrdocs.io/ncs5500/tutorials/acl-ip-fragments-matching-ncs55xx-and-ncs5xx/ "Matching Fragments"), [Packet Length Matching](https://xrdocs.io/ncs5500/tutorials/acl-packet-length-matching-ncs55xx-and-ncs5xx/ "Packet Length Matching")), and explored the concepts of differenct TCAM keys ([UDK](https://xrdocs.io/ncs5500/tutorials/user-defined-key-udk-for-ncs55xx-and-ncs5xx/ "UDK"), [UDF](https://xrdocs.io/ncs5500/tutorials/user-defined-field-ncs55xx-and-ncs5xx/ "UDF"))for the NCS55xx and NCS5xx portfolio. In this tech-note, we will talk about Chained ACL and see its use cases.


## Overview

![Screenshot 2020-09-21 at 2.00.16 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-21 at 2.00.16 PM.png)

Prior to the IOS-XR release 7.2.1, the packet filter (pfilter) only supported one ACL to be applied per direction and per protocol on any given interface. In a typical customer scenario, there are instances where they have ACL's applied on many interfaces (including physical and sub-interfaces). Most of the time it happens, many ACL's end up having similar ACE's. Consider an edge box of an ISP which has 2 sets of ACEs. One may be common ISP specific ACEs to protect ISP infrastructure as a whole and other would be interface specific ACEs which might be for blocking customer related address block. This is done to protect the ISP infrastructure against attacks by allowing only valid. To achieve this, we have to configure unique ACL per interface out of which most of the ACEs being common across all the ACLs on a box. 

Modying ACL rules which are common to provider infrastructure will require changes to be done on each and every interface. This leads to manageability issues if there are any common ACL entries which are needed on every/most interface/interfaces. Also this ACL provisioning and modification can be very cumbersome for any operator or enterprise as any changes to the ACE results impacts every customer interface. Another important thing to note is, as we have unique ACL's per interface, it also wastes the valuable hardware resources, as the common ACEs are being replicated in all ACL's. 

To avoid the impact to multiple customer interface due to modifications, there have been multiple request to support a feature, which can help accomodate more than one ACL a single interface. The goal is to separate various types of ACLs for specific reasons, yet to apply both of them on the same interface, in a defined order. Therefore from IOS-XR release 7.2.1, we bring in the support for Chained ACL also known as Common ACL, across NCS55xx and NCS5xx portfolio. This feature will be supported on platforms having **Q-MX, Jericho, Jericho+ and Jericho2**.

 
## Feature Support

- Only 1 common/chained IPv4 and IPv6 ACL supported on each line card.
- The common/chained ACL can be applied to any type of interface which supports interface ACLs (i.e VRF enabled interfaces, VLAN interfaces, Bundle interfaces)
- The common/chained ACL is supported in the ingress direction only. 
- The common/chained ACL is searched first before the interface ACL.
- Edit of common/chained ACL is supported.
- The is no change in scale after applying the common/chained ACL.
- There is no performance impact with both the common and interface ACL applied.

## What's Not Supported

- This feature is not supported in egress direction.
- ACL with object groups is not supported with common ACL as of now. 
- ACL with ABF is not supported with common ACL.
- It cannot be configured on the same line card which has compression configured.
- Atomic replace of the common ACL is not supported.

## Common ACL behaviour if hw-module profile is configured

| HW-module Profile                                                                | Common ACL Configuration |
|----------------------------------------------------------------------------------|--------------------------|
| hw-module profile tcam format access-list is used and common-acl is included     | Allowed                  |
| hw-module profile tcam format access-list is used and common-acl is not included | Not Allowed              |
| hw-module profile tcam format access-list is not used                            | Allowed                  |

## Configuring a common ACL

![Screenshot 2020-09-21 at 3.44.22 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-21 at 3.44.22 PM.png)

This is how you configure the common ACL along with interface specific ACL. We will see the configuration of the ACL's along with ACE's in later section.

## Hardware Programming

Consider you have configured below ACL's on 2 interfaces along with a common ACL.

```
interface TenGigE0/0/0/0
ipv4 access-group common ACL_Comm ACL1 ingress

interface TenGigE0/0/0/1
ipv4 access-group common ACL_Comm ACL2 ingress

```

![Screenshot 2020-09-21 at 3.40.16 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-21 at 3.40.16 PM.png)

The above figure shows how the hardware programming will happen in this case. The common ACL is programmed once in a TCAM and is located at the top of the TCAM. Interface ACLs are programmed below the common ACL. The TCAM search order is from top to bottom which gives the common ACL precedence over the interface ACL. The single instance of the common ACL in a TCAM ensures scalability when thousands of interfaces are enabled on an NP.  However, since the hardware resources for the common ACL must be reserved, a static number of TCAM entries are allocated. 

Note: An interface may contain only the common ACL, only an interface ACL, or both the common and interface ACL.
{: .notice--info}


## Common ACL Implementation


## Resource Utilization






