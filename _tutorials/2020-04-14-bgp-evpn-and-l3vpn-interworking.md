---
published: true
date: '2020-04-14 16:35 -0700'
title: BGP-EVPN and L3VPN Interworking
author: Ahmad Bilal
excerpt: BGP-EVPN and L3VPN Interworking Support on NCS5500
---
{% include toc %}

## BGP-EVPN and L3VPN Interworking Support on NCS5500

BGP EVPN and L3VPN interworking is a way to connect EVPN domain such as a DC or CO over an IPVPN network. This is generally needed when we want end-to-end connectivity of Hosts/CEs in EVPN domain to connect to other domains over an IPVPN network providing inter-subnet routing.
Below topology shows an EVPN domain connecting to L3VPN domain, where interworking is done on Boarder-Leaf/DCI routers. Boarder-Leaf/DCI routers are essential in these types of designs to keep prefixes local to domain and send summarized advertisement out.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-topology.png?raw=true)

In this post we will go over the configuration of EVPN and L3VPN interworking on NCS 5500 routers acting as DCI. When we complete the configuration, we will see the Host subnet (10.0.0.0/24) learnt on PE-1, and PE-1’s loopback-100 prefix learnt on Leafs. This will provide end-to-end reachability between Hosts and PE-1’s prefixes, that are advertised by different BGP address families. The configuration setup is based on single BGP AS 65001 but separate ISIS routing domains for EVPN and L3VPN. DCI is participating in both ISIS domains performing EVPN and L3VPN interworking.

As we look at the setup, there are four major tasks we have to implement to achieve end-to-end connectivity. Some of these tasks are already covered in previous posts, their details will not be covered here. Click on the links below visit previous posts. 
•	Configure Segment Routing
•	Configure EVPN Layer-2 Service and IRB
•	Configure L3VPN
•	Configure DCI and Leafs to support EVPN and L3VPN interworking


### Task 1: Configuration of Segment Routing on DCI.
Segment routing configuration is covered in earlier post. For this we will show segment routing configuration of DCI as DCI is part of two segment routing domains. One segment routing domain is providing forwarding to EVPN fabric and the other to L3VPN domain.

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

<table border="0">
  <tr>
    <th>DCI-1</th>
    <th>DCI-2</th>
  </tr>
  <tr>
    <th>
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
    <th>
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
  
  
### Task 2: Configuration of L3VPN on DCIs and PE-1

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
PE-1:

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
DCI-1:

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
DCI-2:

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