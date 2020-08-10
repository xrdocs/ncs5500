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
position: hidden
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

As per the following [Documentation](https://www.cisco.com/c/en/us/support/docs/ip/generic-routing-encapsulation-gre/8014-acl-wp.html#fragkeyscenes "Documentation"), non-fragments and the initial fragment of an IP packet can contain both Layer 3 and 4 information that the ACLs can match against for a permit or deny decision. Non-initial fragments are typically allowed through the ACL, because they can be blocked based only Layer 3 information in the packets. However, because these packets do not contain Layer 4 information, they do not match the Layer 4 information in the ACL entry, if it exists. Allowing the non-initial fragments of an IP datagram through is acceptable because the host receiving the fragments is not able to reassemble the original IP datagram without receiving all the fragments. These initial fragments, may always not be, legitimate packets.

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
{: .notice—info}

