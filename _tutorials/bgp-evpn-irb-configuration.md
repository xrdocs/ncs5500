---
published: true
date: '2019-07-18 17:19 +0100'
title: BGP-EVPN IRB Configuration for Inter-Subnet Routing
author: Ahmad Bilal Siddiqui
excerpt: In this post we will cover the EVPN-IRB configuration on NCS5500
tags:
  - iosxr
  - ncs 5500
  - evpn
  - NCS5500
position: top
---
{% include toc %}

EVPN Integrated Routing and Bridging (IRB) feature allows end hosts across the overlay to communicate with each other within or across the subnets in the VPN. In this post we will cover the implementation of EVPN IRB to route between Host-1 and Host-9. Distributed Anycast Gateway will be configured on Leaf-1 and Leaf-2 for subnet 10.0.0.0/24 and on Leaf-5 for subnet 20.0.0.0/24. 
After configuring IRB we will ping between the Host-1 and Host-9 to verify the reachability and observe the routes are learnt vie BGP EVPN. 

In [last post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/) we configured the Layer-2 stretch between Leaf-1, Leaf-2 and Leaf-5 using BGP EVPN EVI 10 for VLAN 10. We don’t need VLAN 10 on Leaf-5 for this post, that is why EVI 10 and related Bridge Domain is removed from Leaf-5.

## Reference Topology:
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/reference-topology-evpn-post-4.png?raw=true)

### Task 1: Configure the BGP-EVPN Distributed Anycast Gateway on Leaf-1 and Leaf-2  
BGP EVPN provides Distributed anycast gateway feature that enables any Leaf in the fabric to serve as the active default gateway for a host in a subnet. Same virtual gateway IP address and virtual MAC address is configured on the BVI interface for each subnet across the Leafs enabling them to act as gateway for their locally connected hosts. Distributed anycast gateway brings the advantage of seamless workload mobility. 

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-irb-full-topology.png?raw=true)

We will configure the GW BVI on the Leafs inside the VRF for this post, however we can also configure the BVI in the global routing table.  
  
A virtual routing and forwarding instance VRF, represents a tenant. This VRF will have the routes that belong to the overlay network for that tenant. The route-target values should be configured for the VRF to define which prefixes are exported and imported on the Leafs. As we will configure BVI under VRF, the related show commands and troubleshooting should point to the VRF.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/IRB-low-level-details.png?raw=true)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Configure VRF 10 on Leaf-1, Leaf-2 and Leaf-5

      vrf 10
       address-family ipv4 unicast
        import route-target
         10:10
        !
        export route-target
         10:10
       !

Use “show vrf 10” to verify the config.
</code>
</pre>
</div> 

Configure the VRF in BGP to advertised the routes of the VRF to other Leafs. Initiate the VPNv4 address family to advertise VRF label. RD auto under VRF generates RD value automatically based on **[BGP-Router-ID:EVI-ID]**. However, configuring RD manually is also supported.  
We will use “redistribute connected” under VRF to advertise connected routes via BGP. In addition, we are configuring BGP multipathing for load balancing where multiple next-hops are available for a prefix. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Configure the following on Leaf-1, Leaf-2 and Leaf-5

      router bgp 65001
       <mark>address-family vpnv4 unicast</mark>
       !
       vrf 10
        rd auto
        <mark>address-family ipv4 unicast</mark>
         additional-paths receive
         maximum-paths ibgp 10
         redistribute connected
      !
</code>
</pre>
</div>

Now, we will configure the BVI-10 on Leaf-1 and Leaf-2 under VRF 10. The BVI will serve as the Distributed Anycast GW for subnet 10.0.0.0/24. Make sure the BVI IP address and MAC address are identical on Leaf-1 and Leaf-2. Configure "host-routing" under BVI interface to advertise route-type 2. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
      interface BVI 10
       host-routing
       vrf 10
       ipv4 address <mark>10.0.0.1 255.255.255.0</mark>
       mac-address <mark>1001.1001.1001</mark>
      !
</code>
</pre>
</div>

In order for the BVI interface to come up and serve as the gateway to the host connected to the Leaf, we will have to configure the host connectivity to the Leaf _(this is already configured in_ [post-2](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-2/) _and_ [post-3](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/)_)_. Also associate the BVI to a Bridge-Domain.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Associate the BVI interface to the bridge-domain. Configure the following on Leaf-1 and Leaf-2. 
    
    l2vpn
     bridge group bg-1
      bridge-domain <mark>bd-10</mark>
       interface Bundle-Ether 1.10
       !
       routed <mark>interface BVI 10</mark>
    !
</code>
</pre>
</div>

Verify that the BVI is up on Leaf-1 and Leaf-2. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
      Leaf-1

      RP/0/RP0/CPU0:Leaf-1#show ip interface brief

      Interface                      IP-Address      Status          Protocol Vrf-Name
      BVI10                          10.0.0.1        Up              Up       10

      Leaf-2

      RP/0/RP0/CPU0:Leaf-2#show ip interface brief

      Interface                      IP-Address      Status          Protocol Vrf-Name
      BVI10                          10.0.0.1        Up              Up       10      
</code>
</pre>
</div>

  
**Reference config of Host-1 with default route to BVI interface on Leaf-1 and Leaf-2 serving as Gateway:**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
     Host-1
     
      interface Bundle-Ether1.10
      description "Dual-homed Bundle to Leaf-1 and Leaf-2"
       ipv4 address 10.0.0.10 255.255.255.0
       encapsulation dot1q 10
      !

      router static
       address-family ipv4 unicast
        0.0.0.0/0 10.0.0.1
       !
</code>
</pre>
</div>


### Task 2: Configure the BGP-EVPN Distributed Anycast Gateway on Leaf-5

Configure the BVI for subnet 20.0.0.0/24 on Leaf-5. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
      interface BVI 20
       host-routing
       vrf 10
       ipv4 address <mark>20.0.0.1 255.255.255.0</mark>
       mac-address <mark>1001.1001.2002</mark>
      !
</code>
</pre>
</div>

Associate the BVI to a Bridge-Domain and add the Host's attachment circuit to the Bridge Domain. The BVI will come up once the host connectivity to the Leaf is configured. For Host's connectivity, configure layer-2 interface with dot1q encapsulation for VLAN 20 on Leaf-5.
<div class="highlighter-rouge">
<pre class="highlight">
<code>
     Leaf-5 
     
      interface TenGigE0/0/0/45.20 l2transport
       encapsulation dot1q 20
       rewrite ingress tag pop 1 symmetric
      !

Configure Bridge domain for the VLAN 20 and add the VLAN tagged interface to the bridge-domain. Configure the following on Leaf-5.

     Leaf-5
     
      l2vpn
       bridge group bg-1
        bridge-domain bd-20
         interface TenGigE0/0/0/45.20
      ! 
</code>
</pre>
</div>

Associate the BVI interface to the bridge-domain. Configure the following on Leaf-5. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
     Leaf-5
      
      l2vpn
       bridge group bg-1
        bridge-domain bd-20
             interface TenGigE0/0/0/45.20
         !
         <mark>routed interface BVI 20</mark>
      !
</code>
</pre>
</div>

Verify that the BVI is up on Leaf-5. 
<div class="highlighter-rouge">
<pre class="highlight">
<code>
Leaf-5

RP/0/RP0/CPU0:Leaf-5#show ip interface brief

Interface                      IP-Address      Status          Protocol Vrf-Name
BVI20                          <mark>20.0.0.1</mark>        <mark>Up</mark>              <mark>Up</mark>       10      
</code>
</pre>
</div>
 

**Reference config of Host-9 with default route to BVI interface on Leaf-5 serving as Gateway:**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
     Host-9
     
      interface TenGigE0/0/1/3.20
       description "Link to Leaf-5"
       ipv4 address 20.0.0.50 255.255.255.0
       encapsulation dot1q 20
      !
      router static
       address-family ipv4 unicast
        0.0.0.0/0 20.0.0.1
       !
</code>
</pre>
</div>

Configure EVI under EVPN config on Leaf-5 to create EVPN service for VLAN 20. This EVI 20 will then be associated to the Bridge-Domain for VLAN 20. Assign the route-target values for the EVI to import and export prefixes via BGP EVPN control-plane.  
In the below configuration route-target is manually configured, however route-target can be automatically generated as well, based on [BGP-AS]:[EVI-ID].

<div class="highlighter-rouge">
<pre class="highlight">
<code>
       Leaf-5

        evpn
         <mark>evi 20</mark>
          bgp
           route-target import <mark>1001:22</mark>
           route-target export <mark>1001:22</mark>
          !
         !
</code>
</pre>
</div>

Associate the EVI 20 to Bridge-Domain for VLAN 20 that has attachment-circuit/host-9 connected.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
       Leaf-5

        l2vpn
         bridge group bg-1
          bridge-domain <mark>bd-20</mark>
           interface TenGigE0/0/0/45.20
           !
           routed interface BVI20
           !
           <mark>evi 20</mark>
           !
          !
</code>
</pre>
</div>


Lets check the host reachability by pinging from Host-1 (IP 10.0.0.10/32) to Host-9 (IP 20.0.0.50/32).  
In the below output we can see that we can ping between the Host-1 (IP 10.0.0.10) and Host-9 (IP 20.0.0.50) successfully which are both on different subnets.

<div class="highlighter-rouge">
<pre class="highlight">
<code>

      Host-1:

      RP/0/RSP0/CPU0:Host-1#ping <mark>20.0.0.50</mark>
      Type escape sequence to abort.
      Sending 5, 100-byte ICMP Echos to 20.0.0.50, timeout is 2 seconds:
      !!!!!
      Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
      RP/0/RSP0/CPU0:Host-1#


      Host-9:

      RP/0/RSP0/CPU0:Host-9#ping <mark>10.0.0.10</mark>
      Type escape sequence to abort.
      Sending 5, 100-byte ICMP Echos to 10.0.0.10, timeout is 2 seconds:
      !!!!!
      Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
      RP/0/RSP0/CPU0:Host-9#

</code>
</pre>
</div>

We can verify the routes advertisement using the BGP EVPN control-plane. In the below output from Leaf-5 we can see the MAC and IP address of Host-1 are learnt under their respective route distinguishers via EVPN Route-Type-2.

Example of Host-1 MAC+IP learnt via Route-Type-2 **([2][0][48][6c9c.ed6d.1d8b][32][10.0.0.10]/136)**

The route distinguisher value is comprised of **[BGP-Router-ID:EVI-ID] eg**. for **Leaf-1: 1.1.1.1:10, Leaf-2: 2.2.2.2:10**.

<div class="highlighter-rouge">
<pre class="highlight">
<code>

      Leaf-5:
      
        RP/0/RP0/CPU0:Leaf-5#show bgp l2vpn evpn 
        BGP router identifier 5.5.5.5, local AS number 65001
        BGP generic scan interval 60 secs
        Non-stop routing is enabled
        BGP table state: Active
        Table ID: 0x0   RD version: 0
        BGP main routing table version 147
        BGP NSR Initial initsync version 10 (Reached)
        BGP NSR/ISSU Sync-Group versions 0/0
        BGP scan interval 60 secs

        Status codes: s suppressed, d damped, h history, * valid, > best
                      i - internal, r RIB-failure, S stale, N Nexthop-discard
        Origin codes: i - IGP, e - EGP, ? - incomplete
           Network            Next Hop            Metric LocPrf Weight Path
        Route Distinguisher: 1.1.1.1:10
        *>i<mark>[2][0][48][6c9c.ed6d.1d8b][32][10.0.0.10]/136</mark>
                              1.1.1.1                       100      0 i
        * i                   1.1.1.1                       100      0 i
        Route Distinguisher: 2.2.2.2:10
        *>i<mark>[2][0][48][6c9c.ed6d.1d8b][32][10.0.0.10]/136</mark>
                              2.2.2.2                       100      0 i
        * i                   2.2.2.2                       100      0 i
        Route Distinguisher: 5.5.5.5:20 (default for vrf bd-20)
        *> [2][0][48][a03d.6f3d.5447][32][20.0.0.50]/136
                              0.0.0.0                                0 i
        *> [3][0][32][5.5.5.5]/80
                              0.0.0.0                                0 i

        Processed 4 prefixes, 6 paths
        RP/0/RP0/CPU0:Leaf-5#

      Similarly, on Leaf-1 and Leaf-2 we can see the prefix learnt that is advertised by Leaf-5.

      Leaf-1:
      
        RP/0/RP0/CPU0:Leaf-1#show bgp l2vpn EVpn rd 5.5.5.5:20
        BGP router identifier 1.1.1.1, local AS number 65001
        BGP generic scan interval 60 secs
        Non-stop routing is enabled
        BGP table state: Active
        Table ID: 0x0   RD version: 0
        BGP main routing table version 3911
        BGP NSR Initial initsync version 5 (Reached)
        BGP NSR/ISSU Sync-Group versions 0/0
        BGP scan interval 60 secs

        Status codes: s suppressed, d damped, h history, * valid, > best
                      i - internal, r RIB-failure, S stale, N Nexthop-discard
        Origin codes: i - IGP, e - EGP, ? - incomplete
           Network            Next Hop            Metric LocPrf Weight Path
        Route Distinguisher: 5.5.5.5:20
        *>i<mark>[2][0][48][a03d.6f3d.5447][32][20.0.0.50]/136</mark>
                              5.5.5.5                       100      0 i
        * i                   5.5.5.5                       100      0 i

        Processed 1 prefixes, 2 paths
        RP/0/RP0/CPU0:Leaf-1#


      Leaf-2:
      
        RP/0/RP0/CPU0:Leaf-2#show bgp l2vpn evpn rd 5.5.5.5:20
        BGP router identifier 2.2.2.2, local AS number 65001
        BGP generic scan interval 60 secs
        Non-stop routing is enabled
        BGP table state: Active
        Table ID: 0x0   RD version: 0
        BGP main routing table version 3947
        BGP NSR Initial initsync version 5 (Reached)
        BGP NSR/ISSU Sync-Group versions 0/0
        BGP scan interval 60 secs

        Status codes: s suppressed, d damped, h history, * valid, > best
                      i - internal, r RIB-failure, S stale, N Nexthop-discard
        Origin codes: i - IGP, e - EGP, ? - incomplete
           Network            Next Hop            Metric LocPrf Weight Path
        Route Distinguisher: 5.5.5.5:20
        *>i<mark>[2][0][48][a03d.6f3d.5447][32][20.0.0.50]/136</mark>
                              5.5.5.5                       100      0 i
        * i                   5.5.5.5                       100      0 i

        Processed 1 prefixes, 2 paths
        RP/0/RP0/CPU0:Leaf-2#
</code>
</pre>
</div>

When a host is discovered through ARP, the MAC and IP Route Type 2 is advertised with both Bridge-Domain/EVI label and IP VRF label with their respective route-targets. The VRF route-targets and IP VPN labels are associated with Route Type-2 to achieve Leaf-Leaf IP routing similar to traditional L3VPNs. For Layer-2 forwarding between Leaf-Leaf, the Bridge-Domain/EVI route-targets and labels associated with the Route Type 2 are used.  
In the below output on Leaf-5 for the prefix learnt from Leaf-1 (RD 1.1.1.1:10), we can see the highlighted route-target and label values.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
    Leaf-5
      RP/0/RP0/CPU0:Leaf-5#show bgp l2vpn evpn rd 1.1.1.1:10 [2][0][48][6c9c.ed6d.1d8b][32][10.0.0.10]/136 detail
      <mark>BGP routing table entry for [2][0][48][6c9c.ed6d.1d8b][32][10.0.0.10]/136, Route Distinguisher: 1.1.1.1:10</mark>
      Versions:
        Process           bRIB/RIB  SendTblVer
        Speaker                209         209
          Flags: 0x00840001+0x00010000; 
      Last Modified: Jul 25 19:37:14.072 for 00:01:17
      Paths: (2 available, best #1)
        Not advertised to any peer
        Path #1: Received by speaker 0
        Flags: 0x4000000025060005, import: 0x1f, EVPN: 0x3
        Not advertised to any peer
        Local
          1.1.1.1 (metric 20) from 6.6.6.6 (1.1.1.1)
            <mark>Received Label 24060, Second Label 24004</mark>
            Origin IGP, localpref 100, valid, internal, best, group-best, import-candidate, not-in-vrf
            Received Path ID 0, Local Path ID 1, version 209
            Extended community: Flags 0x1e: SoO:2.2.2.2:10 <mark>RT:10:10 RT:1001:11</mark> 
            Originator: 1.1.1.1, Cluster list: 6.6.6.6
            EVPN ESI: 0011.1111.1111.1111.1111
        Path #2: Received by speaker 0
        Flags: 0x4000000020020005, import: 0x40, EVPN: 0x3
        Not advertised to any peer
        Local
          1.1.1.1 (metric 20) from 7.7.7.7 (1.1.1.1)
            <mark>Received Label 24060, Second Label 24004</mark>
            Origin IGP, localpref 100, valid, internal, not-in-vrf
            Received Path ID 0, Local Path ID 0, version 0
            Extended community: Flags 0x1e: SoO:2.2.2.2:10 <mark>RT:10:10 RT:1001:11</mark> 
            Originator: 1.1.1.1, Cluster list: 7.7.7.7
            EVPN ESI: 0011.1111.1111.1111.1111
      RP/0/RP0/CPU0:Leaf-5#
</code>
</pre>
</div>

Lets check the routing table of VRF 10 on the Leafs. In below output we can see that 10.0.0.10/32 and 20.0.0.50/32 prefixes are being learnt on the Leafs.

<div class="highlighter-rouge">
<pre class="highlight">
<code>

      Leaf-1:
          RP/0/RP0/CPU0:Leaf-1#show route vrf 10
          Gateway of last resort is not set

          C    10.0.0.0/24 is directly connected, 02:47:36, BVI10
          L    10.0.0.1/32 is directly connected, 02:47:36, BVI10
          B    10.0.0.10/32 [200/0] via 2.2.2.2 (nexthop in vrf default), 00:55:24
          B    <mark>20.0.0.50/32 [200/0] via 5.5.5.5 (nexthop in vrf default), 00:26:54</mark>
          RP/0/RP0/CPU0:Leaf-1#

      Leaf-2:
          RP/0/RP0/CPU0:Leaf-2#show route vrf 10                 
          Gateway of last resort is not set

          C    10.0.0.0/24 is directly connected, 02:48:31, BVI10
          L    10.0.0.1/32 is directly connected, 02:48:31, BVI10
          B    10.0.0.10/32 [200/0] via 1.1.1.1 (nexthop in vrf default), 00:56:15
          B    <mark>20.0.0.50/32 [200/0] via 5.5.5.5 (nexthop in vrf default), 00:27:45</mark>
          RP/0/RP0/CPU0:Leaf-2#

      Leaf-5:
          RP/0/RP0/CPU0:Leaf-5#show route vrf 10
          Gateway of last resort is not set

          B    <mark>10.0.0.10/32 [200/0] via 1.1.1.1 (nexthop in vrf default), 00:39:32</mark>
                            <mark>[200/0] via 2.2.2.2 (nexthop in vrf default), 00:39:32</mark>
          C    20.0.0.0/24 is directly connected, 1d19h, BVI20
          L    20.0.0.1/32 is directly connected, 1d19h, BVI20
          RP/0/RP0/CPU0:Leaf-5#
</code>
</pre>
</div>

Lastly, we verify the CEF table for Host-1's prefix (10.0.0.10/32) on Leaf-5. We can see that we have ECMP paths available to reach to Host-1 and BGP multipathing is operational.  
<div class="highlighter-rouge">
<pre class="highlight">
<code>
    Leaf-5

      RP/0/RP0/CPU0:Leaf-5#show cef vrf 10 10.0.0.10/32
      10.0.0.10/32, version 16, internal 0x5000001 0x0 (ptr 0x97d2f7fc) [1], 0x0 (0x0), 0x208 (0x98aa3138)
       Updated Jul 25 17:45:29.253
       Prefix Len 32, traffic index 0, precedence n/a, priority 3
         via <mark>1.1.1.1/32</mark>, 3 dependencies, recursive, <mark>bgp-multipath</mark> [flags 0x6080]
          path-idx 0 NHID 0x0 [0x96e3ba50 0x0]
          recursion-via-/32
          next hop VRF - 'default', table - 0xe0000000
          next hop 1.1.1.1/32 via 16001/0/21
           next hop <mark>192.5.6.1/32 BE56</mark>         labels imposed {16001 24004}
           next hop <mark>192.5.7.1/32 BE57</mark>         labels imposed {16001 24004}
         via <mark>2.2.2.2/32</mark>, 3 dependencies, recursive, bgp-multipath [flags 0x6080]
          path-idx 1 NHID 0x0 [0x96e3bbf0 0x0]
          recursion-via-/32
          next hop VRF - 'default', table - 0xe0000000
          next hop 2.2.2.2/32 via 16002/0/21
           next hop <mark>192.5.6.1/32 BE56</mark>         labels imposed {16002 24004}
           next hop <mark>192.5.7.1/32 BE57</mark>         labels imposed {16002 24004}
      RP/0/RP0/CPU0:Leaf-5#
</code>
</pre>
</div>

After the configuration and verifcation we are able to perform inter-subnet routing between Host-1 and Host-5. This was achieved with BGP EVPN Integrated Routing and Bridging (IRB) feature along with Distributed Anycast Gateway. For deep dive details of BGP EVPN, refer to our **[e-vpn.io](https://e-vpn.io/)** webpage, it has a lot of material explaining the core concepts of EVPN, its operations and troubleshooting details.
