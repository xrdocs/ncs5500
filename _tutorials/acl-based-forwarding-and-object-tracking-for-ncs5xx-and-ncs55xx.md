---
published: true
date: '2020-09-08 14:36 +0530'
title: ACL Based Forwarding and Object Tracking for NCS5xx and NCS55xx
author: Tejas Lad
excerpt: >-
  This document aims to cover the concept of ABF and the used of object tracking
  with ABF
tags:
  - NCS5500
  - NCS500
  - NCS5xx
  - NCS55xx
  - ACL
  - ABF
  - ABF_OT
position: hidden
---
{% include toc %}

## Introduction

In today’s converged networks, operators have to implement packet forwarding in a way that goes beyond traditional routing protocol. Sometimes operators want certain traffic to be engineered based on rules which takes a separate path then what is already computed by the dynamic routing protocols. ACL Based Forwarding (ABF) can be used as a technique to achieve the same. ABF is a subset of PBR (Policy Based Routing) infrastructure in the XR platform. ABF allows traffic matching specific ACL rule to be forwarded to user specified nexthop instead of route selected by a routing protocol. ABF does not modify anything in the packet itself and therefore only affects the local routing/forwarding decision. The path for reaching the ABF next-hop, however, is determined by the normal routing/forwarding table.

![Screenshot 2020-09-08 at 3.23.09 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 3.23.09 PM.png)

## Different Types of ABF 

There are 3 types of ABF supported on NCS55xx and NCS5xx [Reference](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/72x/b-ip-addresses-cg-ncs5500-72x/m-implementing-access-lists-prefix-lists-ncs5500.html "Reference")

  - **ABF for IPv4/IPv6**
  
    - Only the next-hop IP address is specified in the ACE rule.
	- The matching traffic is forwarded to the first "up" next-hop, as specified in the ACE. 
    - Default VRF is used in this type of ABF.
    
    ![Screenshot 2020-09-08 at 4.21.12 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 4.21.12 PM.png)


  - **VRF-aware ABF for IPv4/IPv6**
  
    - Both the next-hop IP address and the next-hop VRF are specified in the ACE rule.
    - The matching traffic is forwarded to the first "up" next-hop, as specified in the ACE. 
    - The specified VRF is used instead of the default VRF for determining the path.
    
    ![Screenshot 2020-09-08 at 4.21.19 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 4.21.19 PM.png)


  - **VRF-select ABF for IPv4/IPv6** 
  
    - Only the next-hop VRF(s) is specified in the ACE rule.
    - The matching traffic is forwarded to the first “UP” VRF as specified in the ACE rule.
    - Supported from IOS-XR 6.5.1 onwards.
    
    ![Screenshot 2020-09-08 at 4.21.26 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 4.21.26 PM.png)


## Possible Use cases of ABF 
   (Reference 1)

**Source-Based Transit Provider Selection** – Internet service providers and other organizations can use ABF to route traffic originating from different sets of users through different internet connections across the policy routers.

**Cost Savings** – Organizations can achieve cost savings by distributing interactive and batch traffic among low-bandwidth, low-cost permanent paths and high-bandwidth, high-cost, switched paths.

**Load Sharing** – In addition to the dynamic load-sharing capabilities offered by destination-based routing that the Cisco IOS-XR software provides network manager can implement policies to distribute traffic among multiple paths based on the traffic characteristics.


## ABF Feature Support

- ABF is supported only in Ingress Direction. 
- ABF is not supported in the Egress direction
- ABF is supported only for IPv4 and IPv6. 
- ABF is not supported for L2 ACL.
- ABF is supported upto only 3 next-hops. The next-hops are selected on the basis of its configuration in the ACE.
- It is supported only for permit action. 
- Deny action is not supported with ABF.
- ABF is supported on Physical Interfaces, Sub-interfaces, Bundle Interfaces and Bundle Sub-Interfaces.
- ABF is not supported for common-ACL's.
- ABF supports dynamic next-hop modifications.
- ABF default route is not supported.
- IPv4 ABF next hops routed over GRE interfaces is supported.

## ABF Platform Implementation 

Now that we have understood the feature and its use cases, let us jump into the configuration and implementaion on NCS55xx and NCS5xx. We will be a taking a very simple topology to illustrate the feature.

![Screenshot 2020-09-08 at 7.11.34 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 7.11.34 PM.png)

In the above topology, we have network 70.1.1.0/24 behind router R5 and network 60.1.1.0/24 behind router R1. We have configured two ISIS neighbors, but the path calculated is just via TenGig 0/0/0/6 as we have higher metric on TenGig 0/0/0/7


<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:N55-24#show isis neighbors 
IS-IS 1 neighbors:
System Id      Interface        SNPA           State Holdtime Type IETF-NSF
N540-49        Te0/0/0/7        *PtoP*         Up    26       L2   Capable 
N540-49        Te0/0/0/6        *PtoP*         Up    28       L2   Capable</mark>
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show route 70.1.1.2
Routing entry for 70.1.1.0/24
  Known via "isis 1", distance 115, metric 20, type level-2
  Installed Sep  8 08:19:09.054 for 05:30:18
  Routing Descriptor Blocks
   <mark>65.1.1.2, from 172.16.4.49, via TenGigE0/0/0/6</mark>
      Route metric is 20
  No advertising protos. 
</code>
</pre>
</div>

As we mentioned above, let us apply an ABF to influence the critical traffic to take a path as per our configured next-hop.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
ipv4 access-list ABF_Test
 <mark>10 permit ipv4 any any dscp af11 nexthop1 ipv4 65.1.1.2 nexthop2 ipv4 70.1.1.1
 20 permit ipv4 any any dscp ef nexthop1 ipv4 66.1.1.2 nexthop2 ipv4 70.1.1.1</mark>
</code>
</pre>
</div>

This ACL will sent the traffic destined for 70.1.1.2 with DSCF AF11 via interface TenGih 0/0/0/6 and traffic with DSCP EF (critical traffic) via TenGig 0/0/0/7. Let us verify the same with traffic.

## ABF Verification

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 usage pfilter location 0/0/CPU0 
<mark>Interface : TenGigE0/0/0/0.10 
    Input  ACL : Common-ACL : N/A  ACL : ABF_Test</mark>  
    Output ACL : N/A
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 ABF_Test hardware ingress location 0/0/CPU0 
<mark>ipv4 access-list ABF_Test
 10 permit ipv4 any any dscp af11 (next-hop: addr=65.1.1.2, vrf name=default)
 20 permit ipv4 any any dscp ef (next-hop: addr=66.1.1.2, vrf name=default)</mark>
</code>
</pre>
</div>

![Screenshot 2020-09-08 at 8.05.04 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 8.05.04 PM.png)


![Screenshot 2020-09-08 at 8.19.07 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 8.19.07 PM.png) 

We can see the traffic (10000 packets of AF11 and 20000 packets of EF) is flowing fine. 

![Screenshot 2020-09-08 at 11.05.55 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 11.05.55 PM.png)


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 ABF_Test hardware ingress location 0/0/CPU0 
ipv4 access-list ABF_Test
 <mark>10 permit ipv4 any any dscp af11 (99377687 matches) (next-hop: addr=65.1.1.2, vrf name=default)
 20 permit ipv4 any any dscp ef (198755422 matches) (next-hop: addr=66.1.1.2, vrf name=default)</mark>
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 ABF_Test hardware ingress detail location 0/0/CPU0 
<mark>ABF_Test Details:
Sequence Number: 10</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 1
ACE Action: PERMIT</mark>
ACE Logging: DISABLED
<mark>ABF Action: 1(ABF_NH)
ABF IP: 65.1.1.2
ABF VRF: default</mark>
ABF FEC ID: 0x2001ffb8
<mark>Hit Packet Count: 99688849</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8ED2D0A8
        DSCP: 0x28 (Mask 0xFC)
<mark>Sequence Number: 20</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 1
ACE Action: PERMIT</mark>
ACE Logging: DISABLED
<mark>ABF Action: 1(ABF_NH)
ABF IP: 66.1.1.2
ABF VRF: default</mark>
ABF FEC ID: 0x2001ffb9
<mark>Hit Packet Count: 199377727</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8ED2D498
        DSCP: 0xB8 (Mask 0xFC)
Sequence Number: IMPLICIT DENY
NPU ID: 0
Number of DPA Entries: 1
ACL ID: 1
ACE Action: DENY
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
Hit Packet Count: 0
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8ED2D888
</code>
</pre>
</div>

From the above outputs we can see that how we could influence the traffic to take a path which is not there in the routing table. This can be used by the operators for diverting the traffic to load balance or for purpose of troubleshooting.



## Reference

  - Reference 1: https://community.cisco.com/t5/service-providers-documents/asr9000-xr-abf-acl-based-forwarding/ta-p/3153403
  - 


