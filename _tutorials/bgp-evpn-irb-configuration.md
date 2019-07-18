---
published: false
date: '2019-07-18 17:19 +0100'
title: BGP-EVPN IRB Configuration for Inter-Subnet Routing
author: Ahmad Bilal Siddiqui
excerpt: In this post we will cover the EVPN-IRB configuration on NCS5500
tags:
  - iosxr
  - ncs 5500
  - evpn
  - NCS5500
---
# Topic: Configure BGP-EVPN Integrated Routing and Bridging (IRB) for inter-subnet routing

In the [last post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-2/), we configured the BGP-EVPN based Multi-homing of host/CE using EVPN Ethernet Segment. In this post, we will provision BGP-EVPN based Layer-2 VPN service between the Leafs. The EVPN Layer-2 service will enable forwarding between host-1 and host-5 which are part of the same subnet.
