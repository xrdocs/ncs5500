---
published: true
date: '2022-09-01 08:46 +0530'
title: Layer 3 VPN over SRv6 Transport on NCS 5500/500
position: hidden
author: Paban Sarma
excerpt: >-
  L3VPN configuration and verification over SRv6 Transport on NCS 500 and NCS
  5500 platforms
---
{% include toc icon="table" title="Table of Contents" %}

|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com)|  
|Tejas Lad, Technical Marketing Engineer (telad@cisco.com)|

## Overview
In Previous Artcile, we discussed how to setup a segment routing v6 (SRv6) transport on the NCS 500 and NCS 5500 platforms. In this article, we will explore setting up layer 3 services over the SRv6 transport. 

## Topology

![Screenshot 2022-08-16 at 12.53.39 PM.png]({{site.baseurl}}/images/Screenshot 2022-08-16 at 12.53.39 PM.png)

The topology used is a simple four node network comprising of Cisco NCS 540 and NCS 5500 series platforms. There are two CE nodes connected to PE1 and PE4 respectively to simulate customer networks. Details of each node along with Loopback IPs are mentioned in the below table.

| Nodes | Device Type | Software Version  |Loopback0   |
|-------|-------------|-------------------|------------|
| PE1   |  NCS 540    | IOS XR 7.5.2      |2001::1/128 |
| P2    |  NCS 5500   | IOS XR 7.5.2      |2001::2/128 |
| P3    |  NCS 5500   | IOS XR 7.5.2      |2001::3/128 |
| PE4   |  NCS 5500   | IOS XR 7.5.2      |2001::4/128 |

## Configuring & Verifying VPNv4
## Configuring & Verifying VPNv6
## Conclusion
