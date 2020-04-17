---
published: true
date: '2020-04-14 16:35 -0700'
title: BGP EVPN and L3VPN Interworking
author: Ahmad Bilal
excerpt: BGP-EVPN and L3VPN Interworking Support on NCS5500
---
{% include toc %}

## BGP EVPN and L3VPN Interworking Support on NCS5500

BGP EVPN and L3VPN interworking is a way to connect EVPN domain such as a DC or CO over an IPVPN Core/WAN network. This is a common use-case for end-to-end connectivity of Hosts/CEs in EVPN domain to other domains over an IPVPN network providing inter-subnet routing.
Below topology shows an EVPN fabric connecting to L3VPN domain with the help of DCI/Boarder-Leaf routers. The DCI routers perform BGP EVPN to L3VPN interworking to provide the reachability between PE-1s and Host prefixes. Boarder-Leaf/DCI routers are essential in these types of designs to keep prefixes local to each domain and send summarized advertisement out for better scalability.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-topology.png?raw=true)

In this post we will go over the configuration of EVPN and L3VPN interworking on NCS 5500 routers acting as DCI. When we complete the configuration, we will have Host subnet (10.0.0.0/24) learnt on PE-1, and PE-1’s loopback-100 learnt on Leafs. We will verify the reachability between Hosts and PE-1’s prefixes, that are advertised by their respective address-families. 

The configuration setup is based on single BGP AS 65001. There are two separate ISIS routing domains for EVPN and L3VPN with Segment Routing enabled for MPLS based forwarding. DCI performs BGP EVPN and L3VPN interworking, hence it is participating in both ISIS domains. There is no route redistribute between ISIS domains.

To configure EVPN and L3VPN interworking and achieve end-to-end connectivity, below is the list of tasks we will implement. Some of these tasks are already covered in previous posts, their details will not be covered here. Click on the links below to visit previous posts. The remaining items from the list that don't have links to previous write-ups are covered in this post.
   
- [Configure BGP-EVPN control-plane & Segment Routing based MPLS forwarding](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-1/)
- [Configure BGP EVPN based Layer-2 VPN Service](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/)
- [Configure BGP EVPN IRB for Inter-subnet Routing](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-irb-configuration/)
- Configure Segment Routing on DCI routers
- Configure BGP L3VPN domain
- Configure DCI routers to perform EVPN and L3VPN interworking 


### Task 1: Configuration of Segment Routing on DCI routers
Segment routing configuration for EVPN fabric is covered in earlier post but DCI routers were not part of that post. That is why we will only cover segment routing configuration for DCI and show it participating in two MPLS forwarding domains. One for providing forwarding to EVPN fabric and other to the L3VPN domain.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-transport.png?raw=true)



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
router isis 1
 is-type level-2-only
 net 49.0001.0000.0000.0008.00
 nsr
 log adjacency changes
 address-family ipv4 unicast
  metric-style wide
  segment-routing mpls
 !
 interface Bundle-Ether68
  point-to-point
  address-family ipv4 unicast
 !
 interface Bundle-Ether78
  point-to-point
  address-family ipv4 unicast
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
   prefix-sid absolute 16008
!
router isis 2
 is-type level-2-only
 net 49.0002.0000.0000.0008.00
 nsr
 log adjacency changes
 address-family ipv4 unicast
  metric-style wide
  segment-routing mpls
 !
 interface Bundle-Ether81
  point-to-point
  address-family ipv4 unicast
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
   prefix-sid absolute 16008
!      
      </code>
      </pre>
      </div>
    </th>
    <th style="border: none;">
      <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
router isis 1
 is-type level-2-only
 net 49.0001.0000.0000.0009.00
 nsr
 log adjacency changes
 address-family ipv4 unicast
  metric-style wide
  segment-routing mpls
 !
 interface Bundle-Ether69
  point-to-point
  address-family ipv4 unicast
 !
 interface Bundle-Ether79
  point-to-point
  address-family ipv4 unicast
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
   prefix-sid absolute 16009
!
router isis 2
 is-type level-2-only
 net 49.0002.0000.0000.0009.00
 nsr
 log adjacency changes
 address-family ipv4 unicast
  metric-style wide
  segment-routing mpls
 !
 interface Bundle-Ether91
  point-to-point
  address-family ipv4 unicast
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
   prefix-sid absolute 16009
!
      </code>
      </pre>
      </div>
    </th>
  </tr>
</table>

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
DCI-1#show isis segment-routing label table 

IS-IS 1 IS Label Table
Label         Prefix/Interface
----------    ----------------
16001         1.1.1.1/32
16002         2.2.2.2/32
16006         6.6.6.6/32
16007         7.7.7.7/32
16008         Loopback0
16009         9.9.9.9/32

IS-IS 2 IS Label Table
Label         Prefix/Interface
----------    ----------------
16008         Loopback0
16009         9.9.9.9/32
16010         10.10.10.10/32 
      </code>
      </pre>
      </div>
    </th>
    <th style="border: none;">
      <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
DCI-2#show isis segment-routing label table 

IS-IS 1 IS Label Table
Label         Prefix/Interface
----------    ----------------
16001         1.1.1.1/32
16002         2.2.2.2/32
16006         6.6.6.6/32
16007         7.7.7.7/32
16008         8.8.8.8/32
16009         Loopback0

IS-IS 2 IS Label Table
Label         Prefix/Interface
----------    ----------------
16008         8.8.8.8/32
16009         Loopback0
16010         10.10.10.10/32
      </code>
      </pre>
      </div>
    </th>
  </tr>
</table>
  
  
### Task 2: Configuration of L3VPN on DCI and PE-1

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-vpnv4-topology.png?raw=true)
As per the topology we have L3VPN configured between DCIs and PE-1. VRF 10 is configured on DCIs and PE-1 with route-target 110:110. This route-target is different from the one we will configure for BGP-EVPN routes.  

<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
Configure VPN VRF on PE-1, DCI-1 and DCI-2

vrf 10
 address-family ipv4 unicast
  import route-target
   110:110
  !
  export route-target
   110:110
  !
</code>
</pre>
</div>


Configure BGP L3VPN
<table style="border-collapse: collapse; border: none;">
  <tr style="border: none;">
    <th style="border: none;">PE-1</th>
    <th style="border: none;">DCI-1</th>
    <th style="border: none;">DCI-2</th>
  </tr>
  <tr style="border: none;">
    <th style="border: none;">
      <div class="highlighter-rouge">
      <pre class="highlight">
      <code>

router bgp 65001
 bgp router-id 10.10.10.10
 address-family vpnv4 unicast
 !
 neighbor 8.8.8.8
  remote-as 65001
  description "vonv4 session to DCI-1"
  update-source Loopback0
  address-family vpnv4 unicast
 !
 neighbor 9.9.9.9
  remote-as 65001
  description "vpnv4 session to DCI-2"
  update-source Loopback0
  address-family vpnv4 unicast
 !
 vrf 10
  rd auto
  address-family ipv4 unicast
   maximum-paths ibgp 10
   redistribute connected
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
 bgp router-id 8.8.8.8
 address-family vpnv4 unicast
 !
 neighbor 9.9.9.9
  remote-as 65001
  description "vpnv4 session to DCI-2"
  update-source Loopback0
  address-family vpnv4 unicast
   next-hop-self
 !
 neighbor 10.10.10.10
  remote-as 65001
  description "vpnv4 session to PE-1"
  update-source Loopback0
  address-family vpnv4 unicast
   next-hop-self
 !        
 vrf 10
  rd auto
  address-family ipv4 unicast
   maximum-paths ibgp 10
   redistribute connected
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
 address-family vpnv4 unicast
 !
 neighbor 8.8.8.8
  remote-as 65001
  description "vpnv4 session to DCI-1"
  update-source Loopback0
  address-family vpnv4 unicast
   next-hop-self
 !
 neighbor 10.10.10.10
  remote-as 65001
  description "vpnv4 session to PE-1"
  update-source Loopback0
  address-family vpnv4 unicast
   next-hop-self
 !        
 vrf 10
  rd auto
  address-family ipv4 unicast
   maximum-paths ibgp 10
   redistribute connected
 !

      </code>
      </pre>
      </div>
    </th>
  </tr>
</table>


Configure Loopback 100 on PE-1. This will be advertised as vpnv4 prefix.
<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
PE-1:

interface Loopback100
 vrf 10
 ipv4 address 111.1.1.1 255.255.255.255
!
      </code>
      </pre>
      </div>


Check routing table of DCIs for VRF 10 to verify that PE-1 prefix is learnt via vpnv4 address family.
<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
RP/0/RP0/CPU0:DCI-1#show route vrf 10   
Gateway of last resort is not set
B    111.1.1.1/32 [200/0] via 10.10.10.10 (nexthop in vrf default), 01:07:05
RP/0/RP0/CPU0:DCI-1#


RP/0/RP0/CPU0:DCI-1#sh cef vrf 10 111.1.1.1/32
111.1.1.1/32, version 1, internal 0x5000001 0x0 (ptr 0x97c1d714) [1], 0x0 (0x0), 0x208 (0x98422d28)
 Updated Apr 14 11:39:51.099
 Prefix Len 32, traffic index 0, precedence n/a, priority 3
   via 10.10.10.10/32, 3 dependencies, recursive [flags 0x6000]
    path-idx 0 NHID 0x0 [0x972aef08 0x0]
    recursion-via-/32
    next hop VRF - 'default', table - 0xe0000000
    next hop 10.10.10.10/32 via 16010/0/21
     next hop 192.8.10.2/32 BE81         labels imposed {ImplNull 24017}


RP/0/RP0/CPU0:DCI-1#show bgp vpnv4 unicast rd 10.10.10.10:0 111.1.1.1/32 detail
BGP routing table entry for 111.1.1.1/32, Route Distinguisher: 10.10.10.10:0
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker                  3           3
    Flags: 0x00040001+0x00000000; 
Last Modified: Apr 14 11:39:50.740 for 00:02:05
Paths: (1 available, best #1)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Flags: 0x4000000025060005, import: 0x1f
  Not advertised to any peer
  Local
    10.10.10.10 (metric 10) from 10.10.10.10 (10.10.10.10)
      Received Label 24017 
      Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, not-in-vrf
      Received Path ID 0, Local Path ID 1, version 3
      Extended community: RT:110:110 
RP/0/RP0/CPU0:DCI-1#
      </code>
      </pre>
      </div>

Based on above output we can see the route is learnt and programming in the forwarding table for vrf 10. Next lets configure BGP-EVPN fabric.

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