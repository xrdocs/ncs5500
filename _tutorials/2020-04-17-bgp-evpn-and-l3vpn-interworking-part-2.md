---
published: false
date: '2020-04-17 00:10 -0700'
title: BGP EVPN and L3VPN Interworking - Part-2
author: Ahmad Bilal
---
{% include toc %}

## BGP EVPN and L3VPN Interworking Support on NCS5500 - Part-2

We concluded the previous post by configuration and verification of BGP L3VPN. In this post we are going to cover the remaining tasks of configuration BGP EVPN on DCI routers and implementing EVPN to L3VPN interworking. 

### Task 3: Configuration of BGP-EVPN on Leafs, Spines and DCIs
The EVPN fabric configuration is done in previous post but that did not include DCIs. We will configure DCIs now and form BGP EVPN neighborship with Route-Reflectors/Spines.
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-evpn-topology.png?raw=true)

Configure VPN VRF on DCIs.
<div class="highlighter-rouge">
<pre class="highlight">
<code>
vrf 10
 address-family ipv4 unicast
  import route-target
   10:10
  !
  export route-target
   10:10
  !
pv4 address 111.1.1.1 255.255.255.255
!
</code>
</pre>
</div>

Configure BGP-EVPN neighborship with Route Reflectors.

<table style="border-collapse: collapse; border: none;">
  <tr style="border: none;">
    <th style="border: none;">DCI-1</th>
    <th style="border: none;">DCI-2</th>
  </tr>
  <tr style="border: none;">
    <th style="border: none;">
      <div class="highlighter-rouge">
      <pre class="highlight">
      <code>

router bgp 65001
 bgp router-id 8.8.8.8
 address-family l2vpn evpn
 !
 neighbor 6.6.6.6
  remote-as 65001
  description "BGP-EVPN session to Spine-1"      
  update-source Loopback0
  address-family l2vpn evpn
   next-hop-self
  !
 !
 neighbor 7.7.7.7
  remote-as 65001
  description "BGP-EVPN session to Spine-2"      
  update-source Loopback0
  address-family l2vpn evpn
   next-hop-self
  !
 !
 vrf 10 ----- VRF is already configured under BGP, when we configured L3VPN on DCI -----
  rd auto
  address-family ipv4 unicast
   maximum-paths ibgp 10
   redistribute connected
  !
 ! 
 </code>
      </pre>
      </div>
    </th>
    <th style="border: none;">
      <div class="highlighter-rouge">
      <pre class="highlight">
      <code>

router bgp 65001
 bgp router-id 9.9.9.9
 address-family l2vpn evpn
 !
 neighbor 6.6.6.6
  remote-as 65001
  description "BGP-EVPN session to Spine-1"      
  update-source Loopback0
  address-family l2vpn evpn
   next-hop-self
  !
 !
 neighbor 7.7.7.7
  remote-as 65001
  description "BGP-EVPN session to Spine-2"      
  update-source Loopback0
  address-family l2vpn evpn
   next-hop-self
  !       
 !
 vrf 10 ----- VRF is already configured under BGP, when we configured L3VPN on DCI -----
  rd auto 
  address-family ipv4 unicast
   maximum-paths ibgp 10
   redistribute connected
  !
 !
      </code>
      </pre>
      </div>
    </th>
  </tr>
</table>

As BGP-EVPN Layer-2 service and IRB on Leafs is already configured (refer to this post), with DCI connectivity we should be able to learn Host prefixes on DCI in VRF 10. Lets verify the routing table as well as the BGP-EVPN control-plane on DCIs.

<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
      
RP/0/RP0/CPU0:DCI-1#sh route vrf 10

Gateway of last resort is not set

B    10.0.0.20/32 [200/0] via 1.1.1.1 (nexthop in vrf default), 00:02:32
B    10.0.0.40/32 [200/0] via 2.2.2.2 (nexthop in vrf default), 00:00:59
B    111.1.1.1/32 [200/0] via 10.10.10.10 (nexthop in vrf default), 00:28:38
RP/0/RP0/CPU0:DCI-1#



RP/0/RP0/CPU0:DCI-1#show bgp l2vpn evpn 
BGP router identifier 8.8.8.8, local AS number 65001
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 0
BGP main routing table version 3
BGP NSR Initial initsync version 1 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1.1.1.1:10
*>i[2][0][48][6c9c.ed6a.9504][32][10.0.0.20]/136
                      1.1.1.1                       100      0 i
* i                   1.1.1.1                       100      0 i
Route Distinguisher: 2.2.2.2:10
*>i[2][0][48][6c9c.ed6a.9505][32][10.0.0.40]/136
                      2.2.2.2                       100      0 i
* i                   2.2.2.2                       100      0 i

Processed 2 prefixes, 4 paths
RP/0/RP0/CPU0:DCI-1#




RP/0/RP0/CPU0:DCI-1#sh bgp l2vpn evpn rd 2.2.2.2:10 [2][0][48][6c9c.ed6a.9505][32][10.0.0.40]/136 detail
BGP routing table entry for [2][0][48][6c9c.ed6a.9505][32][10.0.0.40]/136, Route Distinguisher: 2.2.2.2:10
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker                  3           3
    Flags: 0x00040001+0x00010000; 
Last Modified: Apr 14 12:07:29.740 for 00:03:35
Paths: (2 available, best #1)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Flags: 0x4000000025060005, import: 0x1f
  Not advertised to any peer
  Local
    2.2.2.2 (metric 20) from 6.6.6.6 (2.2.2.2)
      Received Label 24009, Second Label 24010
      Origin IGP, localpref 100, valid, internal, best, group-best, import-candidate, not-in-vrf
      Received Path ID 0, Local Path ID 1, version 3
      Extended community: Flags 0x1e: SoO:2.2.2.2:10 0x060e:0000.0000.000a RT:10:10 RT:1001:11 
      Originator: 2.2.2.2, Cluster list: 6.6.6.6
      EVPN ESI: 0000.0000.0000.0000.0000
  Path #2: Received by speaker 0
  Flags: 0x4000000020020005, import: 0x20
  Not advertised to any peer
  Local
    2.2.2.2 (metric 20) from 7.7.7.7 (2.2.2.2)
      Received Label 24009, Second Label 24010
      Origin IGP, localpref 100, valid, internal, not-in-vrf
      Received Path ID 0, Local Path ID 0, version 0
      Extended community: Flags 0x1e: SoO:2.2.2.2:10 0x060e:0000.0000.000a RT:10:10 RT:1001:11 
      Originator: 2.2.2.2, Cluster list: 7.7.7.7
      EVPN ESI: 0000.0000.0000.0000.0000
RP/0/RP0/CPU0:DCI-1#
      </code>
      </pre>
      </div>

Above output verifies the evpn host routes are learnt for VRF 10 and control-plane verifies that these routes have route-target (RT) as 10:10 as advertised by Leafs. PE-1 was configured with RT 110:110. The routing table on DCI for vrf 10 also shows the route learnt from PE-1 that was advertised by vpnv4 address-family with route-target 110:110. As DCI has reachability to both the EVPN as well as the L3VPN domains, lets now configure the EVPN and L3VPN interworking on DCI for end-to-end Leafs and PE-1 reachability. 


### Task 4: Configure BGP EVPN and L3VPN interworking

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-evpn-topology.png?raw=true)

So far we have configured EVPN fabric and L3VPN domain. PE-1’s prefix is advertised via vpnv4 address-family and Leafs’ prefixes are advertised via evpn address-family towards DCI. On DCI, VRF 10 is configured with two sets of import and export route-targets. One set is associated to L3VPN domain using vpnv4 to advertise layer-3 information; while the other set is for EVPN fabric using evpn address-family for advertisement of routes. The separation of route-targets enables DCI to have two separate domains configured independently and then stitching the two set of route-targets for interworking. Below configuration is making EVPN RTs as stitching RTs, while the L3VPN are normal RTs. 