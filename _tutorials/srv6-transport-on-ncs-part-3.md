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

<h2> Overview </h2>

Until now we covered setting up, [SRv6 Transport](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-1/) and bringing up [Layer3 VPN](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-2/) using that on NCS 5500 and NCS 500 platforms. In this tutorial, we will cover the impelementaion of EVPN based point-to-point (E-Line) L2 service (EVPN-VPWS) over SRv6. 

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
	- Establishing EVPN control plane over BGP
    - Configuring l2transport between CE-PE links
    - Configuring EVPN EVI and L2VPN Service
    
### BGP Control Plane

Traditional L2 services uses LDP for signalling, which is simplified by EVPN with the use of BGP for control plane operation. In our [previous tutorial](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-2/), we established BGP neighborship between PE1 and PE4 with VPNv4 AFI. Now we need to enable EVPN AFI over BGP. Below snippet shows full BGP configuration needed for layer2 service over SRv6.

_**PE1**_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
router bgp 100
 bgp router-id 1.1.1.1
 <mark>address-family l2vpn evpn</mark>
 !
 neighbor fcbb:bb00:4::1
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
 neighbor fcbb:bb00:1::1
  remote-as 100
  update-source Loopback0
 <mark>address-family l2vpn evpn</mark>
  !
 !
!
</code>
</pre>
</div>

### Configuring Layer2 Attachment Circuits

We need to configure l2transport sub-interface (on the PE-CE link) with appropriate VLAN encapsulations. This tutorial is showing VLAN based service with VLAN ID 100. We are not showing any VLAN translation operation (rewrite commands) as the are out of scope of this tutorial. 

_**PE1 and PE4**_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
interface TenGigE0/0/0/0.2 l2transport
 encapsulation dot1q 2
</code>
</pre>
</div>

### Configuring EVPN and L2VPN Service

Next step is to configure EVPN and L2VPN service construct on both the PE. since we have a symmetric topology, our configuration on both node will be similar. Configure the below on PE1 and PE4.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
evpn
 <mark>interface TenGigE0/0/0/0</mark>
 !
 segment-routing srv6
  locator POD0
 !
!
l2vpn
 xconnect group 2
  p2p 2
   interface TenGigE0/0/0/0.2
   neighbor evpn evi 2 service 2 segment-routing srv6
   !
  !
 !
!
</code>
</pre>
</div>

The interface under EVPN configuration doesn't have any ESI configured, this is because of single homed service and default ESI being used. For detailed understanding on evpn configuration and modes refer [e-evpn.io](https://e-vpn.io/).
We have globally enabled srv6 locator POD0 under evpn, this means l2vpn SIDs (UDX2) will be allocated from the same locator. The srv6 configuration under l2vpn xconnect group service construct can be used to override the global evpn configuration and assign new locator.

## Verifiation Steps

At first we will verify that the layer2 P2P service is up,

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:LABSP-3393-PE1#show l2vpn xconnect 
Legend: ST = State, UP = Up, DN = Down, AD = Admin Down, UR = Unresolved,
        SB = Standby, SR = Standby Ready, (PP) = Partially Programmed,
        LU = Local Up, RU = Remote Up, CO = Connected, (SI) = Seamless Inactive

XConnect                   Segment 1                       Segment 2                
Group      Name       ST   Description            ST       Description            ST    
------------------------   -----------------------------   -----------------------------
<mark>2          2          UP   Te0/0/0/0.2            UP       EVPN 2,2,::ffff:10.0.0.2 
                                                                                  UP    </mark>
----------------------------------------------------------------------------------------
</code>
</pre>
</div>

We can also see, the SRv6 uDX2 SID assigned to each segment of the service in the detailed show command below:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:LABSP-3393-PE1#show  l2vpn xconnect group 2 detail 
Mon Nov 14 04:53:49.165 UTC

Group 2, XC 2, state is up; Interworking none
  AC: TenGigE0/0/0/0.2, state is up
    Type VLAN; Num Ranges: 1
    Rewrite Tags: []
    VLAN ranges: [2, 2]
    MTU 1504; XC ID 0x2; interworking none
    Statistics:
      packets: received 0, sent 0
      bytes: received 0, sent 0
      drops: illegal VLAN 0, illegal length 0
  EVPN: neighbor ::ffff:10.0.0.2, PW ID: evi 2, ac-id 2, state is up ( established )
    XC ID 0xc0000002
    Encapsulation SRv6
    Encap type Ethernet
    Ignore MTU mismatch: Enabled
    Transmit MTU zero: Enabled
    Reachability: Up

      SRv6              Local                        Remote                      
      ----------------  ---------------------------- --------------------------
      <mark> uDX2              fcbb:bb00:1:e006::           fcbb:bb00:4:e006::</mark>
      AC ID             2                            2                           
      MTU               1518                         0                           
      Locator           POD0                         N/A                         
      Locator Resolved  Yes                          N/A                         
      SRv6 Headend      H.Encaps.L2.Red              N/A                         
    Statistics:
      packets: received 0, sent 0
      bytes: received 0, sent 0
      
 </code>
</pre>
</div>

The same SID information is updated in the SRv6 SID table as well.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:LABSP-3393-PE1#show  segment-routing srv6  locator POD0 sid 
SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  ------------------------------    ------------------  -----  --
fcbb:bb00:1::               uN (PSP/USD)      'default':1                       sidmgr              InUse  Y 
fcbb:bb00:1:e001::          uA (PSP/USD)      [BE12, Link-Local]:0:P            isis-1              InUse  Y 
fcbb:bb00:1:e002::          uA (PSP/USD)      [BE12, Link-Local]:0              isis-1              InUse  Y 
fcbb:bb00:1:e003::          uA (PSP/USD)      [BE13, Link-Local]:0:P            isis-1              InUse  Y 
fcbb:bb00:1:e004::          uA (PSP/USD)      [BE13, Link-Local]:0              isis-1              InUse  Y 
fcbb:bb00:1:e005::          uDT4              '1'                               bgp-100             InUse  Y 
<mark>fcbb:bb00:1:e006::          uDX2              2:2                               l2vpn_srv6          InUse  Y </mark>

 </code>
</pre>
</div>

The SID details and functions can also be verified using SID details CLI as shown below. It shows that the SID function is 0xe0006 and it is in the context of EVPN EVI 2 with AC IDs 2 (eth-tag=2).

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:LABSP-3393-PE1#show  segment-routing srv6 sid fcbb:bb00:1:e006::  detail 


*** Locator: 'POD0' *** 

SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  ------------------------------    ------------------  -----  --
fcbb:bb00:1:e006::          uDX2              2:2                               l2vpn_srv6          InUse  Y 
  <mark>SID Function: 0xe006</mark>
  <mark>SID context: { evi=2, eth-tag=2 }</mark>
  <mark>Locator: 'POD0'</mark>
  Allocation type: Dynamic
  Created: Nov 14 04:49:43.505 (00:08:54 ago)
  
 </code>
</pre>
</div>

Finally, to verify the data plane operation we will initiate ICMP ping from CE1 to CE2. we already have configured CE1 & CE2 in the same subnet and established the L2 stretch between the two nodes with EVPN-VPWS over SRv6 transort. 
<div class="highlighter-rouge">
<pre class="highlight">
<code>

<mark>RP/0/RP0/CPU0:CE1</mark>#show run int tenGigE 0/0/0/0.2
interface TenGigE0/0/0/0.2
 ipv4 address 192.2.0.1 255.255.255.0
 encapsulation dot1q 2
!

<mark>RP/0/RP0/CPU0:CE2</mark>#show run int tenGigE 0/0/0/0.2
interface TenGigE0/0/0/0.2
 ipv4 address 192.2.0.2 255.255.255.0
 encapsulation dot1q 2
!

<mark>RP/0/RP0/CPU0:CE1</mark>#ping 192.2.0.2 repeat 20
Type escape sequence to abort.
Sending 20, 100-byte ICMP Echos to 192.2.0.2, timeout is 2 seconds:
!!!!!!!!!!!!!!!!!!!!
<mark>Success rate is 100 percent (20/20), round-trip min/avg/max = 1/3/45 ms</mark>
RP/0/RP0/CPU0:CE1#
 </code>
</pre>
</div>

## Conclusion
This concludes Part 3 of our tutorial explaing point-to-point l2 serviec over SRv6 transport. Stay tuned for our upcoming tutorials.
