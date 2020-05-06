---
published: true
date: '2020-04-06 21:24 -0700'
title: BGP-EVPN based Single-Active Multi-Homing
author: Ahmad Bilal Siddiqui
excerpt: >-
  Implementation and configuration of BGP-EVPN based Single-Active Multi-Homing
  on IOS-XR routers.
tags:
  - iosxr
  - cisco
  - EVPN
  - NCS 5500
Position: top
---
{% include toc %}

## Implementation of BGP-EVPN based Single-Active Multi-Homing

In single-active multi-homing mode, only a single Leaf among a group of Leafs attached to a Host is allowed to forward the traffic to and from on a given VLAN.

In this post we will cover the BGP-EVPN based Single-Active Multi-Homing of CE/Hosts. Similar to Active/Active Multi-homing, Single-Active is also achieved by EVPN Ethernet Segment feature. Single-active offers redundant connectivity with forwarding for a VLAN on a single link at a time with failover to the second link in case of active link’s failure. Single-Active load balancing’s strengths arise from directing traffic to a single uplink as opposed to all-active's approach of ECMP-hashing. This approach is very useful for network scenarios where policing, metering and billing are required.  

## Reference Topology:
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/reference-topology-evpn-single-active.png?raw=true)

For this post, we will leverage EVPN control-plane and ISIS Segment Routing based forwarding that we configured in a previous [post.](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-1/)

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/single-active-multi-homing.png?raw=true)

As shown in the above topology, Host-1 is multi-homed to Leaf-1 and Leaf-2. For EVPN single-active multi-homing, each link towards the Leaf will be in a unique ethernet bundle interface. VLAN 10 and 20 are allowed on both the ethernet-bundles. As both the links are in separate ethernet bundles, the host H-1 will flood traffic at first to both the Leafs but only the Ethernet-Segment's Designated Forwarder (DF) Leaf will forward the traffic. As a result, the host will have only one ethernet bundle interface in its forwarding table to forward the traffic and achieve per VLAN single-active multi-homing. 

### Task 1: Configure Ethernet bundles on Host-1 for multi-homing

As per the reference topology Host-1 is multi-homed to Leaf-1 and Leaf-2 via LACP bundle-ethernet 11 going to Leaf-1 and bundle-ethernet 12 going to Leaf-2. ASR9K is acting as the host/CE with IP address 10.0.0.10/24 configured on a BVI. Following is the configuration of LAG on Host-1.  

The LAG on Host-1 will come up after we configure single-active multi-homing using EVPN Ether-Segment on the Leaf-1 and Leaf-2.

**Note:** In this post we will configure VLAN 10 to show the single-active behavior. Configuration of VLAN 20 is out of scope for this post but follows the same procedure. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>

Host-1:

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
interface BVI10
 description "Host-1 IP"
 ipv4 address 10.0.0.10 255.255.255.0
!
l2vpn
 bridge group bg1
  bridge-domain bd-10
   interface Bundle-Ether<mark>11.10</mark>
   !
   interface Bundle-Ether<mark>12.10</mark>
   !
   routed interface BVI10
 !
!
</code>
</pre>
</div>

### Task 2: Configure EVPN based single-active multi-homing

Configure Leaf-1 and Leaf-2 to provision single-active multi-homing to host-1. The set of links from Host-1 to the Leafs will be configured as Ethernet Segment on the Leafs.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/ethernet-segment-single-active-multi-homing.png?raw=true)

Configure the bundles on the Leaf-1 and Leaf-2. Use below configuration for the Leafs.
**Note:** For single-active multi-homing, the LACP System MAC address should not be configured on ethernet bundle interface.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1:

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

**Note:** Single-active mode is the default for Physical interfaces and no extra configuration to enable single-active is required.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1:

evpn
 interface Bundle-Ether 11
  ethernet-segment
   identifier type 0 <mark>11.11.11.11.11.11.11.11.11</mark>
   <mark>load-balancing-mode single-active</mark>
  !


Leaf-2:

evpn
 interface Bundle-Ether 12
  ethernet-segment
   identifier type 0 <mark>11.11.11.11.11.11.11.11.11</mark>
   <mark>load-balancing-mode single-active</mark>
  !
</code>
</pre>
</div>

Use “show bundle bundle-ether <bundle-id>” CLI command to verify the state of the bundle interface on Leafs and Host-1.
  
<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1:

RP/0/RP0/CPU0:Leaf-1#show bundle bundle-ether 11

Bundle-Ether11
  Status:                                    <mark>Up</mark>
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
  LACP:                                      <mark>Operational</mark>
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
  Status:                                    <mark>Up</mark>
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
  LACP:                                      <mark>Operational</mark>
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

Above output shows that the bundle interfaces are up. Next, lets provision the EVPN layer-2 service. 


### Task 3: Configure BGP EVPN based layer-2 multipoint service
Configure the EVPN layer-2 service between Leaf-1, Leaf-2 and Leaf-5 and then check the status of ethernet segment. For detailed explanation of configuring BGP EVPN based layer-2 service, refer to this [post.](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/)  
  
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/single-active-layer2-stretch.png?raw=true)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1:

interface Bundle-Ether 11.10 l2transport
  encapsulation dot1q 10
  rewrite ingress tag pop 1 symmetric
  !
l2vpn
  bridge group bg-1
   bridge-domain bd-10
    interface Bundle-Ether 11.10
    evi 10
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
 !


Leaf-2:

interface Bundle-Ether 12.10 l2transport
 encapsulation dot1q 10
 rewrite ingress tag pop 1 symmetric
 !
l2vpn
 bridge group bg-1
  bridge-domain bd-10
   interface Bundle-Ether 12.10
   evi 10
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
  !



Leaf-5:

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

Host-5:

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
Leaf-1:

RP/0/RP0/CPU0:Leaf-1#show evpn ethernet-segment detail
Legend:

Ethernet Segment Id      Interface                          Nexthops            
------------------------ ---------------------------------- --------------------
0011.1111.1111.1111.1111 BE11                               <mark>1.1.1.1</mark>
                                                            <mark>2.2.2.2</mark>
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
     Operational    : <mark>MH, Single-active</mark>
     Configured     : <mark>Single-active (AApS)</mark>
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Peering Details   :
     1.1.1.1 [MOD:P:00]
     2.2.2.2 [MOD:P:00]

  Service Carving Results:
     Forwarders     : <mark>1</mark>
     Permanent      : 0
     Elected        : <mark>1</mark>
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
0011.1111.1111.1111.1111 BE12                               <mark>1.1.1.1</mark>
                                                            <mark>2.2.2.2</mark>
  ES to BGP Gates   : Ready
  ES to L2FIB Gates : Ready
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
     Operational    : <mark>MH, Single-active</mark>
     Configured     : <mark>Single-active (AApS)</mark>
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Peering Details   :
     1.1.1.1 [MOD:P:00]
     2.2.2.2 [MOD:P:00]

  Service Carving Results:
     Forwarders     : <mark>1</mark>
     Permanent      : 0
     Elected        : 0
     Not Elected    : <mark>1</mark>
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

The output of both the Leafs show that both are forwarders of 1 subnet (10.0.0.x/24 in our case), while Leaf-1 is elected as Designated Forwarder (DF) and Leaf-2 is the non-DF. This means that any Uniccast and BUM traffic that comes to Leaf-2 will not be forwarded and only Leaf-1 being the DF will forward it. 

Ping from Host-1 to Host-5 shows that the hosts are reachable.

<div class="highlighter-rouge">
<pre class="highlight">
<code>

Host-1:

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
RP/0/RP0/CPU0:Leaf-5#show bgp l2vpn evpn <mark>rd 1.1.1.1:10</mark>

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1.1.1.1:10
*>i[1][0011.1111.1111.1111.1111][0]/120
                      1.1.1.1                       100      0 i
* i                   1.1.1.1                       100      0 i
*>i<mark>[2][0][48][6c9c.ed6d.1d90][0]/104</mark>
                      1.1.1.1                       100      0 i
* i                   1.1.1.1                       100      0 i
*>i[3][0][32][1.1.1.1]/80
                      1.1.1.1                       100      0 i
* i                   1.1.1.1                       100      0 i

Processed 3 prefixes, 6 paths
RP/0/RP0/CPU0:Leaf-5#


Leaf-5 – Route advertisement from Leaf-2

RP/0/RP0/CPU0:Leaf-5#show bgp l2vpn evpn <mark>rd 2.2.2.2:10</mark>

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
10         MPLS   <mark>a03d.6f3d.5447 ::              5.5.5.5</mark>                    24010   
RP/0/RP0/CPU0:Leaf-1#

Leaf-2
RP/0/RP0/CPU0:Leaf-2#show evpn evi vpn-id 10 mac

VPN-ID     Encap  MAC address    IP address      Nexthop                    Label   
---------- ------ -------------- --------------- -------------------------- --------
10         MPLS   6c9c.ed6d.1d90 ::              1.1.1.1                    24004   
10         MPLS   <mark>a03d.6f3d.5447 ::              5.5.5.5</mark>                    24010   
RP/0/RP0/CPU0:Leaf-2#


Leaf-5
RP/0/RP0/CPU0:Leaf-5#show evpn evi vpn-id 10 mac

VPN-ID     Encap  MAC address    IP addres       Nexthop                    Label   
---------- ------ -------------- --------------- -------------------------- --------
10         MPLS   <mark>6c9c.ed6d.1d90 ::              1.1.1.1</mark>                    24004   
10         MPLS   a03d.6f3d.5447 ::              TenGigE0/0/0/45.10         24010   
RP/0/RP0/CPU0:Leaf-5#

</code>
</pre>
</div>

As we observe Leaf-5’s output, we see that the Leaf-5 has programmed Leaf-1 as the only next-hop for Host-1’s MAC address reachability, although Host-1 is multi-homed to both Leaf-1 and Leaf-2. This verifies that single-active dual-homing is operational and that at one time only one Leaf will forward the traffic to and from the Host for a given EVI.  

**Note:** As Leaf-2 sees Host-1's MAC reachable via Leaf-1, in case of another Host/ESI connected to Leaf-2 wants to reach to Host-1 it will have to go over Leaf-1 to reach to Host-1.

### Task 5: Configure the BGP-EVPN Distributed Anycast Gateway for inter-subnet routing
For Layer-3 inter-subnet routing use case; similar to Host-1’s layer-2 reachability, Host-1’s IP will also only be reachable via Leaf-1 as next-hop. After we configure BGP-EVPN distributed anycast gateway for inter-subnet routing, we will observe the routing table of Leaf-5.

Configure the BGP-EVPN Distributed Anycast Gateway on Leaf-1, Leaf-2 and Leaf-5. For detailed explanation of distributed anycast gateway, refer to this [post.]( https://xrdocs.io/ncs5500/tutorials/bgp-evpn-irb-configuration/)  

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/single-active-layer3-irb.png?raw=true)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
BGP-EVPN distributed anycast gateway configuration.

Configure VRFs on Leaf-1, Leaf-2 and Leaf-5.

      vrf 10
       address-family ipv4 unicast
        import route-target
         10:10
        !
        export route-target
         10:10
       !


      router bgp 65001
       address-family vpnv4 unicast
       !
       vrf 10
        rd auto
        address-family ipv4 unicast
         additional-paths receive
         maximum-paths ibgp 10
         redistribute connected
      !

Configure BVI as distributed anycast gateway

      interface BVI 10
       host-routing
       vrf 10
       ipv4 address 10.0.0.1 255.255.255.0
       mac-address 1001.1001.1001
      !

      l2vpn
       bridge group bg-1
        bridge-domain bd-10
        interface Bundle-Ether11.10 ---- configure on Leaf-1
        interface Bundle-Ether12.10 ---- configure on Leaf-2
        interface TenGigE0/0/0/45.10 ---- configure on Leaf-5
        !
        routed interface BVI 10
        evi 10
     !

Configure the static route on Host-1 and Host-5 to reach to the default gateway on Leafs.

      router static
       address-family ipv4 unicast
        0.0.0.0/0 10.0.0.1
       !

</code>
</pre>
</div>


As we have now configured the BGP-EVPN distributed anycast gateway on Leafs, lets observe the routing table of Leaf-5. The below output shows that Host-1's IP 10.0.0.10/32 is reachable via only Leaf-1. 
<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-5:
RP/0/RP0/CPU0:Leaf-5#show route vrf 10
Gateway of last resort is not set

C    10.0.0.0/24 is directly connected, 00:41:23, BVI10
L    10.0.0.1/32 is directly connected, 00:41:23, BVI10
B    <mark>10.0.0.10/32 [200/0] via 1.1.1.1</mark> (nexthop in vrf default), 00:46:13
RP/0/RP0/CPU0:Leaf-5#

Leaf-1:
RP/0/RP0/CPU0:Leaf-1#show arp vrf 10

-------------------------------------------------------------------------------
0/0/CPU0
-------------------------------------------------------------------------------
Address         Age        Hardware Addr   State      Type  Interface
10.0.0.1        -          1001.1001.1001  Interface  ARPA  BVI10
<mark>10.0.0.10       00:04:16   6c9c.ed6d.1d91  Dynamic    ARPA  BVI10</mark>
RP/0/RP0/CPU0:Leaf-1#

Leaf-2:
RP/0/RP0/CPU0:Leaf-2#show arp vrf 10

-------------------------------------------------------------------------------
0/0/CPU0
-------------------------------------------------------------------------------
Address         Age        Hardware Addr   State      Type  Interface
10.0.0.1        -          1001.1001.1001  Interface  ARPA  BVI10
RP/0/RP0/CPU0:Leaf-2#
</code>
</pre>
</div>

Finally we can observe in the ARP table output of Leaf-1 and Leaf-2 that the ARP entry for Host-1 is only programmed on Leaf-1. This is becuase of the single-active behavior of ethernet-segment and Leaf-1 being the designated-forwarder. This concludes the BGP-EVPN single-active implementation, for further technical details refer to our [e-vpn.io](http://e-vpn.io/) webpage that has a lot of material explaining the core concepts of EVPN, its operations and troubleshooting.
