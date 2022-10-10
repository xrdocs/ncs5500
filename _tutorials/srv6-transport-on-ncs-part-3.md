---
published: true
date: '2022-09-23 10:06 +0530'
title: Implementing Layer2 VPN Over SRv6 Transport on NCS 5500/500
position: hidden
author: Paban Sarma
excerpt: >-
  L2VPN config and verification over SRV6 Transport on NCS 5500/500 series
  platforms
---
{% include toc icon="table" title="Table of Contents" %}

|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com)|  
|Tejas Lad, Technical Marketing Engineer (telad@cisco.com)|

## Overview

Until now, we covered [SRv6 Transport](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-1/) and [Layer3 VPN](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-2/) over SRv6 transport on NCS 5500 and NCS 500 platforms. In this tutorial, we will cover the impelementaion of EVPN based point-to-point (E-Line) L2 service (EVPN-VPWS) over SRv6. 

## Topology

![Screenshot 2022-08-16 at 12.53.39 PM.png]({{site.baseurl}}/images/Screenshot 2022-08-16 at 12.53.39 PM.png)

The topology used is a simple four node network comprising of Cisco NCS 540 and NCS 5500 series platforms. There are two CE nodes connected to PE1 and PE4 respectively to simulate customer networks. Details of each node along with Loopback IPs are mentioned in the below table.

| Nodes | Device Type | Software Version  |Loopback0   |
|-------|-------------|-------------------|------------|
| PE1   |  NCS 540    | IOS XR 7.5.2      |fcbb:bb00:1::1/128 |
| P2    |  NCS 5500   | IOS XR 7.5.2      |fcbb:bb00:2::1/128 |
| P3    |  NCS 5500   | IOS XR 7.5.2      |fcbb:bb00:3::1/128 |
| PE4   |  NCS 5500   | IOS XR 7.5.2      |fcbb:bb00:4::1/128 |


The loopback0 IPs are chosen as per the SRv6 addressing best practice (check out [segment-routing.net](https://www.segment-routing.net/) for more details). 

In this tutorial, we will establish a L2VPN (EVPN-VPWS) connecting CE1 and CE2. the example will demonstrate VLAN based E-Line (EVPL) service and establish L2 stretch across CE1 and CE2 for VLAN 100. 

## Configuration Steps
EVPN based P2P service over SRv6 transport will involve 3 steps, viz.
	- Establishing EVPN control plane  over BGP
    - Configuring l2transport CE-PE liks
    - Configutring EVPN EVI and L2VPN Service
    
### BGP Control Plane

Traditional L2 Services uses LDP for signalling, which is simplified by EVPN with the use of BGP for control plane operation. In our [previous tutorial](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-2/), we established BGP neighburship between PE1 and PE4 with VPNv4 AFI. Now we need to enable EVPN AFI over BGP. Below snippet shows full BGP configuration needed for L2 Service over SRv6.

<code>
  BGP config snippet
</code>


### Layer2 attachment circuits

### Configuring EVPN and L2VPN Service

## Verifiation Steps
