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
Below topology shows an EVPN fabric connecting to L3VPN domain with the help of DCI/Boarder-Leaf routers. The DCI routers perform BGP EVPN to L3VPN interworking to provide the reachability between PE-1's and Host prefixes. Boarder-Leaf/DCI routers are essential in these types of designs to keep prefixes local to each domain and send summarized advertisement out.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-topology.png?raw=true)

In this post we will go over the configuration of EVPN and L3VPN interworking on NCS 5500 routers acting as DCI. When we complete the configuration, we will have Host subnet (10.0.0.0/24) learnt on PE-1, and PE-1’s vpnv4 prefix loopback-100 learnt on Leafs. We will verify the reachability between Hosts and PE-1’s prefixes, that are advertised by their respective address-families. 

The configuration setup is based on single BGP AS 65001. There are two separate ISIS routing domains for EVPN and L3VPN with Segment Routing enabled for MPLS based forwarding. DCI performs BGP EVPN and L3VPN interworking, hence it is participating in both ISIS domains. There is no route redistribute between ISIS domains.

Though EVPN and L3VPN interworking is going to be configured on DCI routers only, yet in this post we will go over the configuration of overall setup including EVPN fabric and L3VPN domain.  To achieve end-to-end connectivity, below is the list of tasks we will implement. Some of these tasks are already covered in previous posts, their details will not be covered here. Click on the links below to visit previous posts. The remaining items from the list that don't have links to previous write-ups are covered in this post.
   
- [Configure BGP-EVPN control-plane & Segment Routing based MPLS forwarding](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-1/)
- [Configure BGP EVPN based Layer-2 VPN Service](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/)
- [Configure BGP EVPN IRB for Inter-subnet Routing](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-irb-configuration/)
- Configure Segment Routing on DCI routers
- Configure BGP L3VPN domain
- Configure DCI routers to perform EVPN and L3VPN interworking 


### Task 1: Configuration of Segment Routing on DCI routers
Segment routing configuration for EVPN fabric is covered in earlier [post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-1/) but DCI routers were not part of that post. That is why we will only cover segment routing configuration for DCI and show it participating in two MPLS forwarding domains. One for providing forwarding to EVPN fabric and other to the L3VPN domain.

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
router isis <mark>1</mark>
 is-type level-2-only
 net <mark>49.0001.0000.0000.0008.00</mark>
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
   prefix-sid <mark>absolute 16008</mark>
!
router isis <mark>2</mark>
 is-type level-2-only
 net <mark>49.0002.0000.0000.0008.00</mark>
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
   prefix-sid <mark>absolute 16008</mark>
!      
      </code>
      </pre>
      </div>
    </th>
    <th style="border: none;">
      <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
router isis <mark>1</mark>
 is-type level-2-only
 net <mark>49.0001.0000.0000.0009.00</mark>
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
   prefix-sid <mark>absolute 16009</mark>
!
router isis <mark>2</mark>
 is-type level-2-only
 net <mark>49.0002.0000.0000.0009.00</mark>
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
   prefix-sid <mark>absolute 16009</mark>
!
      </code>
      </pre>
      </div>
    </th>
  </tr>
</table>

Below output shows the segment routing label table for both the ISIS processes on DCI routers. Routers in EVPN and L3VPN domains are reachable from DCI routers.
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

<mark>IS-IS 1 IS Label Table</mark>
Label         Prefix/Interface
----------    ----------------
16001         1.1.1.1/32
16002         2.2.2.2/32
16006         6.6.6.6/32
16007         7.7.7.7/32
16008         Loopback0
16009         9.9.9.9/32

<mark>IS-IS 2 IS Label Table</mark>
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

<mark>IS-IS 1 IS Label Table</mark>
Label         Prefix/Interface
----------    ----------------
16001         1.1.1.1/32
16002         2.2.2.2/32
16006         6.6.6.6/32
16007         7.7.7.7/32
16008         8.8.8.8/32
16009         Loopback0

<mark>IS-IS 2 IS Label Table</mark>
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
  
  
### Task 2: Configuration of BGP L3VPN on DCI and PE-1

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-vpnv4-topology.png?raw=true)
As per the topology we have L3VPN configured between DCIs and PE-1. VRF 10 is configured on DCIs and PE-1 with route-target 110:110. iBGP neighborship is formed using vpnv4 address-family between DCI routers and PE-1. Though we are not using a Route-Reflector in L3VPN domain for this write-up; a Route-Reflector is supported and can be used for this design.   
<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
Configure L3VPN VRF on PE-1, DCI-1 and DCI-2.

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

Configure BGP L3VPN neighborship via vpnv4 address-family. Also, configure the VRF under BGP to advertised the routes of the VRF to other PE routers. Initiate the VPNv4 address family to advertise VRF label. Route-Distinguisher (RD) auto under VRF generates RD value automatically. However, configuring RD manually is also supported.
We will use “redistribute connected” under VRF to advertise connected routes via BGP. In addition, we are configuring BGP multipathing for load balancing where multiple next-hops are available for a prefix.
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
  description "vpnv4 session to DCI-1"
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
   additional-paths receive
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
   additional-paths receive
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
   additional-paths receive
   maximum-paths ibgp 10
   redistribute connected
 !

      </code>
      </pre>
      </div>
    </th>
  </tr>
</table>


Configure Loopback 100 for VRF 10 on PE-1. This will be advertised as vpnv4 prefix to DCI routers, the DCI routers will re-originate this prefix and advertise to Leafs in EVPN fabric for end-to-end reachability.
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
At this point we are done with BGP L3VPN (VPNv4) configuration on DCI routers and PE-1. We are advertising interface Loopback 100's prefix from PE-1 towards DCI routers with vpnv4 address-family and route-targets 110:110 for import and export of routes for VRF 10. Check routing table of DCI routers for VRF 10 to verify that PE-1 prefix is learnt.
<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
RP/0/RP0/CPU0:DCI-1#show route vrf 10   
Gateway of last resort is not set
B    111.1.1.1/32 [200/0] via 10.10.10.10 (nexthop in vrf default), 01:07:05
RP/0/RP0/CPU0:DCI-1#


RP/0/RP0/CPU0:DCI-1#show cef vrf 10 111.1.1.1/32
111.1.1.1/32, version 1, internal 0x5000001 0x0 (ptr 0x97c1d714) [1], 0x0 (0x0), 0x208 (0x98422d28)
 Updated Apr 14 11:39:51.099
 Prefix Len 32, traffic index 0, precedence n/a, priority 3
   via 10.10.10.10/32, 3 dependencies, recursive [flags 0x6000]
    path-idx 0 NHID 0x0 [0x972aef08 0x0]
    recursion-via-/32
    next hop VRF - 'default', table - 0xe0000000
    next hop <mark>10.10.10.10/32</mark> via 16010/0/21
     next hop 192.8.10.2/32 BE81         <mark>labels imposed {ImplNull 24017}</mark>
      </code>
      </pre>
      </div>

The above output shows that PE-1's advertised prefix 111.1.1.1/32 is learnt on DCI in VRF 10. We can also verify the prefix advertisement using the L3VPN BGP control-plane. In the below output from DCI-1 we can see the details of prefix 111.1.1.1/32; that it is received from PE-1 (10.10.10.10), its label value and route-target (RT) information.  

<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
RP/0/RP0/CPU0:DCI-1#show bgp vpnv4 unicast rd 10.10.10.10:0 111.1.1.1/32 detail
BGP routing table entry for <mark>111.1.1.1/32</mark>, Route Distinguisher: 10.10.10.10:0
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
    <mark>10.10.10.10</mark> (metric 10) from 10.10.10.10 (10.10.10.10)
      Received Label <mark>24017</mark> 
      Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, not-in-vrf
      Received Path ID 0, Local Path ID 1, version 3
      Extended community: <mark>RT:110:110</mark> 
RP/0/RP0/CPU0:DCI-1#
      </code>
      </pre>
      </div>

Based on above output we can see the prefix from PE-1 learnt and programmed in the forwarding table for VRF 10 on DCI routers. This concludes the configuration and verification of BGP L3VPN (VPNv4) domain in the above setup. Next we will cover the remaining tasks of configuring BGP EVPN on DCI routers and implementing EVPN to L3VPN interworking.
  
   
### Task 3: Configuration of BGP-EVPN on Leafs, Spines and DCIs
The EVPN fabric configuration was done in [another post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/) but that did not include DCI router's configuration. We will configure DCI routers now and form BGP EVPN neighborship with Spines serving as Route-Reflectors.

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
 vrf 10 --- <mark>VRF was already configured in L3VPN config task</mark>
  rd auto
  address-family ipv4 unicast
   additional-paths receive
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
 vrf 10 --- <mark>VRF was already configured in L3VPN config task</mark>
  rd auto 
  address-family ipv4 unicast
   additional-paths receive
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

As BGP-EVPN Layer-2 VPN service and EVPN-IRB on Leafs is already configured in earlier posts (refer to [EVPN Layer-2 Service](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/) and [EVPN-IRB](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-irb-configuration/)); in next task lets configure route-target for VRF 10 to import and export EVPN routes. 


### Task 4: Configure BGP EVPN and L3VPN interworking on DCI routers
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-stitching-rt.png?raw=true)

**Configure route-target stitching for EVPN routes:**  
We already have configured VRF 10 on DCI when we configured L3VPN on DCI routers in Task 2. Now, since we are extending EVPN to DCI routers, we will configure EVPN RT (10:10) under VRF 10.
With this, DCI routers for VRF 10 are configured with two sets of import and export route-targets. One set is associated to L3VPN domain using VPNv4 to advertise layer-3 information; while the other set is for EVPN fabric using EVPN address-family for advertisement of routes. The separation of route-targets enables DCI routers to have two separate domains configured independently. In order for EVPN and L3VPN to interwork, “Stitching” keyword configuration under VRF is required to stitch the two set of route-targets. Below configuration is making EVPN RTs as stitching RTs, while the L3VPN remain normal RTs.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
vrf 10
 address-family ipv4 unicast
  import route-target
   <mark>10:10 stitching</mark>
   110:110
  !
  export route-target
   <mark>10:10 stitching</mark>
   110:110
  !
!
</code>
</pre>
</div>

We will import the evpn routes with stitching RT and then will re-originate these with vpnv4 towards PEs. For this we will need two knobs in BGP, “**import stitching-rt re-originate**” and “**advertise vpnv4 unicast re-originated**”.
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-evpn-to-vpnv4-advertisement.png?raw=true)

**Import EVPN routes using “import stitching-rt re-originate”:**  
In order to import evpn routes, we will have to import routes using “**stitching-rt**” keyword in evpn address-family. The “**re-originate**” keyword will enable the routes to be re-originated with vpnv4 normal RT (110:110).

Configure below on DCI-1 and DCI-2:
<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
DCI:

router bgp 65001
 neighbor 6.6.6.6
  remote-as 65001
  description "BGP-EVPN session to Spine-1"        
  address-family l2vpn evpn
   <mark>import stitching-rt re-originate</mark>
 !
 neighbor 7.7.7.7
  remote-as 65001
  description "BGP-EVPN session to Spine-2"       
  address-family l2vpn evpn
   <mark>import stitching-rt re-originate</mark>
!
 </code>
      </pre>
      </div>
      
As a result we can see the routes are programmed in the routing table of VRF 10 on DCI.
<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
DCI-1:

RP/0/RP0/CPU0:DCI-1#sh route vrf 10

Gateway of last resort is not set

B    10.0.0.20/32 [200/0] via 1.1.1.1 (nexthop in vrf default), 00:06:07
B    10.0.0.40/32 [200/0] via 2.2.2.2 (nexthop in vrf default), 00:06:07
B    111.1.1.1/32 [200/0] via 10.10.10.10 (nexthop in vrf default), 21:31:55
RP/0/RP0/CPU0:DCI-1#
 </code>
      </pre>
      </div>

**Re-originate evpn routes with vpnv4 RT “advertise vpnv4 unicast re-originated”:**  
Next we will advertise the routes learnt from EVPN fabric to L3VPN PE. Configure “**advertise vpnv4 unicast re-originated**” keyword under vpnv4 address family to re-originate the evpn routes matching stitching RT to vpnv4 using vpnv4 RT (110:110). Since, PE-1 does not have reachability to Leafs in EVPN fabric, DCI will act as inline-RR. DCI will change the next-hop to itself as it re-originates the routes and advertises to PE. We also need to configure “*ibgp policy out enforce-modifications*” to send the updated BGP route attributes to peers. 

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
 <mark>ibgp policy out enforce-modifications</mark>
 neighbor 9.9.9.9
  remote-as 65001
  description "vpnv4 session to DCI-1"  
  update-source Loopback0
  address-family vpnv4 unicast
   <mark>route-reflector-client</mark>
   <mark>advertise vpnv4 unicast re-originated</mark>
   next-hop-self
  !
 !
 neighbor 10.10.10.10
  remote-as 65001
  description "vpnv4 session to PE-1"    
  update-source Loopback0
  address-family vpnv4 unicast
   <mark>route-reflector-client</mark>
   <mark>advertise vpnv4 unicast re-originated</mark>
   next-hop-self
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
 <mark>ibgp policy out enforce-modifications</mark>
 neighbor 8.8.8.8
  remote-as 65001
  description "vpnv4 session to DCI-1"  
  update-source Loopback0
  address-family vpnv4 unicast
   <mark>route-reflector-client</mark>
   <mark>advertise vpnv4 unicast re-originated</mark>
   next-hop-self
  !
 !
 neighbor 10.10.10.10
  remote-as 65001
  description "vpnv4 session to PE-1"    
  update-source Loopback0
  address-family vpnv4 unicast
   <mark>route-reflector-client</mark>
   <mark>advertise vpnv4 unicast re-originated</mark>
   next-hop-self
  !
      </code>
      </pre>
      </div>
    </th>
  </tr>
</table>

Lets verify the routing table and BGP vpnv4 control-plane on PE-1.
<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
PE-1:

RP/0/RP0/CPU0:PE-1#sh route vrf 10

Gateway of last resort is not set

B    <mark>10.0.0.20/32</mark> [200/0] via 8.8.8.8 (nexthop in vrf default), 00:00:00
                  [200/0] via 9.9.9.9 (nexthop in vrf default), 00:00:00
B    <mark>10.0.0.40/32</mark> [200/0] via 8.8.8.8 (nexthop in vrf default), 00:00:00
                  [200/0] via 9.9.9.9 (nexthop in vrf default), 00:00:00
L    111.1.1.1/32 is directly connected, 1d00h, Loopback100
RP/0/RP0/CPU0:PE-1#


RP/0/RP0/CPU0:PE-1#sh bgp vpnv4 unicast rd 8.8.8.8:0 10.0.0.20/32 detail 
BGP routing table entry for 10.0.0.20/32, Route Distinguisher: 8.8.8.8:0
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker                207         207
    Flags: 0x00040001+0x00010200; 
Last Modified: Mar  8 14:39:48.767 for 1d13h
Paths: (2 available, best #1)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Flags: 0x4000000025060005, import: 0x3f
  Not advertised to any peer
  Local
    <mark>8.8.8.8</mark> (metric 10) from 8.8.8.8 (1.1.1.1)
      Received Label 64000
      Origin IGP, localpref 100, valid, internal, best, group-best, import-candidate, not-in-vrf
      Received Path ID 0, Local Path ID 0, version 207
      Extended community: SoO:1.1.1.1:10 0x060e:0000.0000.000a <mark>RT:110:110</mark> 
      <mark>Originator: 1.1.1.1</mark>, Cluster list: 8.8.8.8, 6.6.6.6
  Path #2: Received by speaker 0
  Flags: 0x4000000024020005, import: 0x16
  Not advertised to any peer
  Local
    <mark>9.9.9.9</mark> (metric 10) from 9.9.9.9 (1.1.1.1)
      Received Label 64002
      Origin IGP, localpref 100, valid, internal, import-candidate, not-in-vrf
      Received Path ID 0, Local Path ID 0, version 0
      Extended community: SoO:1.1.1.1:10 0x060e:0000.0000.000a <mark>RT:110:110</mark> 
      Originator: <mark>1.1.1.1</mark>, Cluster list: 9.9.9.9, 8.8.8.8, 6.6.6.6
RP/0/RP0/CPU0:PE-1#

RP/0/RP0/CPU0:PE-1#sh cef vrf 10 10.0.0.20/32
10.0.0.20/32, version 228, internal 0x5000001 0x0 (ptr 0x8d1ccacc) [1], 0x0 (0x0), 0x208 (0x8d9fe0e0)
 Updated Mar  8 14:46:37.085
 Prefix Len 32, traffic index 0, precedence n/a, priority 3
   via <mark>8.8.8.8/32</mark>, 5 dependencies, recursive, bgp-multipath [flags 0x6080]
    path-idx 0 NHID 0x0 [0x8cce6d08 0x0]
    recursion-via-/32
    next hop VRF - 'default', table - 0xe0000000
    next hop 8.8.8.8/32 via 16008/0/21
     next hop 192.8.10.1/32 BE81         labels imposed {ImplNull 64000}
   via <mark>9.9.9.9/32</mark>, 5 dependencies, recursive, bgp-multipath [flags 0x6080]
    path-idx 1 NHID 0x0 [0x8cce8268 0x0]
    recursion-via-/32
    next hop VRF - 'default', table - 0xe0000000
    next hop 9.9.9.9/32 via 16009/0/21
     next hop 192.9.10.1/32 BE91         labels imposed {ImplNull 64002}
RP/0/RP0/CPU0:PE-1#
 </code>
      </pre>
      </div>

The routing table on PE-1 shows the hosts routes of EVPN fabric are learnt in VRF 10. We have DCI-1 and DCI-2 as the next-hops to get to host prefixes in EVPN fabric. This accomplishes the reachability from PE-1 to host prefixes on Leafs. 

Next we will configure the DCI routers to re-originate the routes to its BGP EVPN neighbor that are received from PE-1 via vpnv4 address-family. This will need two knobs configured in BGP, “**import re-originate stitching-rt**” and “**advertise vpnv4 unicast re-originated stitching-rt**”.  
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-vpnv4-to-evpn-advertisement.png?raw=true)

**Re-originate VPNv4 routes using EVPN stitching RT “import re-originate stitching-rt”:**  
This is configured under vpnv4 address-family to enable import of vpnv4 routes with normal RT 100:100 and re-originate it with evpn stitching-rt. 

**Advertise re-originated routes to EVPN “advertise vpnv4 unicast re-originated”:**  
Lastly, configure “advertise vpnv4 unicast re-originated” keyword under evpn address family. This keyword will configure advertisement of vpnv4 routes to BGP evpn neighbors. The route targets will change to the stitching route target before advertising to EVPN neighbors. DCI advertises this as evpn route type 5.

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
 neighbor 6.6.6.6
  remote-as 65001
  description "BGP-EVPN session to Spine-1"              
  update-source Loopback0
  address-family l2vpn evpn
   import stitching-rt re-originate
   <mark>advertise vpnv4 unicast re-originated</mark>
   next-hop-self
  !
 !
 neighbor 7.7.7.7
  remote-as 65001
  description "BGP-EVPN session to Spine-2"             
  update-source Loopback0
  address-family l2vpn evpn
   import stitching-rt re-originate
   <mark>advertise vpnv4 unicast re-originated</mark>
   next-hop-self
  !
 !
 neighbor 9.9.9.9
  remote-as 65001
  description "vpnv4 session to DCI-1"    
  update-source Loopback0
  address-family vpnv4 unicast
   <mark>import re-originate stitching-rt</mark>
   route-reflector-client
   advertise vpnv4 unicast re-originated
   next-hop-self
  !
 !
 neighbor 10.10.10.10
  remote-as 65001
  description "vpnv4 session to PE-1"      
  update-source Loopback0
  address-family vpnv4 unicast
   <mark>import re-originate stitching-rt</mark>
   route-reflector-client
   advertise vpnv4 unicast re-originated
   next-hop-self
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
 neighbor 6.6.6.6
  remote-as 65001
  description "BGP session to Spine-1"       
  update-source Loopback0
  address-family l2vpn evpn
   import stitching-rt re-originate
   <mark>advertise vpnv4 unicast re-originated</mark>
   next-hop-self
  !
 !
 neighbor 7.7.7.7
  remote-as 65001
  description "BGP session to Spine-2"       
  update-source Loopback0
  address-family l2vpn evpn
   import stitching-rt re-originate
   <mark>advertise vpnv4 unicast re-originated</mark> 
   next-hop-self
  !
 !
 neighbor 8.8.8.8
  remote-as 65001
  description "vpnv4 session to DCI-1"   
  update-source Loopback0
  address-family vpnv4 unicast
   <mark>import re-originate stitching-rt</mark>
   route-reflector-client
   advertise vpnv4 unicast re-originated
   next-hop-self
  !
 !
 neighbor 10.10.10.10
  remote-as 65001
  description "vpnv4 session to PE-1"    
  update-source Loopback0
  address-family vpnv4 unicast
   <mark>import re-originate stitching-rt</mark>
   route-reflector-client
   advertise vpnv4 unicast re-originated
   next-hop-self
  !
 !
      </code>
      </pre>
      </div>
    </th>
  </tr>
</table>


Finally lets observe the routing table and BGP-EVPN control-plane on Leafs to verify PE-1 prefix is reachable.

<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
Leaf-1:

RP/0/RP0/CPU0:Leaf-1#sh route vrf 10

Gateway of last resort is not set

C    10.0.0.0/24 is directly connected, 2d16h, BVI10
L    10.0.0.1/32 is directly connected, 2d16h, BVI10
B    10.0.0.40/32 [200/0] via 2.2.2.2 (nexthop in vrf default), 1d18h
B    111.1.1.1/32 [200/0] via 8.8.8.8 (nexthop in vrf default), 1d16h
                  [200/0] via 9.9.9.9 (nexthop in vrf default), 1d16h
RP/0/RP0/CPU0:Leaf-1#

Leaf-2

RP/0/RP0/CPU0:Leaf-2#sh route vrf 10

Gateway of last resort is not set

C    10.0.0.0/24 is directly connected, 2d16h, BVI10
L    10.0.0.1/32 is directly connected, 2d16h, BVI10
B    10.0.0.20/32 [200/0] via 1.1.1.1 (nexthop in vrf default), 1d18h
B    111.1.1.1/32 [200/0] via 8.8.8.8 (nexthop in vrf default), 1d16h
                  [200/0] via 9.9.9.9 (nexthop in vrf default), 1d16h
RP/0/RP0/CPU0:Leaf-2#


Leaf-1

RP/0/RP0/CPU0:Leaf-1#sh bgp l2vpn evpn rd 8.8.8.8:0
BGP router identifier 1.1.1.1, local AS number 65001
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 0
BGP main routing table version 2235
BGP NSR Initial initsync version 2 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 8.8.8.8:0
*>i[5][0][32][111.1.1.1]/80
                      8.8.8.8                  0    100      0 ?
* i                   8.8.8.8                  0    100      0 ?

Processed 1 prefixes, 2 paths
RP/0/RP0/CPU0:Leaf-1#

RP/0/RP0/CPU0:Leaf-1#sh bgp l2vpn evpn rd 9.9.9.9:0
BGP router identifier 1.1.1.1, local AS number 65001
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 0
BGP main routing table version 2235
BGP NSR Initial initsync version 2 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 9.9.9.9:0
*>i[5][0][32][111.1.1.1]/80
                      9.9.9.9                  0    100      0 ?
* i                   9.9.9.9                  0    100      0 ?

Processed 1 prefixes, 2 paths
RP/0/RP0/CPU0:Leaf-1#


RP/0/RP0/CPU0:Leaf-1#sh bgp l2vpn evpn rd 8.8.8.8:0 [5][0][32][111.1.1.1]/80 detail
BGP routing table entry for [5][0][32][111.1.1.1]/80, Route Distinguisher: 8.8.8.8:0
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker                704         704
    Flags: 0x00040001+0x00010000; 
Last Modified: Apr 15 04:23:51.766 for 1d16h
Paths: (2 available, best #1)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Flags: 0x4000000025060005, import: 0x1f, EVPN: 0x1
  Not advertised to any peer
  Local
    8.8.8.8 (metric 20) from 6.6.6.6 (10.10.10.10)
      Received Label 64000 
      Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, not-in-vrf
      Received Path ID 0, Local Path ID 1, version 704
      Extended community: Flags 0x6: RT:10:10 
      Originator: 10.10.10.10, Cluster list: 6.6.6.6, 8.8.8.8
      EVPN ESI: 0000.0000.0000.0000.0000, Gateway Address : 0.0.0.0
  Path #2: Received by speaker 0
  Flags: 0x4000000020020005, import: 0x20, EVPN: 0x1
  Not advertised to any peer
  Local
    8.8.8.8 (metric 20) from 7.7.7.7 (10.10.10.10)
      Received Label 64000 
      Origin incomplete, metric 0, localpref 100, valid, internal, not-in-vrf
      Received Path ID 0, Local Path ID 0, version 0
      Extended community: Flags 0x6: RT:10:10 
      Originator: 10.10.10.10, Cluster list: 7.7.7.7, 8.8.8.8
      EVPN ESI: 0000.0000.0000.0000.0000, Gateway Address : 0.0.0.0


Reachability to Host prefixes from PE-1:
RP/0/RP0/CPU0:PE-1#ping vrf 10 10.0.0.20 source 111.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.20, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms

RP/0/RP0/CPU0:PE-1#ping vrf 10 10.0.0.40 source 111.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.40, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/3 ms
RP/0/RP0/CPU0:PE-1#
  </code>
      </pre>
      </div>
The routing table on Leafs has the prefix 111.1.1.1/32 from PE-1. The evpn control-plane shows the route is received from DCI-1 (8.8.8.8) while PE-1 (10.10.10.10) is the originator. We have DCI-1 and DCI-2 as the next-hop to get to PE-1 prefix. 
Successful Ping from PE-1 to Host prefixes shows that the BGP EVPN and L3VPN interworking is operational and end-to-end reachability between Leafs and PE-1 is established.
