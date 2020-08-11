---
published: true
date: '2020-08-10 19:08 +0530'
title: ACL IP Fragments Matching - NCS55xx and NCS5xx
author: Tejas Lad
excerpt: >-
  This document provides the capabilities of NCS5xx and NCS55xx to secure the
  data plane by filtering the fragmented packets
tags:
  - NCS5500
  - ACL
  - IP FRAGMENTS
  - NCS560
  - NCS540
  - NCS5xx
  - NCS55xx
position: top
---
{% include toc %}


## Introduction

In the previous [TechNote](https://xrdocs.io/ncs5500/tutorials/acl-packet-length-matching-ncs55xx-and-ncs5xx/ "TechNote"), we covered the matching criteria on the basis of Packet Length. In this note, we will discuss how customers can protect the data-plane from Fragmented Packets. On many occasions,fragmented packets are not expected in the network. It becomes very important for network administrators to add that extra layer of filtering. Hardware should be capable of filtering incoming packets on the basis of fragment flags and offset values. In this document, we will explore yet another filtering capability of NCS55xx and NCS5xx.

## Fragmentation

Let us have a quick refresher, before jumping into the implementation. In simple words, IP fragmentation can be considered a process of breaking down large packets into smaller fragments. This happens when intermediate devices has lower MTU on the interface than the arriving packet.

The packet is reassembled later with various fields of the IP packet.

**IP Header**

![Screenshot 2020-08-10 at 7.22.16 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.22.16 PM.png)

Below are the important fields during fragmentation and reassembling of the original packet.
  - Identification
  - Flags
  - Offset
  - Source and Destination Addresses
 

Let us take a simple example.

![Screenshot 2020-08-10 at 7.23.32 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.23.32 PM.png)

In the above figure, all the links are default size of 1500 bytes, except the link connected between R1 and H1. We have configured it as 1000 bytes. When a packet of 1500 byte arrives on the interface, R1 has to fragment the packet. The packet will be fragmented as below

![Screenshot 2020-08-10 at 7.23.42 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.23.42 PM.png)

For more information on IP fragmentation please [Refer](https://www.cisco.com/c/en/us/support/docs/ip/generic-routing-encapsulation-gre/25885-pmtud-ipfrag.html#anc2 "Refer")


## IPv4 Packet Types w.r.t Fragmentation

IPv4 Packets fall into the below category. 

| Packet Type          | More Fragment  | Fragment Offset | L4 Header |
|----------------------|----------------|-----------------|-----------|
| Non Fragmented       | 0              | 0               | Yes       |
| Initial Fragment     | 1              | 0               | Yes       |
| Non-Initial Fragment | 1              | Non-Zero        | No        |
| Non-Initial Fragment | 0              | Non-Zero        | No        |

As per the following [Documentation](https://www.cisco.com/c/en/us/support/docs/ip/generic-routing-encapsulation-gre/8014-acl-wp.html#fragkeyscenes "Documentation"), non-fragments and the initial fragment of an IP packet can contain both Layer 3 and 4 information that the ACLs can match against, for a permit or deny decision. Non-initial fragments are typically allowed through the ACL, because they can be blocked based only Layer 3 information in the packets. However, because these packets do not contain Layer 4 information, they do not match the Layer 4 information in the ACL entry, if it exists. Allowing the non-initial fragments of an IP datagram through is acceptable because the host receiving the fragments is not able to reassemble the original IP datagram without receiving all the fragments. These initial or non-initial fragments, may always not be, legitimate packets.

## Understanding the keyword: Fragments

From the above discussion, let us see the use of the keyword Fragments and how it works on NCS55xx and NCS5xx.

![Screenshot 2020-08-10 at 7.23.55 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.23.55 PM.png)

Consider the above scenario. A host wants to access a web server inside the network. The network administrator configures an access list which should allow all non fragmented packets from any user to that server on port number 80. This access-list allows any user to the destination with tcp port 80 and denies all other services.


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 fragment 
ipv4 access-list fragment
 <mark>10 permit tcp any host 70.1.1.2 eq www
 20 deny ipv4 any any</mark>
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>

### ACL Verification 

```
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 usage pfilter location 0/0/CPU0
Thu Aug  6 17:27:24.056 UTC
Interface : TenGigE0/0/0/0.10 
    Input  ACL : Common-ACL : N/A  ACL : fragment  
    Output ACL : N/A



RP/0/RP0/CPU0:N55-24#show access-lists ipv4 fragment hardware ingress verify location 0/0/CPU0
Thu Aug  6 17:27:47.309 UTC

Verifying TCAM entries for fragment
Please wait...                                   
                                                 
 
 
    INTF    NPU lookup  ACL # intf Total  compression Total   result failed(Entry) TCAM entries 
                type    ID  shared ACES   prefix-type Entries        ACE SEQ #     verified
 ---------- --- ------- --- ------ ------ ----------- ------- ------ ------------- ------------
 
TenGigE0_0_0_0.10 (ifhandle: 0x41b8)

              0 IPV4      1      1      2 NONE              4 passed                          4

RP/0/RP0/CPU0:N55-24#

```

Note: The fragment keyword is available in the default key and this can be applied to all the systems including Q-MX,Jericho,Jericho+ with and without external TCAM.
{: .notice--info}


### Traffic Tests and Validation

First we send non fragmented packets with TCP destination port 80. ACE 10 is matching the packets allowing the access to the web server.

![Screenshot 2020-08-10 at 7.29.14 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.29.14 PM.png)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 fragment hardware ingress location 0/0/CPU0 
Fri Aug  7 05:35:14.811 UTC
ipv4 access-list fragment
 <mark>10 permit tcp any host 70.1.1.2 eq www (70495 matches)</mark>
 20 deny ipv4 any any
</code>
</pre>
</div>

Now let us modify the packet and make it a fragmented packet and check if the ACL allows or denies the traffic

![Screenshot 2020-08-10 at 7.29.22 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.29.22 PM.png)


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 fragment hardware ingress location 0/0/CPU0 
Fri Aug  7 05:39:50.345 UTC
ipv4 access-list fragment
 <mark>10 permit tcp any host 70.1.1.2 eq www (1863448 matches)</mark>
 20 deny ipv4 any any
</code>
</pre>
</div>

From the above, we could see the that Fragmented packets are also making their way through the network, which is against what the network administrator had intented. It permits these packets because non-initial fragments do not contain Layer 4 information, and the ACL logic assumes that if the Layer 3 information matches, then the Layer 4 information would also match, if it was available. This could lead to data plane security issues. 

Now how to stop this ? Lets see the use of the keyword Fragments and how we can use the same to drop the fragmented packets and allow only non-fragmented traffic.

**Modifying the ACL as below** 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 fragment 
Fri Aug  7 07:07:09.803 UTC
ipv4 access-list fragment
 <mark>10 deny ipv4 any host 70.1.1.2 fragments
 20 permit tcp any host 70.1.1.2 eq www
 30 deny ipv4 any any</mark>
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>

### Sending non fragmented packets with TCP destination port 80

We can see from the below output that ACE 20 is matching and traffic is allowed. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 fragment hardware ingress location 0/0/CPU0 
Fri Aug  7 07:12:37.129 UTC
ipv4 access-list fragment
 10 deny ipv4 any host 70.1.1.2 fragments
 <mark>20 permit tcp any host 70.1.1.2 eq www (20260 matches)</mark>
 30 deny ipv4 any any
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>

### Sending fragmented packets: Non-initial Fragment with Offset > 0

We can see the packets are matching the ACE 10 now and traffic is denied 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 fragment hardware ingress location 0/0/CPU0 
ipv4 access-list fragment
 <mark>10 deny ipv4 any host 70.1.1.2 fragments (40966 matches)</mark>
 20 permit tcp any host 70.1.1.2 eq www
 30 deny ipv4 any any
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>

We can use the below command, to check the node counters to see the reason behind the packet drops. Here we can see the counter is getting increased due to deny ACL

```
RP/0/RP0/CPU0:N55-24#show spp node-counters location 0/0/CPU0 | in ACL
PUNT ACL_DENY: 220484
RP/0/RP0/CPU0:N55-24#
```


### Limitation with Keyword: Fragments

In the above section, we saw how a fragmented packet with a non zero offset value is filtered out. The limitation of the keyword Fragments is, it can be used only for offset values greater than 0. If we want to filter out the initial fragments (FO=0), we will not be able to do by this keyword. For example consider the below packet 

It has More Fragment = 1 and Fragment Offset = 0. Therefore this is the initial fragment.

![Screenshot 2020-08-10 at 7.32.22 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.32.22 PM.png)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-list ipv4 fragment hardware ingress location 0/0/CPU0  
Fri Aug  7 08:08:07.663 UTC
ipv4 access-list fragment
 10 deny ipv4 any host 70.1.1.2 fragments
 <mark>20 permit tcp any host 70.1.1.2 eq www (264506 matches)</mark>
 30 deny ipv4 any any
</code>
</pre>
</div>

We can see it matches the ACE 20 and traffic is allowed, though it is a fragmented packet. To avoid this scenario, we need to use another keyword Fragment-Type. 


## Understanding the keyword: Fragment-Type

Let us try to understand the keyword Fragment-Type in more details and the way we can use it to overcome the limitation of the keyword Fragments. 

### Fragment-Types 

Below are the options available for matching the fragments

| Fragment Type  | Description               | Flags           |
|----------------|---------------------------|-----------------|
| dont-fragment  | Match don't fragment flag | DF=1            |
| first-fragment | Match first fragment flag | MF=1 & FO=0     |
| is-fragment    | Match is fragment flag    | Any fragments.  |
| last-fragment  | Match last fragment flag  | MF=0 and FO>0   |

### Configuring ACL with fragment-type

We will take into consideration packet type which is first-fragment. MF=1 & FO=0 (this is the packet which escaped the keyword - fragments)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-20#show access-lists ipv4 fragment-type 
Fri Aug  7 12:15:36.349 UTC
ipv4 access-list fragment-type
 <mark>10 deny ipv4 any host 60.1.1.2 fragment-type first-fragment
 20 permit tcp any host 60.1.1.2 eq www
 30 deny ipv4 any any</mark>
</code>
</pre>
</div>

Note: Below hw-module profile needs to be configured along with a UDK
{: .notice--info}

```
hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto frag-bit
hw-module profile tcam acl-prefix percent 20
```

### Applying the ACL to the interface

Before applying the ACL to the interface let us understand 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto frag-bit
hw-module profile tcam acl-prefix percent 20</mark>
</code>
</pre>
</div>

Keyword _fragment-type_ is not supported with default TCAM keys. We need to define a UDK with frag-bit. When configuring it to an interface it needs to be applied along with compression level.
There are multiple levels of ACL compression supported,however the NCS5xx and NCS55xx only supports certain levels, protocols and directions. Uncompressed or compression level 0 ACLs utilize only one TCAM lookup in hardware. Compressed ACLs utilize two TCAM lookups in hardware : Stage 1: External TCAM & Stage 2: Internal TCAM. Only compress level 3 is supported. 

Because compression requires two TCAM lookups, the keyword fragment-type can only be supported on systems with an external TCAM. Compression is only supported for IPv4/IPv6 in the ingress direction.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-20#show running-config interface gigabitEthernet 0/0/0/2.10
Fri Aug  7 12:27:15.320 UTC
interface GigabitEthernet0/0/0/2.10
 ipv4 address 70.1.1.1 255.255.255.0
 encapsulation dot1q 10
 <mark>ipv4 access-group fragment-type ingress compress level 3</mark>
!
</code>
</pre>
</div>

The above 2 profiles are only applicable for systems having Jericho and Q-MX with NetLogic (NL12k) eTCAM. For systems having Jericho+ and Optimus Prime (OP) eTCAM it works by default due to larger space available. (We will dedicate a separate post for Jericho2 and its properties.)

| Hardware             | ASIC | eTCAM |
|----------------------|------|-------|
| NCS-5501-SE          | Q-MX | NL12k    |
| NCS-5502-SE          | J    | NL12k    |
| NCS55A1-36H-SE-S     | J+   | OP    |
| NC55-24H12F-SE       | J    | NL12k    |
| NC55-24X100G-SE      | J    | NL12k    |
| NC55-36X100G-A (-SE) | J+   | OP    |
| NCS55A2-MOD-SE-S     | J+   | OP    |

To understand in detail regarding the profile and compression support, please refer couple of excellent articles ([Hybrid ACL's](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part2-hybrid-acl/ "Hybrid ACL's"), [HW-Module Profiles](https://xrdocs.io/ncs5500/tutorials/ncs5500-hw-module-profiles/ "HW-Module Profiles"))

### ACL Verification

![Screenshot 2020-08-10 at 7.32.39 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.32.39 PM.png)

![Screenshot 2020-08-10 at 7.32.52 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.32.52 PM.png)

![Screenshot 2020-08-10 at 7.33.13 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.33.13 PM.png)

### Traffic Tests and Validation

First we send non fragmented packets with TCP destination port 80 and could ACE 20 is matching and traffic is permitted.

![Screenshot 2020-08-10 at 7.47.06 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-10 at 7.47.06 PM.png)


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-20#show access-lists ipv4 fragment-type hardware ingress location 0/0/CPU0 
Fri Aug  7 15:30:33.954 UTC
ipv4 access-list fragment-type
 10 deny ipv4 any host 60.1.1.2 fragment-type first-fragment
 <mark>20 permit tcp any host 60.1.1.2 eq www (513249 matches)</mark>
 30 deny ipv4 any any
RP/0/RP0/CPU0:N55-20#
</code>
</pre>
</div>


### Sending first fragmented packet MF=1 and Offset = 0

Packets are matching the ACE 10 and getting denied

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-20#show access-lists ipv4 fragment-type hardware ingress location 0/0/CPU0
Fri Aug  7 17:28:06.005 UTC
ipv4 access-list fragment-type
 <mark>10 deny ipv4 any host 60.1.1.2 fragment-type first-fragment (186591 matches)</mark>
 20 permit tcp any host 60.1.1.2 eq www
 30 deny ipv4 any any
RP/0/RP0/CPU0:N55-20#
</code>
</pre>
</div>

We can also use is-fragment in place of first-fragment.

```
RP/0/RP0/CPU0:N55-20#show spp node-counters location 0/0/CPU0 | in ACL
Fri Aug  7 17:32:14.929 UTC
                 PUNT ACL_DENY:           23004
RP/0/RP0/CPU0:N55-20#show spp node-counters location 0/0/CPU0 | in ACL
Fri Aug  7 17:32:18.124 UTC
                 PUNT ACL_DENY:           23278
RP/0/RP0/CPU0:N55-20#
```

### Changing the ACE 10 to fragment-type don't fragment with same packet MF=1 and Offset = 0 


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-20#show access-lists ipv4 fragment-type 
Fri Aug  7 17:37:43.211 UTC
ipv4 access-list fragment-type
 <mark>10 deny ipv4 any host 60.1.1.2 fragment-type dont-fragment</mark>
 20 permit tcp any host 60.1.1.2 eq www
 30 deny ipv4 any any
RP/0/RP0/CPU0:N55-20#
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-20#show access-lists ipv4 fragment-type hardware ingress location 0/0/CPU0 
Fri Aug  7 17:37:00.856 UTC
ipv4 access-list fragment-type
 10 deny ipv4 any host 60.1.1.2 fragment-type dont-fragment
 <mark>20 permit tcp any host 60.1.1.2 eq www (98063 matches)</mark>
 30 deny ipv4 any any
RP/0/RP0/CPU0:N55-20#
</code>
</pre>
</div>

From the above output we can see, it is not matching sequence 10, hence it moves to the next sequence 20. The fragment matches the criteria, hence it is permitted.

  
**To summarise the behaviour for different fragment types**

Considering Packet-Fragment has MF=1 and FO=0

| Fragment Type  | Action     | Reason               |
|----------------|------------|----------------------|
| first-fragment | Dropped    | Expects MF=1 FO=0    |
| dont-fragment  | Permitted  | Expects DF=1         |
| is-fragment    | Dropped    | Matches Any fragments|
| last-fragment  | Permitted  | Expects MF=0 FO>0    |

You can use the available options for fragment-type, to filter the fragments as per different scenarios. Fragment-type keyword gives user granular level of filtering the packets.

## References

- [CCO Config Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/71x/b-ip-addresses-cg-ncs5500-71x/b-ip-addresses-cg-ncs5500-71x_chapter_0111.html "CCO Config Guide")
- [Fragmentation White Paper](https://www.cisco.com/c/en/us/support/docs/ip/generic-routing-encapsulation-gre/8014-acl-wp.html "Fragmentation White Paper")
- [Security ACL Part 2](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part2-hybrid-acl/ "Security ACL Part 2")
- [NCS5500 HW-Module Profiles](https://xrdocs.io/ncs5500/tutorials/ncs5500-hw-module-profiles/ "NCS5500 HW-Module Profiles")


## Summary

Fragmentation is a process of breaking bigger packet into smaller packets and reassembling it. We saw how malicious fragments can make their way in and cause security issues. NCS5xx and NCS55xx is equipped with the capabilities to provide that extra security to the data plane. The platforms support different keywords - Fragments and Fragment-type to filter out fragments which are not expected to enter the network. They can be filtered before reaching the target. Hope this helps to clear the filtering criteria for fragmented packets. 

Stay tuned for next article, where we will explore another ACL matching capabilities of the portfolio.
