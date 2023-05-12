---
published: false
date: '2023-05-12 09:03 +0530'
title: Implementing EVPN ELAN over SRv6 Transport on NCS 500/5500
author: Paban Sarma
---
## Overview

In our previous tutorials, covered SRv6 Transport with uSID on the NCS 500 and 5500 platforms, and L3/L2 P2P services on top of it. This tutorial will cover, implementaion of Ethernet VPN based multipoint layer 2 service (ELAN) over the SRv6 uSID transport. As of today, only Single homed EVPN ELAN is supported on these platform.

## Topology


| Nodes | Device Type | Software Version  |Loopback0   |
|-------|-------------|-------------------|------------|
| PE1   |  NCS 540    | IOS XR 7.5.2      |fcbb:bb00:1::1/128 |
| P2    |  NCS 5500   | IOS XR 7.5.2      |fcbb:bb00:2::1/128 |
| P3    |  NCS 5500   | IOS XR 7.5.2      |fcbb:bb00:3::1/128 |
| PE4   |  NCS 5500   | IOS XR 7.5.2      |fcbb:bb00:4::1/128 |
| PE5   |  NCS 5500   | IOS XR 7.5.2      |fcbb:bb00:5::1/128 |


The loopback0 IPs are chosen as per the SRv6 addressing best practice (check out [segment-routing.net](https://www.segment-routing.net/) for more details). 

In this tutorial, we will establish a multipoint L2VPN (EVPN-ELAN) connecting CE1, CE2 and CE3. the example will demonstrate VLAN based E-LAN (EVPLAN) service and establish L2 stretch across CE1 CE2 and CE3 for VLAN 200.

## Configuration Steps
We already covered the configuration steps for the transport in our previous tutorial. The below table summerizes the SRv6 uSID locator used (name POD0) on each Node for reference. 

| Nodes | SRv6 Locator         |
|-------|----------------------|
| PE1   |  fcbb:bb00:1::/48    |
| P2    |  fcbb:bb00:2::/48    |
| P3    |  fcbb:bb00:3::/48    |
| PE4   |  fcbb:bb00:4::/48    |
| PE5   |  fcbb:bb00:5::/48    |

Configuration steps included in this tutorial will focus only on  the service specific tasks including,

- BGP EVPN control plane 
- EVPN ES and EVI configuration
- Layer 2 UNI and L2VPN configuration

## Verification Steps

## Summary
