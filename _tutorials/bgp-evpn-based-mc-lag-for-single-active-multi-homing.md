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

