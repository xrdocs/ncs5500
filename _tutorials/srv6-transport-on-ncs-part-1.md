---
published: true
date: '2022-08-16 11:27 -0700'
title: Segment Routing v6 (SRv6) Transport on NCS5500/NCS500 Platforms - Part 1
author: Tejas Lad
excerpt: >-
  This is the first document in the series which will focus on SRv6 basics and
  underlay transport bringup
tags:
  - iosxr
  - NCS5500
  - SRv6
  - Segment Routing v6
  - NCS500
  - NCS5700
position: hidden
---
{% include toc icon="table" title="Table of Contents" %}

|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com)|  
|Tejas Lad, Technical Marketing Engineer (telad@cisco.com)|

## Introduction

This is the first document of the series focussing on SRv6 transport. In this document we will focus on SRv6 basics and understand how to bring up SRv6 transport on Cisco NCS 500 and NCS 5500 platforms. In the subsequent tutorials, we will cover more topics related to SRv6 transport showing implementation of layer2 and layer3 services, QoS behaviour, Traffic-Engineering etc.

## Brief Background
The Service Provider transport network has evolved recently to provide converged transport for various 5G applications and use cases. As we already know, Segment Routing (SR) brings in programmability to the transport network using the MPLS data plane. Whereas Segment Routing v6 (SRv6) uses IPv6 instead of MPLS. This brings more simpler way to build a programmable transport where we can easily do network slicing and traffic engineering for various services. This advanced series of documents will demonstrate, new SRv6 transport on the IOS XR based NCS 5500/500 routers and how to configure and implement various overlay services like L3VPN/BGP-EVPN based E-Line service using the underlay. In this document series, we will be using SRv6 Transport with ISIS as the IGP and provision end-to-end L3/L2 services over the transport. Service SLA and QoS being one of the important aspects of any services, we will also explore how end-to-end QoS can be managed for Services over SRv6 on these routers.

Note: This document is to familiarise users with the technology. The configurations and lab setup can be taken as a reference, and this by no means represent a real production network.
{: .notice--info}

## SRv6 Terminology

Before starting with the topology and the configurations, let us brush up some of the important terminologies w.r.t SRv6. In this series, we will use the SRv6 uSID implementation. The SRv6 micro-segment (uSID) is an extension of the SRv6 architecture. It leverages the SRv6 Network Programming architecture to encode several SRv6 Micro-SID (uSID) instructions within a single 128-bit SID address. (In SRv6, a SID represents a 128-bit value) Such a SID address is called a uSID Carrier. For further information on SRv6 usid please [visit](https://www.ietf.org/id/draft-filsfils-spring-net-pgm-extension-srv6-usid-13.html).   

| SID components | Details                                                                                                                                                                                                    |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Locator        | This is the first part of the SID with most significant bits and represents an address of a specific SRv6 node                                                                                             |
| Function       | This is the portion of the SID that is local to the owner node and designates a specific SRv6 function (network instruction) that is executed locally on a particular node, specified by the locator bits. |
| Args           | This field is optional and represents optional arguments to the function.                                                                                                                                  |

The locator part can be further divided into two parts:

| Locator Components | Details                                                                                                                                                                            |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SID Block          |  						  							  								 This field is the SRv6 network designator and is a fixed or known address space for an SRv6 domain. This is the most significant bit (MSB) portion of a locator  subnet.  							 						 					 |
| Node Id            | This field is the node designator in an SRv6 network and is the least significant bit (LSB) portion of a locator subnet.                                                           |

For understanding the technology in details and the latest enhancements, please visit the following [page](https://www.segment-routing.net/)


## Reference Topology

![Screenshot 2022-08-16 at 12.53.39 PM.png]({{site.baseurl}}/images/Screenshot 2022-08-16 at 12.53.39 PM.png)

| Nodes | Loopback    |
|-------|-------------|
| PE1   | 2001::1/128 |
| P2    | 2001::2/128 |
| P3    | 2001::3/128 |
| PE4   | 2001::4/128 |

## Configuration Steps
To bring up SRv6 transport, the very first task needed to perform is making the underlay IGP ready. We will be using ISIS as the underlay IGP protocol to bring up IPv6 connecticity across the nodes. Once, the network is ready with IGP, there are three steps to enable SRv6, i.e.

- enabling platform hw-module profile
- configuring SRv6 locator
- Enabling SRv6 over ISIS

### Configuring ISIS for reachability

<div class="highlighter-rouge">
<pre class="highlight">
<code>
router isis 1
 is-type level-2-only
 net 49.0000.0000.0001.00 
 address-family ipv6 unicast
  metric-style wide 
 !
 interface Bundle-Ether12 
 point-to-point 
 address-family ipv6 unicast
  fast-reroute per-prefix
  fast-reroute per-prefix ti-lfa 
 !
!
 interface Bundle-Ether13
 point-to-point 
 address-family ipv6 unicast
 fast-reroute per-prefix
 fast-reroute per-prefix ti-lfa 
 !
!
 interface Loopback0
 address-family ipv6 unicast
 ! 
 !
!
</code>
</pre>
</div>

The above configuration is for router PE1. Similarly configure the IGP ISIS on all the other routers P2, P3 and PE4.

### Enabling SRv6

#### Platform hw-module profile
To start with, we will configure the hw-module profile on all the routers.

<div class="highlighter-rouge">
<pre class="highlight">
<code> 
RP/0/RP0/CPU0:PE1#show running-config | in hw-module 
Building configuration...
<mark>hw-module profile segment-routing srv6 mode micro-segment format f3216</mark>
</code>
</pre>
</div> 
Note: Configure the same on all the routers.

The reason for configuring the above is because, data plane in NCS500 and NCS5500 needs to be explicitly enabled for SRv6 with specific mode i.e. whether SRv6 Base or SRv6 micro-segment (uSID). We are using uSID based SRv6 transport and this is done by configuring the  hardware module profiles:“hw-module profile segment-routing srv6 mode micro-segment format f3216”. The hw-module profile is kind of self-explanatory, we have enabled segment routing v6 (srv6) and the mode used is micro-segment. (another mode for SRv6 is the base mode). The uSID carrier format used is f3216, i.e 32 bit block size and 16bit uSID. Thus a single DA can carry upto 6 micro-instructions or uSID.

Note: This hardware-module profile configuration needs reload of the router.
{: .notice--info}



#### Configuring SRv6 locators

As discussed above, we need to configure f3216 format locator. So for each node we will configure /48 locator of which first 32 bits will be SID block and remaining 16 bits will be Node ID. 

| PE1                                                                                                                                                                                                                 | P2                                                                                                                                                                                                                   | P3                                                                                                                                                                                                              | PE4                                                                                                                                                                                                             |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| segment-routing<br> srv6<br>  encapsulation <br>  <br>  source-address 2001::1<br> !<br> locators<br>  locator POD0<br>   micro-segment behavior unode psp-usd<br>   prefix fcbb:bb00:1::/48<br>  !<br> <br> !<br>! | segment-routing<br> srv6<br>  encapsulation<br><br>  source-address 2001::2<br> !<br> locators<br>  locator POD0<br>   micro-segment behavior unode psp-usd<br>   prefix fcbb:bb00:2::/48<br>  !<br><br> !<br>!<br>  | segment-routing<br> srv6<br><br>  encapsulation<br>  source-address 2001::3<br> !<br> locators<br>  locator POD0<br>   micro-segment behavior unode psp-usd<br>   prefix fcbb:bb00:3::/48<br><br>  !<br> !<br>! | segment-routing<br> srv6<br><br>  encapsulation<br>  source-address 2001::4<br> !<br> locators<br>  locator POD0<br>   micro-segment behavior unode psp-usd<br>   prefix fcbb:bb00:4::/48<br><br>  !<br> !<br>! |

#### Enabling SRv6 over ISIS

<div class="highlighter-rouge">
<pre class="highlight">
<code>
router isis 1 
 address-family ipv6 unicast
  segment-routing srv6 
   <mark>locator POD0</mark>
   !
  ! 
 !
!
</code>
</pre>
</div>

Configure the above on all the routers and Thats it !!!!! You are done with the SRv6 Underlay Transport 


## Verification

You can use a few verification commands to check the SRv6 transport. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:PE1#show segment-routing  srv6 locator 
Name                  ID       Algo  Prefix                    Status   Flags   
--------------------  -------  ----  ------------------------  -------  --------
<mark>POD0                  2        0     fcbb:bb00:1::/48          Up       U</mark> 
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:PE1#show segment-routing  srv6 locator POD0 detail 
Name                  ID       Algo  Prefix                    Status   Flags   
--------------------  -------  ----  ------------------------  -------  --------
POD0                  2        0     fcbb:bb00:1::/48          Up       U       
  <mark>(U): Micro-segment (behavior: uN (PSP/USD))</mark>
  Interface: 
    Name: srv6-POD0
    IFH : 0x2000800c
    IPv6 address: fcbb:bb00:1::/48
  Number of SIDs: 4
  Created: Apr 21 08:34:45.886 (1w1d ago)
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:PE1#show segment-routing srv6 sid 

<mark>*** Locator: 'POD0' ***</mark> 

SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  ------------------------------    ------------------  -----  --
fcbb:bb00:1::               uN (PSP/USD)      'default':1                       sidmgr              InUse  Y 
fcbb:bb00:1:e001::          uA (PSP/USD)      [BE12, Link-Local]:0              isis-1              InUse  Y 
fcbb:bb00:1:e002::          uA (PSP/USD)      [BE13, Link-Local]:0              isis-1              InUse  Y 
</code>
</pre>
</div>

## Conclusion

This concludes the Part 1 of the this document series. Stay tuned for the next article.
