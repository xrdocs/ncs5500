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

In this tutorial, we will establish a L3VPN (VPNv4 & VPNv6) connecting two subnets across CE1 and CE2. 

## Configuration & Verification for VPNv4 
### Configuring BGP Control Plane
At first, we will setup the BGP control palne with VPNv4 address family between PE1 and PE2. We are directly peering between PE1 and PE4 in this example, however in a real network there can be route reflectors used for BGP to improve simplicity and scalability. 

#### BGP configuration on PE1

<div class="highlighter-rouge">
<pre class="highlight">
<code>
router bgp 100
 bgp router-id 1.1.1.1
 address-family vpnv4 unicast
 !
 neighbor 2001::4
  remote-as 100
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
!
</code>
</pre>
</div>

#### BGP configuration on PE4

<div class="highlighter-rouge">
<pre class="highlight">
<code>
router bgp 100
 bgp router-id 4.4.4.4
 address-family vpnv4 unicast
 !
 neighbor 2001::1
  remote-as 100
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
!
</code>
</pre>
</div>

we can verify the BGP neighbourship with VPNv4 AFI using `show bgp vpnv4 unicast summary`
<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:PE4#show  bgp  vpnv4 unicast summary 
Thu Sep  1 08:12:21.733 UTC
BGP router identifier 4.4.4.4, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0
BGP main routing table version 1
BGP NSR Initial initsync version 1 (Not Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker               1          1          1          1           1           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001::1           0   100       3       3        1    0    0 00:00:07          0
</code>
</pre>
</div>


### Configuring VRF and PE-CE links
### Configuring VRF under BGP
### Verification of VPNv4

## Configuration & Verification for VPNv6 
### Configuring BGP Control Plane
### Configuring VRF and PE-CE links
### Configuring VRF under BGP
### Verification of VPNv6

## Summary
