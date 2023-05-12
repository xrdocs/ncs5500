---
published: true
date: '2023-05-12 09:03 +0530'
title: Implementing EVPN ELAN over SRv6 Transport on NCS 500/5500
author: Paban Sarma
position: hidden
excerpt: >-
  Ethernet VPN based layer 2 multipoint (ELAN) service over SRv6 uSID Transport
  on NCS 500 and NCS 5500 series platforms. Includes detailed explanation of
  configuration and verificationsteps
---
## Overview

In our previous tutorials, covered SRv6 Transport with uSID on the NCS 500 and 5500 platforms, and L3/L2 P2P services on top of it. This tutorial will cover, implementaion of Ethernet VPN based multipoint layer 2 service (ELAN) over the SRv6 uSID transport. As of today, only Single homed EVPN ELAN is supported on these platform.

## Topology
image placeholder

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
BGP configuration is simplar to what we did in our previous tutorial. However, since we have multiple PE nodes here, we need to establish full mesh BGP with EVPN AFI here. For simplicity, we are using P2 as a route-reflector. In real time deployment, it is recomended to use dedicated route reflectors in the netwrok. The following config snippet shows the BGP confifuration on all the PEs and the RR node.

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

_**PE1, PE4  and PE5**_
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

### Configuring Layer2 Attachment Circuits & Bridge-Domain

We need to configure l2transport sub-interface (on the PE-CE link) with appropriate VLAN encapsulations. This tutorial is showing VLAN based service with VLAN ID 200. We are not showing any VLAN translation operation (rewrite commands) as the are out of scope of this tutorial. 

_**PE1, PE4 and PE5**_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
interface TenGigE0/0/0/0.2 l2transport
 encapsulation dot1q 200
</code>
</pre>
</div>

The layer2 sub interface created now needs to be stitched with the EVI by using the l2vpn bridge-domain service construct as shown below. Note that the `segment-routing srv6` keyword is must here , along with that we can also specify a locator if we wish to use different locator for the bridge-domain.

_**PE1, PE4 and PE5**_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
l2vpn
 bridge group POD0
  bridge-domain POD0
   interface TenGigE0/0/0/0.2
   !
   evi 200 segment-routing srv6
   !
  !
 !
!
</code>
</pre>
</div>

## Verification Steps

### Verifying EVPN ELAN control Plane
The very first step is to verify weather the configured bridge-domain is in Up state on all the PE nodes. For brevity we have included the verification outputs only from PE5.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:LABSP-3393-PE5#show  l2vpn bridge-domain brief 
Fri May 12 04:35:22.647 UTC
Legend: pp = Partially Programmed.
Bridge Group:Bridge-Domain Name  ID    State          Num ACs/up   Num PWs/up    Num PBBs/up Num VNIs/up
-------------------------------- ----- -------------- ------------ ------------- ----------- -----------
POD0:POD0                        1     up             1/1          0/0           0/0         0/0   
</code>
</pre>
</div>

The next, step is to see the programmed SRv6 SIDs for the service we configured. For each EVI , there are two SRv6 SID programmed, uDT2U and uDT2M for Unicast and BUM traffic respectively.


<div class="highlighter-rouge">
<pre class="highlight">
<code>

RP/0/RP0/CPU0:LABSP-3393-PE5#show  segment-routing  srv6  sid 
Fri May 12 04:38:57.345 UTC

*** Locator: 'POD0' *** 

SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  --------------------------------  ------------------  -----  --
fcbb:bb00:5::               uN (PSP/USD)      'default':5                       sidmgr              InUse  Y 
fcbb:bb00:5:e000::          uA (PSP/USD)      [BE35, Link-Local]:0:P            isis-1              InUse  Y 
fcbb:bb00:5:e001::          uA (PSP/USD)      [BE35, Link-Local]:0              isis-1              InUse  Y 
fcbb:bb00:5:e002::          uA (PSP/USD)      [BE25, Link-Local]:0:P            isis-1              InUse  Y 
fcbb:bb00:5:e003::          uA (PSP/USD)      [BE25, Link-Local]:0              isis-1              InUse  Y 
fcbb:bb00:5:e004::          uA (PSP/USD)      [BE45, Link-Local]:0:P            isis-1              InUse  Y 
fcbb:bb00:5:e005::          uA (PSP/USD)      [BE45, Link-Local]:0              isis-1              InUse  Y 
<mark> fcbb:bb00:5:e006::          uDT2U             200:0                             l2vpn_srv6          InUse  Y</mark>
<mark>fcbb:bb00:5:e007::          uDT2M             200:0                             l2vpn_srv6          InUse  Y</mark>
</code>
</pre>
</div>

The same SIDs can also be verified using the EVI detail CLI.
<div class="highlighter-rouge">
<pre class="highlight">
<code>

RP/0/RP0/CPU0:LABSP-3393-PE5#show  evpn  evi  vpn-id  200 detail 
Fri May 12 06:04:53.970 UTC

VPN-ID     Encap      Bridge Domain                Type               
---------- ---------- ---------------------------- -------------------
<mark> 200        SRv6       POD0                         EVPN     </mark>          
   Stitching: Regular
   <mark>Unicast SID:  fcbb:bb00:5:e006::</mark>                  
   <mark>Multicast SID:  fcbb:bb00:5:e007::</mark>                     
   E-Tree: Root
   Forward-class: 0
   Advertise MACs: Yes
   Advertise BVI MACs: No
   Aliasing: Enabled
   UUF: Enabled
   Re-origination: Enabled
   Multicast:
     Source connected   : No
     IGMP-Snooping Proxy: No
     MLD-Snooping Proxy : No
   BGP Implicit Import: Enabled
   VRF Name: 
   SRv6 Locator Name: POD0
   Preferred Nexthop Mode: Off
   BVI Coupled Mode: No
   BVI Subnet Withheld: ipv4 No, ipv6 No
   RD Config: none
   RD Auto  : (auto) 5.5.5.5:200
   RT Auto  : 100:200
   Route Targets in Use           Type                 
   ------------------------------ ---------------------
   100:200                        Import               
   100:200                        Export 
</code>
</pre>
</div>

The above output also shows the two SIDs for two types of traffic for the EVI. we can check details of the SIDs using `show segment-routing srv6 sid <> detail` commands as below:

<div class="highlighter-rouge">
<pre class="highlight">
<code>

RP/0/RP0/CPU0:LABSP-3393-PE5#show  segment-routing  srv6  sid  fcbb:bb00:5:e006:: detail 
Fri May 12 06:11:15.177 UTC

*** Locator: 'POD0' *** 

SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  --------------------------------  ------------------  -----  --
fcbb:bb00:5:e006::          uDT2U             200:0                             l2vpn_srv6          InUse  Y 
  <mark> SID Function: 0xe006</mark>
  <mark>SID context: { evi=200, opaque-id=0 }</mark>
  <mark>Locator: 'POD0'</mark>
  Allocation type: Dynamic
  Created: May 11 06:29:44.913 (23:41:30 ago)
  </code>
</pre>
</div>

### Verifying EVPN ELAN Data Plane and MAC Learnings
The CE nodes are are configured in the same L2 subnets which we want to stich using the EVPN service. Below are the IP configurations on each CE

<table>
<tr>
    <th>CE1</th>
    <th>CE2</th>
    <th>CE3</th>
</tr>
<tr>
  <td>
	<div class="highlighter-rouge">
		<pre class="highlight">
			<code>
	interface TenGigE0/0/0/0.2
 	ipv4 address 200.0.0.1 255.255.255.0
 	encapsulation dot1q 200
	!
			</code>
		</pre>
	</div>
  </td>
  <td>
	<div class="highlighter-rouge">
		<pre class="highlight">
			<code>
	interface TenGigE0/0/0/0.2
 	ipv4 address 200.0.0.2 255.255.255.0
 	encapsulation dot1q 200
	!
			</code>
		</pre>
	</div>
  </td>
  <td>
	<div class="highlighter-rouge">
		<pre class="highlight">
			<code>
	interface TenGigE0/0/0/0.2
 	ipv4 address 200.0.0.3 255.255.255.0
 	encapsulation dot1q 200
	!
			</code>
		</pre>
	</div>
  </td>
 </tr>
</table>
  
We can now try verifying end-to-end ping from CE1 to the other CE nodes to confirm the working of the EVPN service. 

<div class="highlighter-rouge">
		<pre class="highlight">
			<code>
RP/0/RP0/CPU0:LABSP-3393-CE1#ping 200.0.0.1
Fri May 12 05:22:43.472 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 200.0.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
RP/0/RP0/CPU0:LABSP-3393-CE1#ping 200.0.0.2
Fri May 12 05:22:44.911 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 200.0.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
RP/0/RP0/CPU0:LABSP-3393-CE1#ping 200.0.0.3
Fri May 12 05:22:46.593 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 200.0.0.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
			</code>
		</pre>
	</div>

As the packets went through the EVPN PE's , we will see the respective MAC addresses learnt locally and via EVPN. The below CLI snippets shows the learnings on PE5.

<div class="highlighter-rouge">
		<pre class="highlight">
			<code>
RP/0/RP0/CPU0:LABSP-3393-PE5#show  evpn  evi  vpn-id 200 mac 
Fri May 12 06:32:40.216 UTC

VPN-ID     Encap      MAC address    IP address                               Nexthop                                 Label    SID                                    
---------- ---------- -------------- ---------------------------------------- --------------------------------------- -------- ---------------------------------------
200        SRv6       00bc.6016.5800 ::                                       fcbb:bb00:1::1                                 IMP-NULL fcbb:bb00:1:e000::                     
200        SRv6       00bc.6024.c400 ::                                       fcbb:bb00:1::4                                IMP-NULL fcbb:bb00:4:e000::                     
200        SRv6       00bc.6027.6400 ::                                       TenGigE0/0/0/0.2                        0        fcbb:bb00:5:e006::    
		</code>
		</pre>
	</div>
We can see one locally learnt MAC on the UNI side and two remote MAC learnt from the peer PEs. The same can also be seen in the EVPN RT2 received.

<div class="highlighter-rouge">
		<pre class="highlight">
			<code>
RP/0/RP0/CPU0:LABSP-3393-PE5#<mark> show  bgp  l2vpn evpn  rd 1.1.1.1:200</mark>
Fri May 12 06:36:10.445 UTC
BGP router identifier 5.5.5.5, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0
BGP main routing table version 218
BGP NSR Initial initsync version 1 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1.1.1.1:200
<mark>*>i[2][0][48][00bc.6016.5800][0]/104
                      fcbb:bb00:1::1                      100      0 i</mark>
*>i[3][0][32][1.1.1.1]/80
                      fcbb:bb00:1::1                       100      0 i

Processed 2 prefixes, 2 paths

RP/0/RP0/CPU0:LABSP-3393-PE5#<mark>show  bgp  l2vpn evpn  rd 4.4.4.4:200</mark>
Fri May 12 06:36:28.768 UTC
BGP router identifier 5.5.5.5, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0
BGP main routing table version 218
BGP NSR Initial initsync version 1 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 4.4.4.4:200
<mark>*>i[2][0][48][00bc.6024.c400][0]/104
                      fcbb:bb00:1::4                       100      0 i</mark>
*>i[3][0][32][4.4.4.4]/80
                      fcbb:bb00:1::4                       100      0 i

Processed 2 prefixes, 2 paths
</code>
</pre>
</div>

## Summary

This concludes Part 6 of our SRv6 tutorial series explaing multipoint L2 serviec over SRv6 transport with EVPN control Plane (EVPN ELAN). Stay tuned for our upcoming tutorials.
