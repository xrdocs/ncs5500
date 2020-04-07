---
published: false
date: '2020-04-06 21:24 -0700'
title: BGP-EVPN based MC-LAG for Single-Active Multi-Homing
author: Ahmad Bilal
excerpt: >-
  Implementation and configuration of BGP-EVPN based MC-LAG for Single-Active
  Multi-Homing on NCS 5500.
tags:
  - iosxr
  - cisco
  - EVPN
  - NCS 5500
---
{% include toc %}

## Implementation of BGP-EVPN based MC-LAG for Single-Active Multi-Homing

In single-active multi-homing mode, only a single Leaf among a group of Leafs attached to a Host is allowed to forward the traffic to and from on a given VLAN.

In this post we will cover the BGP-EVPN based Single-Active Multi-Homing of CE/Hosts. Similar to Active/Active Multi-homing, Single-Active is also achieved by EVPN Ethernet Segment feature. Single-active offers redundant connectivity with forwarding for a VLAN on a single link at a time with failover to the second link in case of active link’s failure. This approach is very useful for network scenarios where policing, metering and billing are required. 

## Reference Topology:
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/reference-topology-evpn-single-active.png?raw=true)

For this post, we will leverage EVPN control-plane and ISIS Segment Routing based forwarding that we configured in a previous [post.](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-1/)

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/single-active-multi-homing.png?raw=true)

As shown in topology, Host-1 is multi-homed to Leaf-1 and Leaf-2. For EVPN single-active multi-homing, each link towards the Leaf will be in a unique ethernet bundle interface. As both the links are in separate ethernet bundles, the host H-1 will flood traffic at first to both the Leafs but only the designated forwarder (DF) Leaf will forward the traffic. As a result, the host will have only one ethernet bundle interface in its forwarding table to forward the traffic and achieve per VLAN single-active multi-homing. 

### Task 1: Configure Ethernet bundles on Host-1 for multi-homing

As per the reference topology Host-1 is multi-homed to Leaf-1 and Leaf-2 via LACP bundle-ethernet 11 going to Leaf-1 and bundle-ethernet 12 going to Leaf-2. ASR9K is acting as the host/CE with IP address 10.0.0.10/24 configured on a BVI. Following is the configuration of LAG on Host-1, static route is configured to reach to EVPN-Anycast Gateway on the Leafs. The LAG on Host-1 will come up after we configure single-active multi-homing using EVPN Ether-Segment on the Leaf-1 and Leaf-2.

**Note:** In this post we will configure VLAN 10 to show the single-active behavior. Configuration of VLAN 20 is out of scope for this post. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>

Host-1

interface Bundle-Ether 11
 description "Bundle to Leaf-1"
!
interface TenGigE0/0/2/0
 description "Link to Leaf-1 ten0/0/0/47"
 bundle id 11 mode active
!
interface Bundle-Ether11.10 l2transport
 encapsulation dot1q 10
 rewrite ingress tag pop 1 symmetric
!
interface Bundle-Ether 12
 description "Bundle to Leaf-2"
!
interface TenGigE0/0/2/1
 description "Link to Leaf-2 ten0/0/0/47"
 bundle id 12 mode active
!
interface Bundle-Ether12.10 l2transport
 encapsulation dot1q 10
 rewrite ingress tag pop 1 symmetric
!
interface BVI10
 ipv4 address 10.0.0.10 255.255.255.0
!
router static
 address-family ipv4 unicast
  0.0.0.0/0 10.0.0.1
 !
!
l2vpn
 bridge group bg1
  bridge-domain bd-10
   interface Bundle-Ether11.10
   !
   interface Bundle-Ether12.10
   !
   routed interface BVI10
 !
!
</code>
</pre>
</div>
