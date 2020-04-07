---
published: true
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

As shown in the above topology, Host-1 is multi-homed to Leaf-1 and Leaf-2. For EVPN single-active multi-homing, each link towards the Leaf will be in a unique ethernet bundle interface. As both the links are in separate ethernet bundles, the host H-1 will flood traffic at first to both the Leafs but only the designated forwarder (DF) Leaf will forward the traffic. As a result, the host will have only one ethernet bundle interface in its forwarding table to forward the traffic and achieve per VLAN single-active multi-homing. 

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
l2vpn
 bridge group bg1
  bridge-domain bd-10
   interface Bundle-Ether<mark>11.10</mark>
   !
   interface Bundle-Ether<mark>12.10</mark>
   !
 !
!
</code>
</pre>
</div>

### Task 2: Configure EVPN based single-active multi-homing

Configure Leaf-1 and Leaf-2 to provision single-active multi-homing to host-1. The set of links from Host-1 to the Leafs will be configured as Ethernet Segment on the Leafs. NCS 5500 platform supports static LAG as well as LACP, however in this guide we are using LACP for link aggregation.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/ethernet-segment-single-active-multi-homing.png?raw=true)

Configure the bundles on the Leaf-1 and Leaf-2. Use below configuration for the Leafs.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
**Note:** For single-active multi-homing, the LACP System MAC address should not be configured on ethernet bundle interface.

Leaf-1

interface TenGigE0/0/0/47
 description "Link to Host-1" 
 bundle id 11 mode active
!
interface Bundle-Ether 11
 description "Bundle to Host-1"
!

Leaf-2

interface TenGigE0/0/0/47
 description "Link to Host-1" 
 bundle id 12 mode active
!
interface Bundle-Ether 12
 description "Bundle to Host-1"
!
</code>
</pre>
</div>

Configure Ethernet Segment id (ESI) for the bundle interface to enable multi-homing of the host. Use the identical ethernet-segment configuration on both the Leafs, though the ethernet-bundle interface is different for both Leafs. Configure load-balancing mode to single-active using “single-active” keyword for ethernet-segment.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1

evpn
 interface Bundle-Ether 11
  ethernet-segment
   identifier type 0 <mark>11.11.11.11.11.11.11.11.11</mark>
   <mark>load-balancing-mode single-active</mark>
   bgp route-target 1111.1111.1111
  !


Leaf-2

evpn
 interface Bundle-Ether 12
  ethernet-segment
   identifier type 0 <mark>11.11.11.11.11.11.11.11.11</mark>
   <mark>load-balancing-mode single-active</mark>
   bgp route-target 1111.1111.1111
  !
</code>
</pre>
</div>

Use “show bundle bundle-ether <bundle-id>” CLI command to verify the state of the bundle interface on Leafs and Host-1.
  
<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1

RP/0/RP0/CPU0:Leaf-1#sh bundle bundle-ether 11

Bundle-Ether11
  Status:                                    Up
  Local links <active/standby/configured>:   1 / 0 / 1
  Local bandwidth <effective/available>:     10000000 (10000000) kbps
  MAC address (source):                      00bc.601c.d0da (Chassis pool)
  Inter-chassis link:                        No
  Minimum active links / bandwidth:          1 / 1 kbps
  Maximum active links:                      64
  Wait while timer:                          2000 ms
  Load balancing:                            
    Link order signaling:                    Not configured
    Hash type:                               Default
    Locality threshold:                      None
  LACP:                                      Operational
    Flap suppression timer:                  Off
    Cisco extensions:                        Disabled
    Non-revertive:                           Disabled
  mLACP:                                     Not configured
  IPv4 BFD:                                  Not configured
  IPv6 BFD:                                  Not configured

  Port                  Device           State        Port ID         B/W, kbps
  --------------------  ---------------  -----------  --------------  ----------
  Te0/0/0/47            Local            Active       0x8000, 0x0003    10000000
      Link is Active
RP/0/RP0/CPU0:Leaf-1#


Leaf-2

RP/0/RP0/CPU0:Leaf-2#sh bundle bundle-ether 12

Bundle-Ether12
  Status:                                    Up
  Local links <active/standby/configured>:   1 / 0 / 1
  Local bandwidth <effective/available>:     10000000 (10000000) kbps
  MAC address (source):                      00bc.600e.40da (Chassis pool)
  Inter-chassis link:                        No
  Minimum active links / bandwidth:          1 / 1 kbps
  Maximum active links:                      64
  Wait while timer:                          2000 ms
  Load balancing:                            
    Link order signaling:                    Not configured
    Hash type:                               Default
    Locality threshold:                      None
  LACP:                                      Operational
    Flap suppression timer:                  Off
    Cisco extensions:                        Disabled
    Non-revertive:                           Disabled
  mLACP:                                     Not configured
  IPv4 BFD:                                  Not configured
  IPv6 BFD:                                  Not configured

  Port                  Device           State        Port ID         B/W, kbps
  --------------------  ---------------  -----------  --------------  ----------
  Te0/0/0/47            Local            Active       0x8000, 0x0003    10000000
      Link is Active
RP/0/RP0/CPU0:Leaf-2#

</code>
</pre>
</div>

Above output shows that the bundle interfaces are up. Next, lets provision the EVPN layer-2 service between Leaf-1, Leaf-2 and Leaf-5 and then check the status of ethernet segment. 

### Task 3: Configure BGP EVPN based layer-2 multipoint service
For detailed explanation of configuring BGP EVPN based layer-2 service, refer to this [post.](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/)  
  
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/single-active-layer2-stretch.png?raw=true)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1

interface Bundle-Ether 11.10 l2transport
     encapsulation dot1q 10
     rewrite ingress tag pop 1 symmetric
    !
l2vpn
     bridge group bg-1
      bridge-domain bd-10
       interface Bundle-Ether 11.10
       !
    !
evpn
     evi 10
      bgp
       route-target import 1001:11
       route-target export 1001:11
      !
      advertise-mac
      !
     source interface loopback 0
     !
l2vpn
     bridge group bg-1
      bridge-domain bd-10
       evi 10
       !
      !

Leaf-2

interface Bundle-Ether 12.10 l2transport
     encapsulation dot1q 10
     rewrite ingress tag pop 1 symmetric
    !
l2vpn
     bridge group bg-1
      bridge-domain bd-10
       interface Bundle-Ether 12.10
       !
    !
evpn
     evi 10
      bgp
       route-target import 1001:11
       route-target export 1001:11
      !
      advertise-mac
      !
     source interface loopback 0
     !
l2vpn
     bridge group bg-1
      bridge-domain bd-10
       evi 10
       !
      !


Leaf-5

interface TenGigE0/0/0/45.10 l2transport
 encapsulation dot1q 10
 rewrite ingress tag pop 1 symmetric
!
evpn
 evi 10
  bgp
   route-target import 1001:11
   route-target export 1001:11
  !
  advertise-mac
  !
 !
 source interface Loopback0
!
l2vpn
 bridge group bg-1
  bridge-domain bd-10
   interface TenGigE0/0/0/45.10
   !
   evi 10
   !
  !
</code>
</pre>
</div>

Host-5 is single-homed to Leaf-5, below is the Host-5 configuration. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>

Host-5

interface TenGigE0/0/1/3.10
 description "Link to Leaf-5"
 ipv4 address 10.0.0.50 255.255.255.0
 encapsulation dot1q 10

</code>
</pre>
</div>

### Task 4: Verify that EVPN based single-active multi-homing is operational

As we have configured the BGP EVPN layer-2 service as well as the ethernet segment, lets verify the ethernet segment status by “show evpn ethernet-segment detail”. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1

RP/0/RP0/CPU0:Leaf-1#show evpn ethernet-segment detail
Legend:

Ethernet Segment Id      Interface                          Nexthops            
------------------------ ---------------------------------- --------------------
0011.1111.1111.1111.1111 BE11                               1.1.1.1
                                                            2.2.2.2
  ES to BGP Gates   : Ready
  ES to L2FIB Gates : Ready
  Main port         :
     Interface name : Bundle-Ether11
     Interface MAC  : 00bc.601c.d0db
     IfHandle       : 0x08000144
     State          : Up
     Redundancy     : Not Defined
  ESI type          : 0
     Value          : 11.1111.1111.1111.1111
  ES Import RT      : 1111.1111.1111 (Local)
  Source MAC        : 0000.0000.0000 (N/A)
  Topology          :
     Operational    : MH, Single-active
     Configured     : Single-active (AApS)
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Peering Details   :
     1.1.1.1 [MOD:P:00]
     2.2.2.2 [MOD:P:00]

  Service Carving Results:
     Forwarders     : 1
     Permanent      : 0
     Elected        : 1
     Not Elected    : 0
  MAC Flushing mode : STP-TCN
  Peering timer     : 3 sec [not running]
  Recovery timer    : 30 sec [not running]
  Carving timer     : 0 sec [not running]
  Local SHG label   : 24005
  Remote SHG labels : 1
              24005 : nexthop 2.2.2.2


Leaf-2

RP/0/RP0/CPU0:Leaf-2#sh evpn ethernet-segment detail 
Legend:

Ethernet Segment Id      Interface                          Nexthops            
------------------------ ---------------------------------- --------------------
0011.1111.1111.1111.1111 BE12                               1.1.1.1
                                                            2.2.2.2
  ES to BGP Gates   : Ready
  ES to L2FIB Gates : O
  Main port         :
     Interface name : Bundle-Ether12
     Interface MAC  : 00bc.600e.40db
     IfHandle       : 0x0800011c
     State          : Up
     Redundancy     : Not Defined
  ESI type          : 0
     Value          : 11.1111.1111.1111.1111
  ES Import RT      : 1111.1111.1111 (Local)
  Source MAC        : 0000.0000.0000 (N/A)
  Topology          :
     Operational    : MH, Single-active
     Configured     : Single-active (AApS)
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Peering Details   :
     1.1.1.1 [MOD:P:00]
     2.2.2.2 [MOD:P:00]

  Service Carving Results:
     Forwarders     : 1
     Permanent      : 0
     Elected        : 0
     Not Elected    : 1
  MAC Flushing mode : STP-TCN
  Peering timer     : 3 sec [not running]
  Recovery timer    : 30 sec [not running]
  Carving timer     : 0 sec [not running]
  Local SHG label   : 24005
  Remote SHG labels : 1
              24005 : nexthop 1.1.1.1


</code>
</pre>
</div>

In the above output we can observe that Leaf-1 has bundle-ethernet 11 and Leaf-2 has bundle-ethernet 12 in ‘Up’ state. Both have two next-hops, one being the Leaf itself and the second next-hop is the peer-leaf/PE. Operational state of the ethernet-segment is multi-homed with single-active load-balancing. 

The output of both the Leafs show that both are forwarders of 1 subnet (10.0.0.x/24 in our case), while Leaf-1 is elected as Designated Forwarder (DF) and Leaf-2 is the non-DF. This means that any BUM traffic that comes to Leaf-2 will not be forwarded and only Leaf-1 being the DF will forward it. 

Ping from Host-1 to Host-5 shows that the hosts are reachable.

<div class="highlighter-rouge">
<pre class="highlight">
<code>

Host-1

RP/0/RSP0/CPU0:Host-1#ping 10.0.0.50
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.50, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
RP/0/RSP0/CPU0:Host-1#

</code>
</pre>
</div>

Let’s take a look at the BGP EVPN control-plane to verify that only Leaf-1 being the designated-forwarder for EVI 10 is advertising itself the next-hop and Leaf-2 is not announcing any MAC addresses related to EVI 10. This is due to the fact that for single-active load-balancing only one Leaf-1 should advertise the reachability. 

In the below output from Leaf-5 we can see the MAC address of Host-1 is learnt from Leaf-1 (rd 1.1.1.1:10) in a route-type 2 advertisement. As we look at Leaf-2’s route distinguishers (rd 2.2.2.2:10) we see that no MAC address is advertised for EVI 10. This verifies that only Leaf-1 will be programmed in Leaf-5 as the next-hop to reach to Host-1.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-5 – Route advertisement from Leaf-1
RP/0/RP0/CPU0:Leaf-5#show bgp l2vpn evpn rd 1.1.1.1:10

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1.1.1.1:10
*>i[1][0011.1111.1111.1111.1111][0]/120
                      1.1.1.1                       100      0 i
* i                   1.1.1.1                       100      0 i
*>i[2][0][48][6c9c.ed6d.1d90][0]/104
                      1.1.1.1                       100      0 i
* i                   1.1.1.1                       100      0 i
*>i[3][0][32][1.1.1.1]/80
                      1.1.1.1                       100      0 i
* i                   1.1.1.1                       100      0 i

Processed 3 prefixes, 6 paths
RP/0/RP0/CPU0:Leaf-5#


Leaf-5 – Route advertisement from Leaf-2

RP/0/RP0/CPU0:Leaf-5#show bgp l2vpn evpn rd 2.2.2.2:10
BGP router identifier 5.5.5.5, local AS number 65001
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 0
BGP main routing table version 55
BGP NSR Initial initsync version 1 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 2.2.2.2:10
*>i[1][0011.1111.1111.1111.1111][0]/120
                      2.2.2.2                       100      0 i
* i                   2.2.2.2                       100      0 i
*>i[3][0][32][2.2.2.2]/80
                      2.2.2.2                       100      0 i
* i                   2.2.2.2                       100      0 i

Processed 2 prefixes, 4 paths
RP/0/RP0/CPU0:Leaf-5#

</code>
</pre>
</div>

Lastly, run “show evpn evi vpn-id 10 mac” command to verify the MAC address learnt for EVI 10. We see that Leaf-1 and Leaf-2 have learnt Host-5’s MAC address with Leaf-5 as the next-hop.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1
RP/0/RP0/CPU0:Leaf-1#show evpn evi vpn-id 10 mac

VPN-ID     Encap  MAC address    IP address      Nexthop                    Label   
---------- ------ -------------- --------------- -------------------------- --------
10         MPLS   6c9c.ed6d.1d90 ::              Bundle-Ether11.10          24004   
10         MPLS   a03d.6f3d.5447 ::              5.5.5.5                    24010   
RP/0/RP0/CPU0:Leaf-1#

Leaf-2
RP/0/RP0/CPU0:Leaf-2#show evpn evi vpn-id 10 mac

VPN-ID     Encap  MAC address    IP address      Nexthop                    Label   
---------- ------ -------------- --------------- -------------------------- --------
10         MPLS   6c9c.ed6d.1d90 ::              1.1.1.1                    24004   
10         MPLS   a03d.6f3d.5447 ::              5.5.5.5                    24010   
RP/0/RP0/CPU0:Leaf-2#


Leaf-5
RP/0/RP0/CPU0:Leaf-5#show evpn evi vpn-id 10 mac

VPN-ID     Encap  MAC address    IP addres       Nexthop                    Label   
---------- ------ -------------- --------------- -------------------------- --------
10         MPLS   6c9c.ed6d.1d90 ::              1.1.1.1                    24004   
10         MPLS   a03d.6f3d.5447 ::              TenGigE0/0/0/45.10         24010   
RP/0/RP0/CPU0:Leaf-5#

</code>
</pre>
</div>

As we observe Leaf-5’s output, we see that the Leaf-5 has programmed Leaf-1 as the only next-hop for Host-1’s MAC address reachability, although Host-1 is multi-homed to both Leaf-1 and Leaf-2. This verifies that single-active dual-homing is operational and that at one time only one Leaf will forward the traffic to and from the Host for a given EVI. For further technical details, refer to our [e-vpn.io](http://e-vpn.io/) webpage that has a lot of material explaining the core concepts of EVPN, its operations and troubleshooting.
