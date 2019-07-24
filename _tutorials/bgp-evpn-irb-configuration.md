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
{% include toc %}
# Topic: Configure BGP-EVPN Integrated Routing and Bridging (IRB) for inter-subnet routing

EVPN Integrated Routing and Bridging (IRB) feature allows end hosts across the overlay to communicate with each other within the same subnet and across different subnets within the VPN. In this post we will cover the implementation of EVPN IRB to route between host-1 and host-9 being in different subnets. Distributed Anycast Gateway will be configured on Leaf-1 and Leaf-2 for subnet 10.0.0.0/24 and on Leaf-5 for subnet 20.0.0.0/24. 
After configuring IRB we will ping between the Hosts-1 and Host-9 to verify the reachability and observe the routes are learnt vie BGP EVPN. 

In [last post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/) we configured the Layer-2 stretch between Leaf-1, Leaf-2 and Leaf-5 using BGP EVPN EVI 10 for VLAN 10. We don’t need VLAN 10 on Leaf-5 for this post, that is why we will remove the EVI 10 from Leaf-5.

## Reference Topology:
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/reference-topology-evpn-post-4.png?raw=true)

