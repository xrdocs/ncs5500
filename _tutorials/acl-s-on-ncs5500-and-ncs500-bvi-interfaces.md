---
published: true
date: '2021-09-07 12:37 +0530'
title: ACL's on NCS5500 and NCS500 BVI Interfaces
author: Tejas Lad
excerpt: This document will introduce the behaviour of ACL on BVI Interfaces
tags:
  - iosxr
  - NCS5500
  - NCS500
  - ACL
  - Access List
  - BVI
position: hidden
---
{% include toc icon="table" title="ACL's on NCS5500 and NCS500 BVI Interfaces" %} 

## Introduction

In the [previous article](https://xrdocs.io/ncs5500/tutorials/access-list-enhancements-on-ncs5500-j2-based-platforms/), we introduced the ACL enhancements on NCS5500 based on J2 chipsets. In this article, we will introduce the ACL implementation on NCS5500 w.r.t to BVI interfaces. We will cover all the platforms based on J/J+ and J2. 

## Quick Recap: Bridged Virtual Interface - BVI 

![Screenshot 2021-09-09 at 12.27.28 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-09 at 12.27.28 PM.png)


Before we move to the ACL features, let us do a quick recap of the BVI interface and understand its use cases. The [**BVI**](https://www.cisco.com/c/en/us/support/docs/lan-switching/integrated-routing-bridging-irb/17054-741-10.html) is a virtual interface within the router that acts like a normal routed interface. BVI provides link between the bridging and the routing domains on the router. The BVI does not support bridging itself, but acts as a gateway for the corresponding bridge-domain to a routed interface within the router. Bridge-Domain is a layer 2 broadcast domain
It is associated to a bridge group using the routed interface bvi command. 


### Main Use cases

  - Interconnect bridged and routed networks
  - Preserve network addresses
  - Bridge local traffic for efficient network performance

BVI provides a much more flexible solution for bridging and routing
  
| Scenarios                                                    | Supported by BVI |
|--------------------------------------------------------------|------------------|
| Communication of multiple interfaces in same BD              | Yes              |
| Communication of multiple interfaces in different BD         | Yes              |
| Communication between Bridged interface and Routed interface | Yes              |
  
  

## Configuring ACL and attaching to BVI interface

Configuration and attachment of an ACL over BVI interface is similar to regular ACLs attachment to a physical interface. Let us check out the same.

### Ingress V4 ACL

  - First we will start with Ingress IPv4 ACL. 
  - IPv4 ACL is supported in Igress direction for J/J+ and J2 (Native and Compatible mode)


<div class="highlighter-rouge">
<pre class="highlight">
<code>

RP/0/RP0/CPU0:5508-2-74142I-C#show platform
Node              Type                       State             Config state
--------------------------------------------------------------------------------               
<mark>0/3/CPU0          NC57-18DD-SE</mark>               IOS XR RUN        NSHUT
0/3/NPU0          Slice                      UP                
0/3/NPU1          Slice                      UP                
<mark>0/4/CPU0          NC55-36X100G-A-SE</mark>          IOS XR RUN        NSHUT
0/4/NPU0          Slice                      UP                
0/4/NPU1          Slice                      UP                
0/4/NPU2          Slice                      UP                
0/4/NPU3          Slice                      UP                                               
<mark>0/7/CPU0          NC55-24X100G-SE</mark>            IOS XR RUN        NSHUT
0/7/NPU0          Slice                      UP                
0/7/NPU1          Slice                      UP                
0/7/NPU2          Slice                      UP                
0/7/NPU3          Slice                      UP                
</code>
</pre>
</div>

Note: Output is truncated
{: .notice--info}

We will consider 3 Line cards.

| Line Card         | No of NPU's |
|-------------------|-------------|
| NC57-18DD-SE      | 2xJ2        |
| NC55-36X100G-A-SE | 4xJ+        |
| NC55-24X100G-SE   | 4xJ         |


We have ingress IPv4 ACL configured as below and applied to BVI interface 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>ipv4 access-list permit-stats</mark>
 10 permit ipv4 25.1.7.0 0.0.0.255 any
 20 deny ipv4 26.1.7.0 0.0.0.255 any
 30 permit ipv4 host 50.1.1.1 any
 35 deny ipv4 62.6.69.128 0.0.0.15 any
 40 deny ipv4 any 62.80.66.128 0.0.0.15
 45 deny ipv4 62.80.66.128 0.0.0.15 any
 50 deny ipv4 any 62.134.38.0 0.0.0.127
 60 permit tcp any eq bgp host 1.2.3.1
 70 permit tcp any host 1.2.3.1 eq bgp
 80 deny ipv4 any host 1.2.3.1
 90 deny ipv4 any 212.21.217.0 0.0.0.255
 100 permit ipv4 any any
 
 
 RP/0/RP0/CPU0:5508-2-74142I-C#show running-config interface bvI 21
 interface BVI21
 <mark>ipv4 access-group permit-stats ingress</mark>
!
</code>
</pre>
</div>

**BVI configurations**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
interface FourHundredGigE0/3/0/21
 <mark>l2transport</mark>
 !
!

l2vpn
 bridge group BVI
  bridge-domain 21
  <span style="background-color:pink">interface FourHundredGigE0/3/0/21</span>
   !
   <span style="background-color:pink">routed interface BVI21</span>
   !
  !
</code>
</pre>
</div>

#### Verification

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show access-lists ipv4 usage pfilter location all
<span style="background-color:pink">Interface : BVI21</span>
Input ACL : Common-ACL : N/A <span style="background-color:yellow">ACL : permit-stats</span>
Output ACL : N/A
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show controllers npu externaltcam location 0/3/CPU0 
External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
<span style="background-color:pink">0</span>    <span style="background-color:yellow">15     320b               4081     15      2725  ext_FG_INGR_V4_ACL</span>
<span style="background-color:pink">1</span>    <span style="background-color:yellow">15     320b               4081     15      2725  ext_FG_INGR_V4_ACL</span>

RP/0/RP0/CPU0:5508-2-74142I-C#
</code>
</pre>
</div>

Note: Output is truncated
{: .notice--info}

### TCAM Programming with BVI interfaces

  - BVI interfaces are designed in such a way that every feature attachment affects all NPUs on the Line Card.
  - Ingress ACL support over BVI interfaces is designed in a way that when an attach is triggered, the corresponding L2 interface's extlif value gets updated with the ACL ID being programmed.
- TCAM entries can be shared between different interfaces.
- TCAM entries are always programmed across all NPUs, regardless of interface membership. This is the platform replication.
- TCAM entries are always programmed across all LCs, regardless of interface membership. This is platform independent and should behave the same across all XR platforms. 
- To understand this behaviour, we need to recap what we discussed in the earlier section w.r.t BVI.
- BVI has no mapping to specific Physical port / LC.
- BVI is rather an entity of Bridge Domain. 
- In XR, BVI has been defined globally just as a Bridge Domain. Hence it is present on all LCs
- Another important consideration for this implementation is modification of the configurations
- If we want to add or remove BVI entries in each LC dynamically - based on the presence of a local AC (from that LC) in the given BD it has a lot of overhead. 
- Hence we allocate resources on all the NPU's of the LCs and also from PI perspective we replicate that on all LCs

#### TCAM entries on other LCs

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:5508-2-74142I-C#show controllers npu internaltcam location 0/4/CPU0</mark> 
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
<span style="background-color:pink">0</span>    <span style="background-color:yellow">2      160b   pmf-0       2023     16      47   INGRESS_ACL_L3_IPV4</span>

<span style="background-color:pink">1</span>    <span style="background-color:yellow">2      160b   pmf-0       2023     16      47   INGRESS_ACL_L3_IPV4</span>

<span style="background-color:pink">2</span>    <span style="background-color:yellow">2      160b   pmf-0       2023     16      47   INGRESS_ACL_L3_IPV4</span>

<span style="background-color:pink">3</span>    <span style="background-color:yellow">2      160b   pmf-0       2023     16      47   INGRESS_ACL_L3_IPV4</span>

RP/0/RP0/CPU0:5508-2-74142I-C#
</code>
</pre>
</div>

Note: Output is truncated
{: .notice--info}

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:5508-2-74142I-C#show controllers npu internaltcam location 0/7/CPU0</mark> 
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
<span style="background-color:pink">0</span>    <span style="background-color:yellow">2      160b   pmf-0       2023     16      47   INGRESS_ACL_L3_IPV4</span>

<span style="background-color:pink">1</span>    <span style="background-color:yellow">2      160b   pmf-0       2023     16      47   INGRESS_ACL_L3_IPV4</span>

<span style="background-color:pink">2</span>    <span style="background-color:yellow">2      160b   pmf-0       2023     16      47   INGRESS_ACL_L3_IPV4</span>

<span style="background-color:pink">3</span>    <span style="background-color:yellow">2      160b   pmf-0       2023     16      47   INGRESS_ACL_L3_IPV4</span>

RP/0/RP0/CPU0:5508-2-74142I-C#
</code>
</pre>
</div>

Note: Output is truncated
{: .notice--info}

So as per the explanation above, we could see the replication is happening on all the LCs in the system and all the NPUs on each LC. We can see that the programming of the ingress traditional ACLs is happening on the external TCAM for a LC based on J2 and on the internal TCAM for LCs based on J/J+. TCAM resources are used even if the LCs are not part of that particular BVI or BD.


### Ingress V6 ACL

Let us verify the behaviour with the IPv6 Ingress ACLs.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show access-lists ipv6 ipv6_1 
<mark>ipv6 access-list ipv6_1</mark>
 10 permit tcp 2001:1:2::/64 any eq 1024
 20 permit tcp 2002:1:2::/64 any eq 1024
 30 permit tcp 2003:1:2::/64 any eq 1024
 40 permit tcp 2004:1:2::/64 any eq 1024
 50 deny udp 2001:4:5::/64 any lt 1000
 60 permit ipv6 any any
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show running-config interface bvI 21
interface BVI21
 <mark>ipv6 access-group ipv6_1 ingress</mark>
!
</code>
</pre>
</div>


#### TCAM entries on all the LCs 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:5508-2-74142I-C#show controllers npu externaltcam location 0/3/CPU0</mark> 
External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================

<span style="background-color:pink">0</span>    <span style="background-color:yellow">12     480b               1998     50      2722  ext_FG_INGR_V6_ACL</span>

<span style="background-color:pink">1</span>    <span style="background-color:yellow">12     480b               1998     50      2722  ext_FG_INGR_V6_ACL</span>

RP/0/RP0/CPU0:5508-2-74142I-C#
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:5508-2-74142I-C#show controllers npu internaltcam location 0/4/CPU0</mark> 
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
<span style="background-color:pink">0</span>    <span style="background-color:yellow">6\7    320b   pmf-0       2025     23      48   INGRESS_ACL_L3_IPV6</span>

<span style="background-color:pink">1</span>    <span style="background-color:yellow">6\7    320b   pmf-0       2025     23      48   INGRESS_ACL_L3_IPV6</span>

<span style="background-color:pink">2</span>    <span style="background-color:yellow">6\7    320b   pmf-0       2025     23      48   INGRESS_ACL_L3_IPV6</span>

<span style="background-color:pink">3</span>    <span style="background-color:yellow">6\7    320b   pmf-0       2025     23      48   INGRESS_ACL_L3_IPV6</span>
    
RP/0/RP0/CPU0:5508-2-74142I-C#
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:5508-2-74142I-C#show controllers npu internaltcam location 0/7/CPU0</mark> 
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
<span style="background-color:pink">0</span>    <span style="background-color:yellow">6\7    320b   pmf-0       2025     23      48   INGRESS_ACL_L3_IPV6</span>

<span style="background-color:pink">1</span>    <span style="background-color:yellow">6\7    320b   pmf-0       2025     23      48   INGRESS_ACL_L3_IPV6</span>

<span style="background-color:pink">2</span>    <span style="background-color:yellow">6\7    320b   pmf-0       2025     23      48   INGRESS_ACL_L3_IPV6</span>

<span style="background-color:pink">3</span>    <span style="background-color:yellow">6\7    320b   pmf-0       2025     23      48   INGRESS_ACL_L3_IPV6</span>
    
RP/0/RP0/CPU0:5508-2-74142I-C#
</code>
</pre>
</div>

Note: Output is truncated
{: .notice--info}

We see the same behaviour in terms of programming and TCAM resource utilization when it comes to IPv6 ingress ACLs as well. IPv6 ACL is supported in Igress direction for J/J+ and J2 (Native and Compatible mode)


### Egress V4 ACL

  - By default, Egress ACLs over BVI interface is disabled and thus ACL filtering will not take effect even after it is attached to BVI interfaces.
  - To enable ACL over BVI in the egress direction, _**hw-module profile acl egress layer3 interface-based**_ should be configured.
  - When we enable the above profile, egress ACL on any non BVI interface will not work.
  - IPv4 ACL is supported in Egress direction for J/J+ and J2 (Native and Compatible mode)
  - TCAM entries are unique per interface, even for the same ACL.
  - TCAM entries are always programmed across all NPUs, regardless of interface membership. This is platform replication.
  - TCAM entries are always programmed across all LCs, regardless of interface membership. This is platform independent replication.
  - In order to activate/deactivate Egress ACL support over BVI interfaces, you must manually reload the chassis/all line cards.



### Egress V6 ACL


## Summary 

In summary, its more of design choice and optimization, but for sure it can be improved or changed as needed.

## References

- https://www.cisco.com/c/en/us/support/docs/lan-switching/integrated-routing-bridging-irb/17054-741-10.html
- https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/71x/configuration/guide/b-interfaces-hardware-component-cg-ncs5500-71x/m-adhoc-fretta-irb-unicast.html
