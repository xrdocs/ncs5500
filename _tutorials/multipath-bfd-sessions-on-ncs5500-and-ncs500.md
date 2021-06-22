---
published: true
date: '2021-06-22 12:12 +0530'
title: Multipath BFD sessions on NCS5500 and NCS500
author: Tejas Lad
excerpt: >-
  This document covers the implementation of BLB and BFD over BVI on NCS5500 and
  NCS500
tags:
  - iosxr
  - NCS5500
  - NCS500
  - NCS 5500
  - BFD
  - BLB
  - BVI
position: hidden
---
{% include toc icon="table" title="Multipath BFD sessions on NCS5500 and NCS500" %} 

## Introduction

In our previous article we discussed [BFD over Bundle Interface or BoB](https://xrdocs.io/ncs5500/tutorials/bfd-over-bundle-interfaces-on-ncs5500-and-ncs500/). In this article, we discuss one more concept for bundle: BFD over Logical Bundle or BLB. We will understand the difference between BLB and BoB and see its use cases. We will also see Multipath and Multihop Sessions with BFD over BVI interfaces.

## BFD over Logical Bundle

With IOS-XR 7.1.1, we support BLB on NCS5500 and NCS500 platforms. The Bidirectional Forwarding Detection (BFD) over Logical Bundle feature implements and deploys BFD over bundle interfaces based on RFC 5880. This is the fundamental difference between BLB and BoB. In the former the bundle interface is a single interface, whereas in the later we implement BFD per member link. BLB is a multipath (MP) single-hop session. BLB requires limited knowledge of the bundle interfaces on which the sessions run, this is because BFD treats the bundle as one big pipe. To function, [BLB](https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/asr9k-r6-2/routing/configuration/guide/b-routing-cg-asr9000-62x/b-routing-cg-asr9000-62x_chapter_0100.html#concept_45D0EC68B0A241B39338087FEB10055F) requires only information about IP addresses, interface types, and caps on bundle interfaces. Information such as list of bundle members, member states, and configured minimum or maximum bundle links are not required. In case of BLB, BFD client is not Bundle link but protocols running over bundle link. BLB is supported on IPv4 and IPv6 addresses and IPv6 link-local address.  

| BFD Session Type              | BFD Client                        |
|-------------------------------|-----------------------------------|
| BFD over Bundle - BoB         | bundlemgr                         |
| BFD over Logical Bundle - BLB | protocols running over the bundle |






