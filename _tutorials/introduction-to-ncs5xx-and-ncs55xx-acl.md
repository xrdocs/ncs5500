---
published: true
date: '2020-08-03 17:07 +0530'
title: Introduction to NCS5xx and NCS55xx ACL
author: Tejas Lad
excerpt: This is an introduction to ACL feature support on NCS5xx and NCS55xx portfolio
tags:
  - cisco
  - 'NCS5500 '
  - NCS 5500
  - ACL
  - NCS540
  - NCS560
position: hidden
---
{% include toc %}

## Introduction

This documents aims to introduce the ACL support on NCS500 and NCS product family. It will cover the overview of ACL, ACL Protcols, Types and Support Matrix. Later documents will deepdive into the ACL features and their use cases.

## Overview

![Screenshot 2020-08-03 at 6.03.20 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-03 at 6.03.20 PM.png)

Though everyone would be aware of what an access control list is, I would like to brush up some of the basics before deep-diving into its complex functionalities. 

ACL's can be considered as a ordered list of conditions used to test the network traffic that traverses through the router's interfaces. On the basis of the defined conditions, the router can decide which packets to accept and which to drop. ACL's help in managing the traffic and secure the acsess to and from the network.

ACL's can make permit/deny decisions based on source/destination address, source/destination ports, L3 protocols, L4 port numbers and many other criterias.

ACL's on NCS5xx and NCS55xx portfolio uses the Programmable Mapping and Filtering (PMF) functionality and TCAM (internal/exertnal) in both the Ingress Receive Packet Processing (IRPP) blocks and Egress Receive Packet Processing (ERPP) blocks. The NPU in these platforms are based on the Broadcom family of chipsets.  These chipsets uses a pipeline architecture which has dedicated hardware blocks for performing various functions. For more information on the product family please refer ([NCS5500 Modular Platforms](https://www.cisco.com/c/dam/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs5500-modular-platform-architecture-white-paper.pdf "NCS5500 Modular Platforms") [NCS5500 Fixed Platforms](https://www.cisco.com/c/dam/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs5500-fixed-platform-architecture-white-paper.pdf "NCS5500 Fixed Platforms") [NCS540](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-540-series-routers/datasheet-c78-740296.html "NCS540") [NCS560](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-560-series-routers/datasheet-c78-742029.html "NCS560"))

## Roles of ACL 

| Task                         | Use Cases                                                                                                                                                                                       |
|------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Increase Network Performance | ACL's limit the network traffic and help in reducing the network congestion For e.g.heavy traffic like videos can be filtered out if needed which can greatly  increase the network performance |
| Traffic Control              | Certain updates and control traffic can be filtered to preserve the bandwidth                                                                                                                   |
| Secure Access                | Different hosts can be allowed to access only required applications inside the network                                                                                                          |
| Traffic Filter               | ACL's can help to block certain traffic and permit the other. For e.g. email traffic can be allowed whereas Telnet traffic can be blocked                                                       |
| Redirect the traffic         | ACL's can be used to decide the next hop of incoming traffic if needed. e.g ACL based forwarding or ABF                                                                                         |

## High Level WorkFlow

As per the [reference](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/71x/b-ip-addresses-cg-ncs5500-71x/b-ip-addresses-cg-ncs5500-71x_chapter_0111.html "reference"), below is the high level work flow of ACL 

![Screenshot 2020-08-03 at 6.21.16 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-03 at 6.21.16 PM.png)



