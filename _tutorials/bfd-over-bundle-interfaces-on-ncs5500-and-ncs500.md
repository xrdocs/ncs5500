---
published: true
date: '2021-06-18 10:53 +0530'
title: BFD over Bundle Interfaces on NCS5500 and NCS500
author: Tejas Lad
excerpt: This document explains the BFD over Bundle Interfaces on NCS5500 and NCS500
position: hidden
tags:
  - iosxr
  - NCS5500
  - NCS500
  - BFD
  - BoB
  - Bundle
---
{% include toc icon="table" title="BFD over Bundle Interfaces on NCS55xx and NCS5xx" %} 

## Introduction

We started the NCS5500 and NCS500 BFD technotes with [BFD architecture](https://xrdocs.io/ncs5500/tutorials/bfd-architecture-on-ncs5500-and-ncs500/#:~:text=BFD%20Feature%20Support,-BFD%20is%20supported&text=Static%2C%20OSPF%2C%20BGP%20and%20IS,are%20supported%20in%20IPv4%20BFD.&text=BFD%20over%20BVI%20is%20supported,dampening%20for%20IPv4%20is%20supported.). Then we looked a bit deeper with the [Hardware Programming](https://xrdocs.io/ncs5500/tutorials/understanding-the-bfd-hardware-programming-on-ncs55xx-and-ncs5xx/). We confirmed the behaviour as per [RFC 5880](https://datatracker.ietf.org/doc/html/rfc5880). We also checked the OAM engine programming in the hardware and saw a few verification commands for quick verification. In this tech-note we will discuss the BFD over Bundles and BFD over Logical Bundles. What are their use cases and see the programming and behaviour w.r.t [RFC 7130](https://datatracker.ietf.org/doc/html/rfc7130) and [RFC 5883](https://datatracker.ietf.org/doc/html/rfc5883).

## Background


![Screenshot 2021-06-18 at 11.18.09 AM.png]({{site.baseurl}}/images/Screenshot 2021-06-18 at 11.18.09 AM.png)

Just a quick backgroun before we move. Link Bundle or Bundle Ethernet has been in the industry for a long time now. [Link Bundle](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/61x/b-ncs5500-interfaces-configuration-guide-61x/b-ncs5500-interfaces-configuration-guide-61x_chapter_0101.html) is simply a group of ports that are bundled together and act as a single link. The Link Bundling feature allows you to group multiple point-to-point links together into one logical link and provide higher bidirectional bandwidth, redundancy, and load balancing between two routers. A virtual interface is assigned to the bundled link. Cisco IOS XR software supports the IEEE 802.3ad—Standard technology that employs a Link Aggregation Control Protocol (LACP) to ensure that all the member links in a bundle are compatible. The advantages of link bundles are as follows:

  - Multiple links can span several line cards to form a single interface. Thus, the failure of a single link does not cause a loss of connectivity.
  - Bundled interfaces increase bandwidth availability, because traffic is forwarded over all available members of the bundle. Therefore, traffic can flow on the available links if one of the links within a bundle fails. 
  - Bandwidth can be added without interrupting packet flow.
  


## Why do we need BFD over Bundle - BoB ?

As we know, LACP allows a network device to negotiate an automatic bundling of links by sending LACP packets to their directly connected peer. LACP has keep-alive mechanism for link member. The default being 30s and it can be configurable to around 1s. Link Aggregation Control Protocol can
detect failures on a per-physical-member link. However the LACP timers do not fulfill the criteria of today's fast convergence requirements. Therefore using BFD for failure detection would([RFC 7130](https://datatracker.ietf.org/doc/html/rfc7130)): 

  - Provide a faster detection
  - Provide detection in the absence of LACP
  - Would be able to verify the ability for each member link to be able to forward L3 packets.

Running a single BFD session over the aggregation without internal knowledge of the member links would make it impossible for BFD to guarantee detection of the physical member link failures. The goal is to verify link Continuity for every member link.


## BFD over Bundle - BoB on NCS5500 and NCS500 

BFD over Bundle(BoB) implementation on NCS5500 and NCS500 is a standard based fast failure detection of link aggregation (LAG) memberlinks that is interoperable between different platforms. There are 2 modes available: **Cisco and IETF**. NCS5500 and NCS500 only supports the IETF mode. For BFD over Bundle, the BFD client is **bundlemgr**. Hence if BFD session goes down, bundlemgr will bring down the bundle if it violates the minimum link criteria. 

## Configuring a BFD session over a Bundle Interface
