---
published: true
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
{% include toc %}

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

```
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
```

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

</code>
</pre>
</div>

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
</code>
</pre>
</div>

Use “_show bundle bundle-ether_” CLI command to verify the state of the bundle interfaces on Leafs and Host-1.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Leaf-1#show  bundle bundle-ether 1

Bundle-Ether1
  <mark>Status:</mark>                                    <mark>Up</mark>
  Local links &lt active/standby/configured &gt :   1 / 0 / 1
  Local bandwidth &lt effective/available&gt :     10000000 (10000000) kbps
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
  Local links &lt active/standby/configured &gt :   0 / 1 / 1
  Local bandwidth &lt effective/available &gt :     0 (0) kbps
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
</code>
</pre>
</div>

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
  <mark>Access signal mode: Bundle OOS (Default)</mark>


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
  <mark>Access signal mode: Bundle OOS (Default)</mark>
</code>
</pre>
</div>

**Note** `In the example shown the ethernet segment Identifier is 00.12.12.12.12.12.12.12.12.12.12 and the portion impacting DF election is 12.12.12.12 as highlighted. For Dual homing an odd-even modulo operation will gives a result of 0. Therefore Leaf1 is our active PE as it has a lower BGP router ID of 1.1.1.1 compared to 2.2.2.2 of Leaf2.`

Above output shows that the bundle interfaces are up and port active redundancy mode has created an active standby Leaf redundancy for the dual homed Host-1. By default the ethernet segment signals bundle OOS on the non-DF PE. The ES may also be configured with 'access-signal bundle-down'. This configuration is used to keep ES down instead of OOS when EVPN cost-out/core-isolation and similar triggers are applied. In the Down signalling mode, the CE side is able to switch ES from one to the other when LACP is not supported. The below snippet shows the configuration and CLI output.

<div>
<pre>
<code>
evpn
 interface Bundle-Ether2
  ethernet-segment
   identifier type 0 18.44.18.44.18.44.18.44.00
   load-balancing-mode port-active
  !
  <mark>access-signal bundle-down</mark>

RP/0/RP0/CPU0:LEAF-1#show evpn  ethernet-segment  interface bundle-Ether 2 detail 
Thu Nov 12 00:43:18.314 GMT+4
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
0018.4418.4418.4418.4400 BE2                                1.1.1.1
                                                            2.2.2.2
  ES to BGP Gates   : Ready
  ES to L2FIB Gates : Ready
  Main port         :
     Interface name : Bundle-Ether2
     Interface MAC  : 0032.1780.98de
     IfHandle       : 0x080040c4
     <mark>State          : Up</mark>
     Redundancy     : Not Defined
  ESI type          : 0
     Value          : 18.4418.4418.4418.4400
  ES Import RT      : 1844.1844.1844 (from ESI)
  Source MAC        : 0000.0000.0000 (N/A)
  Topology          :
     Operational    : MH
     Configured     : Port-Active
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Convergence       :
     Mobility-Flush : Count 0, Skip 0, Last n/a
  Peering Details   : 2 Nexthops
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
  <mark>Access signal mode: Bundle Down</mark>
  
  RP/0/RP0/CPU0:LEAF-2#show evpn  ethernet-segment  interface bundle-Ether 2 detail 
Thu Nov 12 04:49:28.018 UTC
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
0018.4418.4418.4418.4400 BE2                                1.1.1.1
                                                            2.2.2.2
  ES to BGP Gates   : Ready
  ES to L2FIB Gates : Ready
  Main port         :
     Interface name : Bundle-Ether2
     Interface MAC  : 00bc.6013.44de
     IfHandle       : 0x0800403c
     <mark>State          : Standby</mark>
     Redundancy     : Not Defined
  ESI type          : 0
     Value          : 18.4418.4418.4418.4400
  ES Import RT      : 1844.1844.1844 (from ESI)
  Source MAC        : 0000.0000.0000 (N/A)
  Topology          :
     Operational    : MH
     Configured     : Port-Active
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Convergence       :
     Mobility-Flush : Count 0, Skip 0, Last n/a
  Peering Details   : 2 Nexthops
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
  Access signal mode: Bundle Down
</code>
</pre>
</div>

Next, lets’ provision the EVPN layer-2 service over this redundancy.

### Task 3: Configure BGP EVPN based layer-2 multipoint service

Here we will configure a EVPN layer-2 service between Leaf-1, Leaf-2 and Leaf-5 to provide a L2VPN between H1 and H5. Post configuration we will check  the status of ethernet segment. For detailed explanation of configuring BGP EVPN based layer-2 service, refer to this [post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/).  

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/port_active_evpn.png?raw=true)

Here , the L2 service is configured on VLAN 10 (sub-interface on the bundle) and only one VPN (EVI) is shown. We may have multiple services running over different sub-interface (VLAN). 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
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
</code>
</pre>
</div>

Host-5 is single-homed to Leaf-5, below is the Host-5 configuration for reference.

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

Once , the EVPN service is up, H1 will be able to reach H5 and vice-versa. 


### Task 4: Verify that EVPN based Port-active multi-homing is operational

As we have configured the BGP EVPN layer-2 service as well as the ethernet segment, we have already verified the port active operation. Now using the same command again we can see in the service carving details and confirm that the EVPN service is only active on the active PE.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
LEAF1:

RP/0/RP0/CPU0:Leaf-1#show  evpn  ethernet-segment interface bundle-Ether 1 detail 
Thu Aug 13 11:58:07.149 UTC
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
     State          : Up
     Redundancy     : Not Defined
  ESI type          : 0
     Value          : 12.1212.1212.1212.1212
  ES Import RT      : 1212.1212.1212 (from ESI)
  Source MAC        : 0000.0000.0000 (N/A)
  Topology          :
     Operational    : MH
     Configured     : Port-Active
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Peering Details   :
     1.1.1.1 [MOD:P:00]
     2.2.2.2 [MOD:P:00]

  Service Carving Results:
     <mark>Forwarders     : 1</mark>
     Elected        : 1
     <mark>Not Elected    : 0</mark>
  EVPN-VPWS Service Carving Results:
     Primary        : 0
     Backup         : 0
     Non-DF         : 0
  MAC Flushing mode : STP-TCN
  Peering timer     : 3 sec [not running]
  Recovery timer    : 30 sec [not running]
  Carving timer     : 0 sec [not running]
  Local SHG label   : 24001
  Remote SHG labels : 1
              24001 : nexthop 2.2.2.2
  Access signal mode: Bundle OOS (Default)

LEAF2:

RP/0/RP0/CPU0:Leaf-2#show  evpn  ethernet-segment interface bundle-Ether 1 detail
Thu Aug 13 11:58:50.921 UTC
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
     State          : Standby
     Redundancy     : Not Defined
  ESI type          : 0
     Value          : 12.1212.1212.1212.1212
  ES Import RT      : 1212.1212.1212 (from ESI)
  Source MAC        : 0000.0000.0000 (N/A)
  Topology          :
     Operational    : MH
     Configured     : Port-Active
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Peering Details   :
     1.1.1.1 [MOD:P:00]
     2.2.2.2 [MOD:P:00]

  Service Carving Results:
     <mark>Forwarders     : 1</mark>
     Elected        : 0
     <mark>Not Elected    : 1</mark>
  EVPN-VPWS Service Carving Results:
     Primary        : 0
     Backup         : 0
     Non-DF         : 0
  MAC Flushing mode : STP-TCN
  Peering timer     : 3 sec [not running]
  Recovery timer    : 30 sec [not running]
  Carving timer     : 0 sec [not running]
  Local SHG label   : 24001
  Remote SHG labels : 1
              24001 : nexthop 1.1.1.1
  Access signal mode: Bundle OOS (Default)

</code>
</pre>
</div>

The above output on both PE shows that elected field is up only for the active PE,  although the output of both the Leafs show that both are forwarders of 1 service. Unlike All-active or Single-active, the same PE will be the elected PE for any other vlan configured on these ethernet segment. This is the nature of port active redundancy mode. To  

Ping from Host-1 to Host-5 shows that the hosts can reach each other.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Host-1:
RP/0/RSP0/CPU0:Host-1#ping 10.0.0.50
Thu Aug 13 11:29:24.024 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.50, timeout is 2 seconds:
!!!!!
</code>
</pre>
</div>

Let’s now take a look at the BGP EVPN control plane by checking the types of routes received on different leaf’s. We are filtering the route for the specific PE and specific service using rd which is PE:EVI . for example , routes from leaf1 for EVI 10 will come with a RD of _**1.1.1.1:10**_

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Leaf-1#show bgp l2vpn evpn rd <mark>5.5.5.5:10</mark>
------
Status codes: s suppressed, d damped, h history, * valid, &gt best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 5.5.5.5:10
*>i[2][0][48][a03d.6f3d.5447][0]/104
                      5.5.5.5                       100      0 i
*>i[3][0][32][5.5.5.5]/80
                      5.5.5.5                       100      0 i

Processed 2 prefixes, 2 paths

RP/0/RP0/CPU0:Leaf-1#show bgp l2vpn evpn rd <mark>2.2.2.2:10</mark>
-------
Status codes: s suppressed, d damped, h history, * valid, &gt best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 2.2.2.2:10
*>i[1][0012.1212.1212.1212.1212][0]/120
                      2.2.2.2                       100      0 i
*>i[3][0][32][2.2.2.2]/80
                      2.2.2.2                       100      0 i
</code>
</pre>
</div>                     

From Above output from Leaf-1 clearly shows it has reached the RT2 (MAC) from Leaf-5. From Leaf2 it has only received the ESI route.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Leaf-2#show bgp l2vpn evpn rd <mark>1.1.1.1:10</mark>
-------------

Status codes: s suppressed, d damped, h history, * valid, &gt best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1.1.1.1:10
*>i[1][0012.1212.1212.1212.1212][0]/120
                      1.1.1.1                       100      0 i
*>i[2][0][48][6c9c.ed6d.1d89][0]/104
                      1.1.1.1                       100      0 i
*>i[3][0][32][1.1.1.1]/80
                      1.1.1.1                       100      0 i

Processed 3 prefixes, 3 paths

RP/0/RP0/CPU0:Leaf-2#show bgp l2vpn evpn rd <mark>5.5.5.5:10</mark>
-------------
Status codes: s suppressed, d damped, h history, * valid, &gt best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 5.5.5.5:10
*>i[2][0][48][a03d.6f3d.5447][0]/104
                      5.5.5.5                       100      0 i
*>i[3][0][32][5.5.5.5]/80
                      5.5.5.5                       100      0 i

Processed 2 prefixes, 2 paths
</code>
</pre>
</div>

Above output shows Leaf-2 has learnt ESI and MAC of host 1 from Leaf1 and from Leaf 5 it has learnt the MAC of host-5.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Leaf-5#show bgp l2vpn evpn rd 1.1.1.1:10
-------
Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1.1.1.1:10
*>i[1][0012.1212.1212.1212.1212][0]/120
                      1.1.1.1                       100      0 i
*>i[2][0][48][6c9c.ed6d.1d89][0]/104
                      1.1.1.1                       100      0 i
*>i[3][0][32][1.1.1.1]/80
                      1.1.1.1                       100      0 i

Processed 3 prefixes, 3 paths

RP/0/RP0/CPU0:Leaf-5#show bgp l2vpn evpn rd 2.2.2.2:10
-----------
Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 2.2.2.2:10
*>i[1][0012.1212.1212.1212.1212][0]/120
                      2.2.2.2                       100      0 i
*>i[3][0][32][2.2.2.2]/80
                      2.2.2.2                       100      0 i

Processed 2 prefixes, 2 paths100      0 i

</code>
</pre>
</div>

Leaf-5 also learns the MAC of host1 only via Leaf1 as it was the only active PE and there is no aliasing in port active multihoming.

Lastly, run “show evpn evi vpn-id 10 mac” command to verify the MAC address learnt for EVI 10. We see that Leaf-1 and Leaf-2 have learnt Host-5’s MAC address with Leaf-5 as the next-hop. However , Leaf5 has learnt Host-1’s MAC with only Leaf-1 as nexthop.

<div class="highlighter-rouge">
<pre class="highlight">
<code>

RP/0/RP0/CPU0:Leaf-1#show evpn evi vpn-id 10 mac
Thu Aug 13 12:12:45.065 UTC

VPN-IDEncap	MAC address	IP address 	Nexthop		Label
---------- ---------- -------------- ----------------------------
10   MPLS   6c9c.ed6d.1d89 ::     		Bundle-Ether1.10	24000
10   MPLS   a03d.6f3d.5447 ::         	5.5.5.5       	       24004


RP/0/RP0/CPU0:Leaf-2#show evpn evi vpn-id 10 mac
Thu Aug 13 12:12:45.065 UTC

VPN-IDEncap	MAC address	IP address 	Nexthop		Label
---------- ---------- -------------- ----------------------------
10   MPLS    6c9c.ed6d.1d89 ::     		1.1.1.1		 24000
10   MPLS    a03d.6f3d.5447 ::         	5.5.5.5       	 24004



RP/0/RP0/CPU0:Leaf-1#show evpn evi vpn-id 10 mac
Thu Aug 13 12:12:45.065 UTC

VPN-IDEncap	MAC address	IP address 	Nexthop		Label
---------- ---------- -------------- ----------------------------
10   MPLS    6c9c.ed6d.1d89 ::     		1.1.1.1		 24000
10   MPLS    a03d.6f3d.5447 ::         TenGigE0/0/0/45.10	  24004

</code>
</pre>
</div>

The above output verifies the BGP-EVPN control plane for EVPN multipoint service over Port-active multihoming. 
**Note:** `As Leaf-2 sees Host-1's MAC reachable via Leaf-1, in case of another Host/ESI connected to Leaf-2 wants to reach to Host-1 it will have to go over Leaf-1 to reach to Host-1.`

### Task 5: Configure and Verify BGP-EVPN Distributed Anycast Gateway for IRB service

In this section we will demonstrate the Layer-3 inter-subnet routing use case over EVPN port active multihoming. Similar to Host-1’s layer-2 reachability, Host-1’s IP will also only be reachable via Leaf-1 as next-hop. After we configure BGP-EVPN distributed anycast gateway for inter-subnet routing, we will observe the routing table of Leaf-5.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/port_active_irb.png?raw=true)

Configure the BGP-EVPN Distributed Anycast Gateway on Leaf-1, Leaf-2 and Leaf-5. We will configure the IRB service over a different VLAN and show the coexistence of both service over the port active ESI. For detailed explanation of EVPN distributed anycast gateway, refer to this [post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-irb-configuration/).

<div class="highlighter-rouge">
<pre class="highlight">
<code>

<mark> Configure VRFs on Leaf-1, Leaf-2 and Leaf-5.</mark>

  vrf 11
   address-family ipv4 unicast
    import route-target
     11:11
    !
    export route-target
     11:11
   !
  router bgp 65001
   address-family vpnv4 unicast
   !
   vrf 11
    rd auto
    address-family ipv4 unicast
     additional-paths receive
     maximum-paths ibgp 10
     redistribute connected
  !


<mark> Configure BVI as distributed anycast gateway</mark>

On Leaf 1 and Leaf 2:

  interface BVI11
  host-routing
  vrf 11
  ipv4 address 111.0.0.1 255.255.255.0
  mac-address 1001.1001.1001
  !

interface Bundle-Ether1.11 l2transport
 encapsulation dot1q 11
 rewrite ingress tag pop 1 symmetric
! 
evpn
 evi 11
  advertise-mac
  !
 !
!
l2vpn
 bridge group bg1
  bridge-domain irb1
   interface Bundle-Ether1.11
   !
   routed interface BVI11
   !
   evi 11


<mark>Configure BVI as distributed anycast gateway</mark>

On Leaf 5:

interface BVI11
 host-routing
 vrf 11
 ipv4 address 111.0.1.1 255.255.255.0
 mac-address 5001.5001.5001

interface TenGigE0/0/0/45.11 l2transport
 encapsulation dot1q 11
 rewrite ingress tag pop 1 symmetric
!
evpn
 evi 11
  advertise-mac
  !
l2vpn
 bridge group bg1
  bridge-domain irb1
   interface TenGigE0/0/0/45.11
   !
   routed interface BVI11
   !
   evi 11
   
</code>
</pre>
</div>

We will also configure a two different subnet on the Host’s and respective static routing towards the gateways.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
HOST1: 

interface Bundle-Ether1.11
 ipv4 address 111.0.0.10 255.255.255.0
 encapsulation dot1q 11
!
router static
 address-family ipv4 unicast
  111.0.0.0/16 111.0.0.1
 !
!

HOST5: 

interface TenGigE0/0/1/3.11
 ipv4 address 111.0.1.50 255.255.255.0
 encapsulation dot1q 11
!
router static
 address-family ipv4 unicast
  111.0.0.0/16 111.0.1.1
</code>
</pre>
</div>

BGP-EVPN IRB control plane can be verified by observing the route tables on the Leaf node. As we can see the route for remote host’s are learnt on Leaf1 and Leaf-5 via BGP. As Leaf-2 is in standby mode it lean’s route to  Host-1 from Leaf-1 via BGP instead of learning directly.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Leaf-1#sh route vrf 11
---------
Gateway of last resort is not set

C    111.0.0.0/24 is directly connected, 00:32:09, BVI11
L    111.0.0.1/32 is directly connected, 00:32:09, BVI11
<mark>B    111.0.1.50/32</mark> [200/0] via 5.5.5.5 (nexthop in vrf default), 00:22:24

RP/0/RP0/CPU0:Leaf-2#sh route vrf 11
--------

Gateway of last resort is not set

<mark>B    111.0.0.10/32</mark> [200/0] via 1.1.1.1 (nexthop in vrf default), 00:25:16
<mark>B    111.0.1.50/32</mark> [200/0] via 5.5.5.5 (nexthop in vrf default), 00:23:21

RP/0/RP0/CPU0:Leaf-5#show  route vrf 11
--------------

Gateway of last resort is not set

<mark>B    111.0.0.10/32</mark> [200/0] via 1.1.1.1 (nexthop in vrf default), 00:26:38
C    111.0.1.0/24 is directly connected, 00:29:27, BVI11
L    111.0.1.1/32 is directly connected, 00:29:27, BVI11
</code>
</pre>
</div>

As of now we have configured 2 different services over the EVPN port-active multihoming and we see Leaf-1 as DF for both of this service. This is due to the fact that load balancing happens per port/ESI and the bundle on the non DF nodes are in LACP OOS status. If we see the Ethernet segment status on Leaf-1, we will see it as elected forwarder for all the configured services.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Leaf-1#show  evpn  ethernet-segment  in bundle-Ether 1 carving  detail 
Thu Aug 13 13:16:23.816 UTC
---------

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
     State          : Up
     Redundancy     : Not Defined
  ESI type          : 0
     Value          : 12.1212.1212.1212.1212
  ES Import RT      : 1212.1212.1212 (from ESI)
  Source MAC        : 0000.0000.0000 (N/A)
  Topology          :
     Operational    : MH
     Configured     : Port-Active
  Service Carving   : Auto-selection
     Multicast      : Disabled
  Peering Details   :
     1.1.1.1 [MOD:P:00]
     2.2.2.2 [MOD:P:00]

  Service Carving Results:
     <mark>Forwarders     : 2</mark>
     <mark>Elected        : 2</mark>
            <mark>EVI E   :       10,      11</mark>
     Not Elected    : 0
  EVPN-VPWS Service Carving Results:
     Primary        : 0
     Backup         : 0
     Non-DF         : 0
  MAC Flushing mode : STP-TCN
  Peering timer     : 3 sec [not running]
  Recovery timer    : 30 sec [not running]
  Carving timer     : 0 sec [not running]
  Local SHG label   : 24001
  Remote SHG labels : 1
              24001 : nexthop 2.2.2.2
  Access signal mode: Bundle OOS (Default)
</code>
</pre>
</div>

This concludes the BGP-EVPN based port-active implementation. We have shown example of both Layer2 bridging and IRB services over port-active redundancy. However, this redundancy mode can be used for any other services like layer3 or legacy layer 2. For further technical details refer to our [e-vpn.io](e-vpn.io) webpage that has a lot of material explaining the core concepts of EVPN, its operations and troubleshooting.
