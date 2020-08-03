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



