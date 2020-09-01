---
published: false
date: '2020-09-01 10:16 +0530'
title: BGP EVPN based Port Active MultiHoming
author: Paban Sarma
excerpt: >-
  Tutorial Explaining step by step approach for BGP EVPN service with port
  active multihoming
tags:
  - iosxr
  - EVPN
  - NCS 5500
---
{% include_toc %}

## Implementation of BGP-EVPN based Port-Active Multi-Homing

In port-active multi-homing, a host/CE is multihomed to one or more Leaf/PEs and only one of the Leaf is  active and forwards the traffic to and from the connected hosts. The rest of the Leaf remain in standby mode. Thus these mode offers an active-standby PE/Leaf redundancy for multihomed host/CE.  
In this post we will cover the BGP-EVPN based Port-Active Multi-Homing of CE/Hosts. Similar to All active or Single active mode, Ethernet Segment Identifier (ESI) is used to identify the links towards the same multihomed Host. Port-active offers active/standby redundant connectivity with forwarding for all traffic on a single link at a time with switchover to the second link in case of active link’s failure.  Port-Active load balancing mode keeps only one link towards the host as active and rest of the link stays in LACP standby mode, thus creating a complete active standby multihoming for the connected host/CEs.  This is useful when we need protocol simplification from the host network.  

## Reference Topology

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/port_active_topo.png?raw=true)

For this post, we will leverage EVPN control-plane and ISIS Segment Routing based forwarding that we configured in a [previous post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-1/). However, the choice of transport is not mandatorily ISIS+SR and we can have OSPF as IGP and LDP instead of SR as well.  


![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/port_active_mh.png?raw=true)

As shown in the above topology, Host-1 is multi-homed to Leaf-1 and Leaf-2. For EVPN port multi-homing, the link towards the Leaf will be a single ethernet bundle interface. This bundle may operate with different VLANs for different services. EVPN port-active mode at the leaf1 and leaf2 will elect only one leaf as the active node and the bundle on that leaf will be in active state. The bundle on the other leaf will move to standby state and signal LACP out of service towards the host. As a result all traffic from the host H-1 will be able to forward the traffic only towards the active lacp link to achieve  port active redundancy for multihoming.  The election of active Leaf is similar operation like all active DF election, however in this case the election happens based on the ethernet segment identifier.  

### Task 1: Configure Ethernet bundle on Host-1 for multi-homing
As per the reference topology Host-1 is multi-homed to Leaf-1 and Leaf-2 via LACP bundle-ethernet 1 going to both Leaf-1 and Leaf-2.  The host/CE with IP address 10.0.0.10/24 configured on a vlan sub interface on the bundle. . Following is the configuration of LAG on Host-1.
The LAG on Host-1 will come up after we configure lacp and port-active multi-homing using EVPN Ether-Segment on the Leaf-1 and Leaf-2.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Host-1:

interface Bundle-Ether 1
description "Bundle to Leaf-1"
!
interface TenGigE0/0/2/0
description "Link to Leaf-1 ten0/0/0/47"
bundle id 1 mode active
!
interface TenGigE0/0/2/1
description "Link to Leaf-2 ten0/0/0/47"
bundle id 1 mode active
!
interface Bundle-Ether1.10
encapsulation dot1q 10
ipv4 address 10.0.0.10 255.255.255.0
!
</div>
</pre>
</code>

### Task 2: Configure EVPN based port-active multi-homing

Configure Leaf-1 and Leaf-2 to provision port-active multi-homing to host-1. The set of links from Host-1 to the Leafs will be configured as the same Ethernet Segment on the Leafs. 

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/port_active_mh_cfg.png?raw=true)

Configure the LACP bundles on the Leaf-1 and Leaf-2. Use below configuration for the Leafs. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1:

interface TenGigE0/0/0/47
description "Link to Host-1"
bundle id 1 mode active
!
interface Bundle-Ether1
 description "Bundle to Host-1 for port-active"
 lacp system mac 1212.1212.1212

Leaf-2

interface TenGigE0/0/0/47
description "Link to Host-1"
bundle id 1 mode active
!
interface Bundle-Ether1
 description "Bundle to Host-1 for port-active"
 lacp system mac 1212.1212.1212
!
</div>
</pre>
</code>

Configure ESI  for the bundle interface to enable multi-homing of the host. Use the identical ethernet-segment configuration on both the Leafs. Configure load-balancing mode to port-active using “port-active” keyword for ethernet-segment. 

**Note:** `The configured ESI will be used for the selection of active port. Out of the 10 octet ESI, a modulo operation is performed on octet 3-6 to elect the active leaf/PE`

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-1 and leaf 2

evpn
 interface Bundle-Ether1
  ethernet-segment
   identifier type 0 12.12.12.12.12.12.12.12.12
   <mark> load-balancing-mode port-active</mark>

</div>
</pre>
</code>

Use “_show bundle bundle-ether_” CLI command to verify the state of the bundle interfaces on Leafs and Host-1.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Leaf-1#show  bundle bundle-ether 1

Bundle-Ether1
  <mark>Status:</mark>                                    <mark>Up</mark>
  Local links <active/standby/configured>:   1 / 0 / 1
  Local bandwidth <effective/available>:     10000000 (10000000) kbps
  MAC address (source):                      00bc.601c.d0d9 (Chassis pool)
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
  Te0/0/0/47            Local            Active       0x8000, 0x0001    10000000
      Link is Active

RP/0/RP0/CPU0:Leaf-2#show bundle bundle-ether 1

Bundle-Ether1
  <mark>Status:                                    LACP OOS (out of service)</mark>
  Local links <active/standby/configured>:   0 / 1 / 1
  Local bandwidth <effective/available>:     0 (0) kbps
  MAC address (source):                      00bc.600e.40dc (Chassis pool)
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
  Te0/0/0/47            Local            Standby      0x8000, 0x0001    10000000
      <mark> Link is in standby due to bundle out of service state</mark>
</div>
</pre>
</code> 

Also, verify the port-active operation making one leaf active and one leaf standby by verifying the status of the ethernet segment on each PE


<div class="highlighter-rouge">
<pre class="highlight">
<code>
LEAF1:

RP/0/RP0/CPU0:Leaf-1#<mark> sh evpn  ethernet-segment interface bundle-Ether 1 detail</mark>
Legend:
  B   - No Forwarders EVPN-enabled,
  C   - Backbone Source MAC missing (PBB-EVPN),
  RT  - ES-Import Route Target missing,
  E   - ESI missing,
  H   - Interface handle missing,
  I   - Name (Interface or Virtual Access) missing,
  M   - Interface in Down state,
  O   - BGP End of Download missing,
  P   - Interface already Access Protected,
  Pf  - Interface forced single-homed,
  R   - BGP RID not received,
  S   - Interface in redundancy standby state,
  X   - ESI-extracted MAC Conflict
  SHG - No local split-horizon-group label allocated

Ethernet Segment Id      Interface                          Nexthops            
------------------------ ---------------------------------- --------------------
0012.1212.1212.1212.1212 BE1                                1.1.1.1
                                                            2.2.2.2
  ES to BGP Gates   : Ready
  ES to L2FIB Gates : Ready
  Main port         :
     Interface name : Bundle-Ether1
     Interface MAC  : 00bc.601c.d0d9
     IfHandle       : 0x08004034
     <mark>State          : Up</mark>
     Redundancy     : Not Defined
  ESI type          : 0
     Value          : 12.1212.1212.1212.1212
  ES Import RT      : 1212.1212.1212 (from ESI)
  Source MAC        : 0000.0000.0000 (N/A)
  Topology          :
     Operational    : MH
     Configured     : <mark>Port-Active</mark>
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Peering Details   :
     1.1.1.1 [MOD:P:00]
     2.2.2.2 [MOD:P:00]

  Service Carving Results:
     Forwarders     : 0
     Elected        : 0
     Not Elected    : 0
  EVPN-VPWS Service Carving Results:
     Primary        : 0
     Backup         : 0
     Non-DF         : 0
  MAC Flushing mode : STP-TCN
  Peering timer     : 3 sec [not running]
  Recovery timer    : 30 sec [not running]
  Carving timer     : 0 sec [not running]
  Local SHG label   : None
  Remote SHG labels : 0
  Access signal mode: Bundle OOS (Default)


LEAF2: 

RP/0/RP0/CPU0:Leaf-2# <mark> sh evpn  ethernet-segment interface bundle-Ether 1 detail</mark>

Legend:
  B   - No Forwarders EVPN-enabled,
  C   - Backbone Source MAC missing (PBB-EVPN),
  RT  - ES-Import Route Target missing,
  E   - ESI missing,
  H   - Interface handle missing,
  I   - Name (Interface or Virtual Access) missing,
  M   - Interface in Down state,
  O   - BGP End of Download missing,
  P   - Interface already Access Protected,
  Pf  - Interface forced single-homed,
  R   - BGP RID not received,
  S   - Interface in redundancy standby state,
  X   - ESI-extracted MAC Conflict
  SHG - No local split-horizon-group label allocated

Ethernet Segment Id      Interface                          Nexthops            
------------------------ ---------------------------------- --------------------
0012.1212.1212.1212.1212 BE1                                1.1.1.1
                                                            2.2.2.2
  ES to BGP Gates   : Ready
  ES to L2FIB Gates : Ready
  Main port         :
     Interface name : Bundle-Ether1
     Interface MAC  : 00bc.600e.40dc
     IfHandle       : 0x08004014
     <mark>State          : Standby</mark>
     Redundancy     : Not Defined
  ESI type          : 0
     Value          : 12.1212.1212.1212.1212
  ES Import RT      : 1212.1212.1212 (from ESI)
  Source MAC        : 0000.0000.0000 (N/A)
  Topology          :
     Operational    : MH
     Configured     : <mark>Port-Active</mark>
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Peering Details   :
     1.1.1.1 [MOD:P:00]
     2.2.2.2 [MOD:P:00]

  Service Carving Results:
     Forwarders     : 0
     Elected        : 0
     Not Elected    : 0
  EVPN-VPWS Service Carving Results:
     Primary        : 0
     Backup         : 0
     Non-DF         : 0
  MAC Flushing mode : STP-TCN
  Peering timer     : 3 sec [not running]
  Recovery timer    : 30 sec [not running]
  Carving timer     : 0 sec [not running]
  Local SHG label   : None
  Remote SHG labels : 0
  Access signal mode: Bundle OOS (Default)
</div>
</pre>
</code>

**Note** `In the example shown the ethernet segment Identifier is 00.12.12.12.12.12.12.12.12.12.12 and the portion impacting DF election is 12.12.12.12 as highlighted. For Dual homing an odd-even modulo operation will gives a result of 0. Therefore Leaf1 is our active PE as it has a lower BGP router ID of 1.1.1.1 compared to 2.2.2.2 of Leaf2.`

Above output shows that the bundle interfaces are up and port active redundancy mode has created an active standby Leaf redundancy for the dual homed Host-1 . Next, lets’ provision the EVPN layer-2 service over this redundancy.

### Task 3: Configure BGP EVPN based layer-2 multipoint service

Here we will configure a EVPN layer-2 service between Leaf-1, Leaf-2 and Leaf-5 to provide a L2VPN between H1 and H5. Post configuration we will check  the status of ethernet segment. For detailed explanation of configuring BGP EVPN based layer-2 service, refer to this [post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/).  

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/port_active_evpn.png?raw=true)

Here , the L2 service is configured on VLAN 10 (sub-interface on the bundle) and only one VPN (EVI) is shown. We may have multiple services running over different sub-interface (VLAN). 

```
Leaf-1:
interface Bundle-Ether 1.10 l2transport
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
!

Leaf-2:

interface Bundle-Ether 1.10 l2transport
encapsulation dot1q 10
rewrite ingress tag pop 1 symmetric
!

l2vpn
bridge group bg-1
bridge-domain bd-10
interface Bundle-Ether 1.10
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
```

Host-5 is single-homed to Leaf-5, below is the Host-5 configuration for reference.

```
Host-5:

interface TenGigE0/0/1/3.10
description "Link to Leaf-5"
ipv4 address 10.0.0.50 255.255.255.0
encapsulation dot1q 10
```

Once , the EVPN service is up, H1 will be able to reach H5 and vice-versa. 






