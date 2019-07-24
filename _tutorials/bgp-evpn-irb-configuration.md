---
published: false
date: '2019-07-18 17:19 +0100'
title: BGP-EVPN IRB Configuration for Inter-Subnet Routing
author: Ahmad Bilal Siddiqui
excerpt: In this post we will cover the EVPN-IRB configuration on NCS5500
tags:
  - iosxr
  - ncs 5500
  - evpn
  - NCS5500
---
{% include toc %}
# Topic: Configure BGP-EVPN Integrated Routing and Bridging (IRB) for inter-subnet routing

EVPN Integrated Routing and Bridging (IRB) feature allows end hosts across the overlay to communicate with each other within the same subnet and across different subnets within the VPN. In this post we will cover the implementation of EVPN IRB to route between host-1 and host-9 being in different subnets. Distributed Anycast Gateway will be configured on Leaf-1 and Leaf-2 for subnet 10.0.0.0/24 and on Leaf-5 for subnet 20.0.0.0/24. 
After configuring IRB we will ping between the Hosts-1 and Host-9 to verify the reachability and observe the routes are learnt vie BGP EVPN. 

In [last post](https://xrdocs.io/ncs5500/tutorials/bgp-evpn-configuration-ncs-5500-part-3/) we configured the Layer-2 stretch between Leaf-1, Leaf-2 and Leaf-5 using BGP EVPN EVI 10 for VLAN 10. We don’t need VLAN 10 on Leaf-5 for this post, that is why we will remove the EVI 10 from Leaf-5.

## Reference Topology:
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/reference-topology-evpn-post-4.png?raw=true)

### Task 1: Configure the BGP-EVPN Distributed Anycast Gateway on Leaf-1 and Leaf-2
BGP EVPN provides Distributed anycast gateway feature that enables any Leaf in the fabric to serve as the active default gateway for a host in a subnet. Same virtual gateway IP address and virtual MAC address is configured on the BVI for each subnet across the Leafs enabling them to act as gateway for their locally connected hosts. Distributed anycast gateway also brings the advantage of seamless workload mobility. We will configure the GW BVI on the Leafs inside the VRF for this post, however we can also configure the BVI in the global routing table.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-irb-full-topology.png?raw=true)

A virtual routing and forwarding instance VRF, represents a tenant. This VRF will have the routes that belong to the overlay network for that tenant. The route-target values should be configured for the VRF to define which prefixes are exported and imported on the Leafs. As we will configure BVI under VRF, the related show commands and troubleshooting should point to the VRF.

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

Configure the VRF in BGP so that the layer-3 routes can be advertised to the Leafs for layer-3 inter-subnet routing. In order to advertise the VRF via BGP EVPN we will have to initiate the VPNv4 address family in BGP to provide the VRF capability in MP-BGP.
RD auto under VRF generates RD value automatically based on BGP **router-id:EVI-id**. We can also manually configure the RD value. We will use “redistribute connected” to advertise any connected routes via BGP.

<div class="highlighter-rouge">
<pre class="highlight">
<code>

Configure the following on Leaf-1, Leaf-2 and Leaf-5

      router bgp 65001
       address-family vpnv4 unicast
       !
       vrf 10
        rd auto
        address-family ipv4 unicast
         redistribute connected
      !
</code>
</pre>
</div>

Now, we will configure the BVI-10 for subnet 10.0.0.0/24 on Leaf-1 and Leaf-2 under VRF 10. The BVI will serve as the Distributed Anycast GW for subnet 10.0.0.0/24. Make sure the BVI IP address and MAC address are identical on Leaf-1 and Leaf-2.

<div class="highlighter-rouge">
<pre class="highlight">
<code>

      interface BVI 10
       host-routing
       vrf 10
       ipv4 address 10.0.0.1 255.255.255.0
       mac-address 1001.1001.1001
      !
</code>
</pre>
</div>

