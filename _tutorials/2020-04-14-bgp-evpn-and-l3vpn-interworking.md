---
published: false
date: '2020-04-14 16:35 -0700'
title: BGP-EVPN and L3VPN Interworking
author: Ahmad Bilal
excerpt: BGP-EVPN and L3VPN Interworking Support on NCS5500
---
{% include toc %}

## BGP-EVPN and L3VPN Interworking Support on NCS5500

In single-active multi-homing mode, only a single Leaf among a group of Leafs attached to a Host is allowed to forward the traffic to and from on a given VLAN.

In this post we will cover the BGP-EVPN based Single-Active Multi-Homing of CE/Hosts. Similar to Active/Active Multi-homing, Single-Active is also achieved by EVPN Ethernet Segment feature. Single-active offers redundant connectivity with forwarding for a VLAN on a single link at a time with failover to the second link in case of active link’s failure. This approach is very useful for network scenarios where policing, metering and billing are required. 

## Reference Topology:
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/reference-topology-evpn-single-active.png?raw=true)

For this post, we will leverage EVPN control-plane and ISIS Segment Routing based forwarding that we configured in a previous [post.](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-1/)

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/single-active-multi-homing.png?raw=true)

As shown in the above topology, Host-1 is multi-homed to Leaf-1 and Leaf-2. For EVPN single-active multi-homing, each link towards the Leaf will be in a unique ethernet bundle interface. As both the links are in separate ethernet bundles, the host H-1 will flood traffic at first to both the Leafs but only the designated forwarder (DF) Leaf will forward the traffic. As a result, the host will have only one ethernet bundle interface in its forwarding table to forward the traffic and achieve per VLAN single-active multi-homing. 