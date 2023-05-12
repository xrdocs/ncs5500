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

### BGP configuration for EVPN
BGP configuration is simplar to what we did in our previous tutorial. However, since we have multiple PE nodes here, we need to establish full mesh BGP with EVPN AFI here. For simplicity, we are using P2 as a route-reflector (In real time deployment, it is recomended to use dedicated route reflectors in the netwrok). The following config snippet shows the BGP confifuration on all the PEs and the RR node.

_**PE1**_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
router bgp 100
 bgp router-id 1.1.1.1
 <mark>address-family l2vpn evpn</mark>
 !
 neighbor fcbb:bb00:2::1
  remote-as 100
  update-source Loopback0
  <mark>address-family l2vpn evpn</mark>
  !
 !
!
</code>
</pre>
</div>

_**PE4**_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
router bgp 100
 bgp router-id 4.4.4.4
 <mark>address-family l2vpn evpn</mark>
 !
 neighbor fcbb:bb00:2::1
  remote-as 100
  update-source Loopback0
 <mark>address-family l2vpn evpn</mark>
  !
 !
!
</code>
</pre>
</div>

_**PE5**_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
router bgp 100
 bgp router-id 5.5.5.5
 <mark>address-family l2vpn evpn</mark>
 !
 neighbor fcbb:bb00:2::1
  remote-as 100
  update-source Loopback0
  <mark>address-family l2vpn evpn</mark>
  !
 !
!
</code>
</pre>
</div>

_**P2 as RR**_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
router bgp 100
 bgp router-id 2.2.2.2
 address-family vpnv4 unicast
 !
 address-family l2vpn evpn
 !
 neighbor fcbb:bb00:1::1
  remote-as 100
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
  address-family l2vpn evpn
   route-reflector-client
  !
 !
 neighbor fcbb:bb00:4::1
  remote-as 100
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
  address-family l2vpn evpn
   route-reflector-client
  !
 !
 neighbor fcbb:bb00:5::1
  remote-as 100
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
  address-family l2vpn evpn
   route-reflector-client
  !
 !
!
</code>
</pre>
</div>

### EVPN ES and EVI configuration

The next step is to configure the EVPN. It includes three steps,
- ES configuration: Since we are not using Multihoming CE, we won't explictely configure any ESI but simply enable to physical PE-CE link under EVPN  
<div class="highlighter-rouge">
<pre class="highlight">
<code>
evpn
 interface  TenGigE0/0/0/0
  <mark> ethernet-segment
    identifier type 0  1.1.1.1.1.1.0 </mark>
  !
 !
!
</code>
</pre>
</div>

- Enabling SRv6: we need to globally enable SRv6 for EVPN under EVPN global configuration. This steps optionally includes specifying the SRv6 locator to be used for EVPN services.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
evpn
 segment-routing srv6
  <mark>locator  locator-name </mark>
 !
!
</code>
</pre>
</div>

- EVI configuartion: The next important step is to configure the EVPN identifier (EVI) and enable mac advertisement and SRv6 for the EVI. we can also specify the locator per EVI in this stage.  The configuration explained in the tutorial we are using per EVI locator.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
evpn
 evi 200 segment-routing srv6
  advertise-mac
  !
  <mark> locator POD0</mark>
 !
!
</code>
</pre>
</div>

Below is the config snippet from all the PE nodes. (we have the exact same configuration on all three as the topology is symmetric i.e same interfaces are used on each node)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
evpn
 evi 200 segment-routing srv6
  advertise-mac
  !
  locator POD0
 !
 interface TenGigE0/0/0/0
 !
 segment-routing srv6
 !
!
</code>
</pre>
</div>
## Verification Steps

## Summary
